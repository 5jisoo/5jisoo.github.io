---
title: 도커 컨테이너 내부에서 localhost에 접근하기
date: 2025-1-13 02:51:00 +/-TTTT
categories: [Project, COASTER]
tags: [docker, infra, springboot, troubleshooting]
math: true
---

> 이전 포스트: [도커 컨테이너 실행 순서 조정하기](https://5jisoo.github.io/posts/adjust-docker-container-order/) 에서 이어지는 글입니다.
{: .prompt-tip}

## 에러 상황

이전 포스트를 보면 더 자세히 확인하실 수 있습니다!

한줄 요약: 스프링 컨테이너에서 레디스 컨테이너 연결이 되지 않고 있습니다.

## 호스팅 환경

Azure Virtual Machine 으로 호스팅하였습니다.

- Ubuntu 22.04.5 LTS
- Docker version 27.4.1

## 해결 과정

먼저, 컨테이너 실행 순서를 조정해주었음에도 불구하고, 연결이 되지 않는다는 것은 스프링 연결 세팅에서 문제가 발생된 것이라고 생각하였습니다.

스프링 설정파일에서는 다음과 같이 `localhost:6379`와 연결하도록 설정해주었습니다.

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```

localhost와의 연결에서 문제가 발생된 것이라고 생각했고, 곧 이어 컨테이너 내부의 localhost가 아닌 VM의 localhost와 연결할 수 있도록 따로 설정이 필요한지 찾아보았습니다.

그리고, [ 이 글(stack overflow: From inside of a Docker container, how do I connect to the localhost of the machine?)](https://stackoverflow.com/questions/24319662/from-inside-of-a-docker-container-how-do-i-connect-to-the-localhost-of-the-mach)을 발견할 수 있었습니다.

<br>

위 글과 [공식문서 (Add entries to container hosts file)](https://docs.docker.com/reference/cli/docker/buildx/build/#add-host)를 추가로 확인하여 요약하자면,

1. docker compose에 다음과 같은 옵션을 추가하여 컨테이너 호스트 파일에 항목을 추가합니다.
    ``` yaml
    extra_hosts:
        - "host.docker.internal:host-gateway"
    ```
    - 참고로 docker를 실행할 때 `--add-host=host.docker.internal:host-gateway` 옵션으로 대신할 수 있습니다.
2. container 끼리의 연결이 아닌, 호스트에 연결해야하는 경우 `host-gateway`라는 특수값을 사용하여 설정합니다.
3. 이렇게 설정하면, container 내부에서 `host.docker.internal`를 호스트 게이트웨이 IP로 확인할 수 있습니다.

### 최종 docker-compose.yml 파일

위 옵션을 추가한 최종 파일은 다음과 같습니다.

```yaml
services:
  redis:
    image: redis:alpine
    command: redis-server --port 6379
    container_name: coaster_redis
    hostname: root
    volumes:
      - ./redis/data:/data
      - ./redis/conf/redis.conf:/usr/local/conf/redis.conf
    labels:
      - "name=redis"
      - "mode=standalone"
    healthcheck:
      test: [ "CMD", "redis-cli", "ping" ]
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - 6379:6379
    extra_hosts: 
      - host.docker.internal:host-gateway
  api:
    image: usr/coaster:latest
    container_name: coaster
    environment:
      - TZ=Asia/Seoul
      - LANG=ko_KR.UTF-8
      - HTTP_PORT=8080
    ports:
      - '8080:8080'
    depends_on:
      redis:
        condition: service_healthy
    volumes:
      - /var/log/coaster:/var/log/coaster
    extra_hosts:
      - host.docker.internal:host-gateway
```

### application.yml 파일 수정

그리고 컨테이너 내부에서 호스트 게이트웨이 IP를 확인할 수 있도록 `localhost`를 `host.docker.internal`로 변경해주었습니다.

따라서 스프링 설정 파일도 다음과 같이 변경됩니다.

```yaml
spring:
  data:
    redis:
      host: host.docker.internal
      port: 6379
```

저는 개발 및 관리의 편의성을 위해 spring profile을 dev와 local로 나누어 
- dev에서는 `host.docker.internal`로, 
- local에서는 `localhost`로 연결되도록 설정하였습니다.

## 해결 완료!

docker compose를 통해 다시 도커 컨테이너를 실행하고, 로그를 확인하면? 스프링 컨테이너가 정상적으로 실행된 것을 확인할 수 있었습니다!

```shell
% docker compose up
% docker logs {container-name}
```

![img](/assets/img/2025-01-13-connect-localhost-inside-container/1.png)
_해결 완료_