---
title: GCP, Jenkins + Docker로 Spring Boot CI/CD 구축하기
date: 2024-02-22 18:20:00 +/-TTTT
categories: [Infra, CI/CD]
tags: [gcp, cicd, infra, jenkins, docker]     # TAG names should always be lowercase
---

> GDSC Solution Challenge에 도전하면서, 처음 GCP를 사용하게 되었다!  
> 기존 스프링부트 앱을 개발할 때 보통 AWS의 CodeDeploy (+ S3, EC2)를 활용하여 CI/CD를 구축하였는데,  
> 이번엔 GCP도 사용해볼겸, 새로운 툴인 Jenkins를 통하여 CI/CD를 구축하는 방법에 대해 알아본 내용을 정리해보았다.
{: .prompt-info}

## 과정 정리

### GCP Compute Engine 설정

#### 세팅 정보

서버는 스프링부트용 서버, 젠킨스용 서버 총 2개를 세팅해주었다.

-   ubuntu
-   http/https 트래픽 허용 (미리 8080 - 9090 포트를 열어줌.)
-   서버
    1.  remine-springboot-server \[스프링부트용 서버\]
    2.  remine-jenkins-server \[젠킨스용 서버\]

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/0.png) | ![img](/assets/img/2024-02-22-gcp-jenkins-cicd/1.png)

#### 외부 고정 주소 예약

**VPC 네트워크 > IP 주소**로 들어가서 두 인스턴스 모두 외부 고정 IP 주소를 사용할 수 있도록 설정해주었다.

-   전역이 아닌 지역으로 설정:  **asia-northeast3** (서울) 로 설정함.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/2.png){: w="600" }

---

### \[VM\] 스왑 메모리 설정

#### 스왑 메모리를 설정하는 이유?

Swap 메모리는 가상 메모리를 구현하는 방법 중 하나로, 물리적 메모리가 부족할 때 시스템에 사용될 메모리를 추가적으로 자원을 할당하기 위해 사용된다.

(물론 GCP의 경우는 아니지만..) AWS EC2를 프리티어로 설정하고 Gradle 빌드 작업을 시도하는 과정에서 서버의 메모리가 부족했던 경험이 있다. 그래서 이번에도 미리 스프링부트 서버와 젠킨스 서버 둘 다 스왑 메모리를 설정해주었다.

#### 설정 방법

``` bash
$ sudo dd if=/dev/zero of=/swapfile bs=128M count=16
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
$ sudo  swapon -s
$ sudo vi /etc/fstab
```

아래 내용을 **/etc/fstab**에 추가한다.

```
/swapflie swap swap defaults 0 0
```

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/3.png){: w="600" }

그러면 다음 명령어를 통해 메모리 설정이 완료되었음을 확인할 수 있다.

```bash
$ free -h
```

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/4.png){: w="600" }

#### 참고 자료

[https://repost.aws/ko/knowledge-center/ec2-memory-swap-file](https://repost.aws/ko/knowledge-center/ec2-memory-swap-file)

---

### \[DOCKER\] 도커 설치하기

두개의 Compute Engine 중에서 Jenkins server에 설치를 해주도록 하자.

> 이 뒤로도 쭉 따로 언급이 없는 이상 젠킨스 서버에서 계속 세팅해주는 것이다!

아래 명령어들을 쭉 쳐서 도커를 설치해주자!

``` shell
$ sudo apt update
$ sudo apt-get install ca-certificates curl gnupg
$ sudo install -m 0755 -d /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ sudo chmod a+r /etc/apt/keyrings/docker.gpg

$ echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
$ sudo chmod 006 /var/run/docker.sock
```

아래 명령어를 통해 설치를 확인할 수 있다.

```shell
$ docker -v
```

---

### \[JENKINS\] 젠킨스 설치하기

도커에서 젠킨스 공식 이미지를 pull 받아와주자.

```shell
$ docker pull jenkins/jenkins:jdk17
```

아래 명령어를 통하여 다운로드 된 이미지를 확인해줄 수 있다.

```shell
$ docker images
```

젠킨스 이미지를 컨테이너로 실행해보자.

```shell
$ docker run -p 80:8080 -v /var/run/docker.sock:/var/run/docker.sock --name jenkins jenkins/jenkins:jdk17
```

> `-v /var/run/docker.sock:/var/run/docker.sock` 는 ubuntu 서버의 docker를 `jenkins`의 컨테이너에 마운트해준다는 의미이다. <br>
> ※ 마운트: 호스트의 파일 시스템 경로를 컨테이너 내부에 연결하는 것. <br>
> 이렇게 명령어를 작성하면 해당 컨테이너 내에 따로 Docker를 설치할 필요가 없다!
{: .prompt-tip}

참고로 여기서, Jenkins를 80번 포트로 띄워주었는데,  
GCP의 네트워크 보안 > 방화벽 정책 설정을 통해 미리 VM의 포트를 열어놓아야 한다!

---

### \[JENKINS\] 젠킨스 초기 설정

다음과 같은 위치에서 젠킨스의 초기 비밀번호를 확인할 수 있다.

만약 로그가 뜨지 않았다면 jenkins shell로 이동 후 로그인 secret을 확인할 수 있다.

```shell
$ docker exec -it {도커 컨테이너 이름} /bin/bash
$ cat /var/jenkins_home/secrets/initialAdminPassword
```

-   만약 위에서 컨테이너 실행 명령어를 똑같이 입력, 실행했다면 --name을 jenkins로 설정했으므로 도커 컨테이너 이름은 jenkins 이다!

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/5.png){: w="600" }
_다음과 같이 초기 비밀번호를 확인한다._

젠킨스 VM의 퍼블릭 IPV4:젠킨스 포트로 접속하면 다음과 같이 초기 비밀번호를 입력할 수 있다.

`Administrator password` 입력 부분에 초기 비밀번호를 입력하자.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/6.png){: w="500" }

Continue 버튼을 누르면 다음과 같은 페이지로 넘어가게 되는데, 여기서 `Install suggested plugins`를 선택해주면 된다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/7.png){: w="500" }

플러그인을 모두 다운 받았다면, admin user를 생성하는 페이지가 나오는데

**여기서는 꼭 관리자 유저를 생성**해주고 save and continue를 눌러주자.

-   skip and continue as admin을 선택하면 나중에 추가적인 멤버 설정이 필요하다. (저도 딱히 알고 싶지 않았음..🥲)

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/8.png){: w="500" }

---

### \[GITHUB\] git 접근을 위한 credential 생성

이제 젠킨스에서 git을 접근할 수 있도록 자신의 프로필의 credential을 생성해줄 것이다.

GitHub에서 프로필 사진을 누르면 나오는 사이드바의 **Settings** > 왼쪽 사이드의 **Developer settings** 에 들어가자

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/9.png){: w="300" }

Personal access tokens > **Tokens (classic)**에 들어가서,

Generate new token > **Generate new token(classic)**을 클릭하자

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/10.png){: w="600" }

토큰의 사용 용도를 Note에 간단히 적어주고, Scope를 설정해주면 된다.

-   **repo**, **admin:repo\_hook** 에 체크를 해주었다. (아래 사진 참고)

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/11.png){: w="460" }

그 이후 **Generate token**을 클릭하면 ghp\_로 시작하는 토큰을 확인할 수 있다.

(이때만 확인가능하기 때문에 메모장에 잘 복사해두자!)

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/12.png){: w="600" }

---

### \[JENKINS\] Jenkins에 Git credential 설정

**Dashboard > Jenkins 관리 > System** 에 접속한다.

GitHub 부분에 있는 **+ADD** 버튼을 누른 뒤, **Jenkins** 버튼을 클릭하여 새로운 Credential 두 개를 생성해줄 예정이다.

(ADD 버튼 위치는 아래 사진 참고)

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/13.png){: w="500" }

#### 새로운 Credential 설정

1\. **jenkins-github-secret-text**

Secret에 아까 발급받은 ghp\_ 로 시작하는 키를 붙여넣는다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/14.png){: w="500" }
_jenkins-github-secret-text 설정_

2\. **jenkins-github-username-password** 

Username엔 github 이메일, Password엔 아까 발급받은 ghp\_로 시작하는 키를 붙여넣는다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/15.png){: w="500" }
_jenkins-github-username-password 설정_

#### Connection Test

-   Name엔 repository url
-   Credential엔 위에서 작성한 **jenkins-github-secret-text** 를 설정하자

우측 하단의 Test connection 버튼을 누르고 Credentials verified for user 5jisoo, rate limit: 4999 와 같은 문구가

좌측 하단에 생성되면 연결에 성공한 것이다! (동시에 Credential 설정도 잘 된것!)

저장 버튼을 눌러 잘 저장해주자

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/16.png){: w="500" }

---

### \[JENKINS\] Jenkins에 Git Clone Pipeline 생성

#### 새로운 파이프라인 생성

젠킨스 좌측 **\+ 새로운 Item**을 클릭하여 Pipeline을 새롭게 생성해보자

-   이름은 마음대로 지어도 되고,
-   **Pipeline**을 선택한 뒤 **OK**를 누른다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/17.png){: w="550" }

하단에 pipeline syntax를 클릭해준다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/18.png){: w="550" }

#### Git Clone 스크립트 작성하기

Steps에 다음과 같이 기입하고 **Generate Pipeline Script**를 클릭하면 자동으로 파이프라인 스크립트를 작성해준다!

-   Branch: trigger가 되는 브랜치는 마음대로 작성한다 (ex main, develop .. 등등)
    -   이 때 브랜치는 **꼭 존재하는 브랜치**로 설정하자!
-   Credential은 아까 등록한 `jenkins-github-username-password`으로 설정해준다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/19.png){: w="550" }

Script를 복사하여 다음과 같이 파이프라인을 작성해주자.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/20.png){: w="600" }

```
pipeline {
    agent any
    
    stages {
        stage('Git Clone') {
            steps {
                git branch: 'develop', 
                credentialsId: 'jenkins-github-username-password', 
                url: 'https://github.com/re-Mine/reMine-Server'
            }
        }
    }
}
```

저장을 누른 뒤, 지금 빌드 버튼을 클릭하여 클론 여부를 확인해보자

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/21.png){: w="300" }

#### Clone 확인하기

아래 명령어들을 작성해주자.

-   도커 컨테이너엔 vim도 안깔려있으므로 설치해야 한다.  
    → vim을 설치할 때 권한 에러가 발생하면? 이 글 하단에 있는 트러블 슈팅 부분의 '`도커 컨테이너 권한 에러 발생할 경우`' 부분을 참고하면 된다.

```shell
$ apt-get update
$ apt-get upgrade
$ apt-get install vim
```

jenkins 이미지에 접속한 뒤,  
`/var/jenkins\_home/workspace/spring-pipeline(파이프라인 이름)`으로 이동하면 클론된 내용을 확인할 수 있다.

```shell
$ docker exec -it jenkins /bin/bash
$ cd /var/jenkins_home/workspace/spring-pipeline
$ ls -al
```

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/22.png){: w="600" }
_클론 성공!_

---

### \[JENKINS\] 민감 정보 추가 (키 파일 추가)

보통 관리하는 **application-000.properties** 파일들은 민감정보라서 .gitignore 로 github에 공개적으로 올리지 않는데,

이런 민감 정보가 포함된 파일을 설정하는 여러가지 방법(환경 변수 등..)이 있지만 Jenkins 자체에서 제공하는 Credential 기능을 사용해보았다!

Jenkins 관리 > Credentials > 아래 사진처럼 (global) 부분 클릭해서 Global credentials (unrestricted) 들어가기

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/23.png){: w="600" }

#### Credential 추가

Add Credentials 클릭해서 다음과 같이 세팅해주기

-   Kind: Secret file
-   Scope: Global (default)
-   File 선택하기
-   ID: 마음대로 - 파이프라인 작성할 때 쓰이므로 식별가능한 아이디로 작성합니다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/24.png){: w="600" }

#### 파이프라인 작성

파이프라인 > Configuration(구성) 에 들어오면 파이프라인 수정이 가능합니다

-   git clone 이후 실행될 수 있도록 작성해줍시다
-   경로나, 파일명은 각 프로젝트 세팅에 맞게 설정해줍니다

```
stage('Copy application-SECRET.properties') {
	steps {
    	withCredentials([file(credentialsId: 'application-secret', variable: 'dbConfigFile')]) {
        	script {
            	sh 'cp $dbConfigFile src/main/resources/application-SECRET.properties'
            }
		}
	}
}
```

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/25.png){: w="600" }

#### Credential 파일 추가 확인

지금 빌드 버튼을 눌러서 빌드를 실행하고, docker 에 접속하여 확인합니다

```
$ cd /var/jenkins_home/workspace/spring-pipeline(파이프라인 이름)/src/main/resources
$ ls -al
```

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/26.png){: w="600" }
_키 파일 정상 이동 확인_

---

### \[JENKINS\] Build Pipeline 작성

#### gradle 빌드 파이프라인 작성하기

참고로 여기서 **./gradlew clean build**라고 작성하면 굉장히 오래 걸린다 (11분 걸림;;)

-   **이전 캐시 삭제가 필요할 때만 clean을 작성**하도록 하자

```
stage('build with gradle') {        
		steps {
		    sh 'chmod +x gradlew'
		    sh './gradlew build'
		}
}
```

#### 빌드 확인하기

build된 jar 파일은 build/libs 파일에 저장된다!

해당 위치로 이동해서 jar 파일이 무사히 생성되었는지 확인해보자.

-   만약 에러가 발생했다면, 키 파일이 빠진 건 없는지 잘 확인해보자.
-   에러는 Build History의 **#n** 부분을 클릭한 뒤 **Console output** 을 보면 실시간 콘솔 로그를 확인할 수 있다.

---

### \[GCP\] Spring Boot Server의 SSH KEY 설정

이제 Jenkins Server에서 Spring Boot Server로 SSH 접속이 가능하도록 Spring Boot Server에 키를 설정해주자.

윈도우는 puttygen을 이용하여 ppk, pem 키를 만들어줄 수 있는데 만드는 방법은 구글링하면 많고 굉장히 쉬우니 패스하겠다! 이 때 나오는 **ssh-rsa**로 시작하는 public key 부분은 잘 복사해두면 된다.

#### SSH 공개 키 추가

spring boot server 인스턴스에 들어가서 **수정** 버튼을 누르자

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/27.png){: w="600" }

SSH 키 부분에서 **항목 추가** 버튼을 눌러서 아까 복사해둔 ssh-rsa로 시작하는 공개키를 추가해준 뒤, 저장한다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/28.png){: w="500" }

#### 공개키 확인하기

Spring Boot 서버에 SSH 접속을 해보자

-   참고로, 콘솔에서 SSH 옆 **▼** 버튼, **브라우저 창에서 열기** 버튼을 누르면 바로 브라우저에 접속이 가능하다

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/29.png){: w="500" }

.ssh 폴더로 이동하여 authorized\_keys 파일을 확인하여 방금 추가한 SSH 공개키가 있는지 확인한다

```shell
$ cd .ssh
$ cat authorized_keys
```

만약 이 때 공개키가 없다면 **vim 편집기를 활용하여 수동으로** 넣어주는 것도 가능하다.

```shell
$ vim authorized_keys
```

---

### \[JENKINS\] SSH Credential 설정 및 플러그인 설치

#### SSH Credential 설정

아까와 동일하게 Credential 을 추가하자

Jenkins 관리 > Credentials > 아래 사진처럼 (global) 부분 클릭해서 Global credentials (unrestricted)에 들어가자.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/30.png){: w="600" }

**Add Credentials** 버튼을 클릭하고 다음과 같이 Spring Server의 Compute Engine에 접근하는 pem키를 등록해주자

-   참고로 pem키는 **\----BEGIN RSA PRIVATE KEY---—** 부터  
     **\-----END RSA PRIVATE KEY-----** 까지 전부 붙여넣어주어야 한다.
-   Scope는 Global, ID는 **jenkins-ssh-key**로 설정해주었다.
-   Username은 Ubuntu 서버의 기본 유저인 **ubuntu**를 넣어주었다.  
    (따로 설정을 했을 경우 변경해서 넣어주면 된다.)

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/31.png){: w="600" }

#### SSH Agent 플러그인 설치

Jenkins 관리 > Plugins > Available plugins로 이동하여

**SSH Agent** 를 설치하고 jenkins 컨테이너를 재시작해주자

```shell
$ docker ps    # 실행중인 컨테이너 확인
$ docker ps -a # 중지된 컨테이너까지 모두 확인

$ docker restart jenkins
```

---

### \[VM\] Spring Boot Server 설정

(이 부분은 젠킨스 서버가 아닌, 스프링부트 서버의 인스턴스 설정이다.)

#### JDK 설치

아래 명령어를 작성하여 jdk를 설치해주자.

참고로 저의 프로젝트에서는 java 17을 사용하여 17 버전을 설치해주었는데, 이건 프로젝트에 맞게 설치해주면 된다.

```shell
$ sudo apt update
$ sudo apt install openjdk-17-jdk
$ java --version # 자바 버전 확인
```

#### Deploy shell 파일 작성

어플리케이션을 실행할 쉘 파일을 작성해주자.

```shell
$ mkdir remine # application 이름에 맞게 설정
$ vi deploy.sh
```

아래 shell 파일에서

APP\_NAME, APP\_LOG, ERROR\_LOG, DEPLOY\_LOG 변수 부분을 각자의 프로젝트 설정에 맞게 변경하면 된다.

```shell
#!/bin/bash

APP_NAME=remine
APP_LOG=./remine/application.log
ERROR_LOG=./remine/error.log
DEPLOY_LOG=./remine/deploy.log

CURRENT_PID=$(pgrep -f $APP_NAME)

if [ -z $CURRENT_PID ]
then
  echo "> 종료할 애플리케이션이 없습니다." >> $DEPLOY_LOG
else
  echo "> kill -9 $CURRENT_PID" >> $DEPLOY_LOG
  kill -15 $CURRENT_PID
  sleep 5
fi

JAR_NAME=$(ls $APP_NAME | grep 'SNAPSHOT.jar' | tail -n 1)

chmod +x ./$APP_NAME/$JAR_NAME
nohup java -jar -Duser.timezone=Asia/Seoul ./$APP_NAME/$JAR_NAME --logging.level.org.hibernate.SQL=DEBUG > $APP_LOG 2>$ERROR_LOG &
```

---

### \[JENKINS\] Deploy 파이프라인 작성

#### Deploy 파이프라인 생성

파이프라인 > Cofiguration에 들어가서 파이프라인을 새롭게 추가로 작성해준다

-   물론 ssh user name이 ubuntu 가 아니라면 ubuntu가 아니라 새롭게 작성해주어야 한다.

```
stage('deploy') {
    steps {
        sshagent(credentials: ['jenkins-ssh-key']) {
            sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@{spring server ip} uptime
                scp /var/jenkins_home/workspace/spring-pipeline/build/libs/*.jar ubuntu@{spring server ip}:/home/ubuntu/{application name}
                ssh -t ubuntu@{spring server ip} chmod +x ./deploy.sh
                ssh -t ubuntu@{spring server ip} ./deploy.sh
            '''
        }
    }
}
```

#### key file copy 에러가 발생한다면?

이 때 key file copy에서 에러가 발생하는데, Permission Denied 에러가 발생한다.

-   pipeline에서 sudo를 붙여도 비밀번호 입력이 필요하다는 에러가 발생하는데,  
    **chmod** 명령어를 활용해 접근권한을 수정하면 된다.

```
stage('Copy application-SECRET.properties') {
	steps {
    	withCredentials([file(credentialsId: 'application-secret', variable: 'dbConfigFile')]) {
        	script {
            	sh 'cp $dbConfigFile src/main/resources/application-SECRET.properties'
            	sh 'chmod 644 src/main/resources/application-SECRET.properties'
            }
        }
	}
}
```

#### 결과

**{스프링부트 서버 public ipv4}:8080** 포트로 접속하면 다음과 같은 Whitelabel 페이지를 확인할 수 있다.

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/32.png){: w="600" }

아래 명령어를 통해 Spring Boot 서버에서 java 파일이 배포되고 있는지 확인해보자

```shell
$ ps -ef | grep java
```

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/33.png){: w="600" }

---

### \[JENKINS | GITHUB\] Github Webhook 설정하기

#### Github 웹 훅 추가하기

Repository Settings > Webhooks > Add webhook 클릭하기

-   Payload URL : **Jenkins Server IP 주소/github-webhook/**
-   Content type: application/json

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/34.png){: w="600" }

#### Pipeline Trigger 설정

Configuration에 들어가서 **Build Triggers GitHub hook trigger for GITScm polling** 체크하기

![img](/assets/img/2024-02-22-gcp-jenkins-cicd/35.png){: w="600" }

---

## 트러블 슈팅

### 도커 컨테이너 권한 에러가 발생할 경우

상황

1.  apt-get update 를 실행시, Permission denied 에러가 발생함
2.  **sudo apt-get update**를 실행 시, sudo command not found 에러가 발생함.

이 때는 sudo도 깔아주어야 하는 상황이라서 아래 명령어처럼 docker 를 root권한으로 접속하여야 한다.

```shell
$ docker exec -itu0 jenkins /bin/bash
```

기존엔 **docker exec -it jenkins /bin/bash** 로 접근했으나, **\-itu0** 로 실행하면 루트 권한으로 접근이 가능하다.

---

## 마무리

### 보완할 점

스프링 docker-compose를 따로 사용하지 않아서 배포 과정에서 따로 Docker를 사용하진 않았는데, (Jenkins 제외)

다음번엔 도커를 도입해서 새로운 배포 파이프라인을 제작해보는 것도 좋을 것 같다!

---

### 제작하며 참고한 사이트

-   [https://hellodavid.tistory.com/78](https://hellodavid.tistory.com/78)
-   [https://velog.io/@rungoat/CICD-Jenkins-credentials%EB%A1%9C-application-secret.yml-%ED%8C%8C%EC%9D%BC-%EA%B4%80%EB%A6%AC](https://velog.io/@rungoat/CICD-Jenkins-credentials%EB%A1%9C-application-secret.yml-%ED%8C%8C%EC%9D%BC-%EA%B4%80%EB%A6%AC)
-   [https://kangwoojin.github.io/programing/gradle-build-performance/](https://kangwoojin.github.io/programing/gradle-build-performance/)