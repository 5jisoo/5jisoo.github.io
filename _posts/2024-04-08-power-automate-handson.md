---
title: «Power Automate로 업무 자동화하기» 핸즈온
date: 2024-04-08 18:20:00 +/-TTTT
categories: [Microsoft Learn Student Ambassadors]
tags: [mlsa, powerautomate, microsoft]     # TAG names should always be lowercase
description: Power Automate를 사용한 핸즈온 내용을 정리합니다!
image:
  path: /assets/img/2024-04-08-power-automate-handson/0.png
  alt: 핸즈온 시나리오
---

사전에 Teams로 미리 첨부해둔 Word 파일입니다. <br>
핸즈온에서 사용되는 파일이니, 이 글을 읽고 따라해주실 분들은 아래 파일을 다운받아 실습을 진행해주시면 됩니다.

[지원서_원본.docx](/assets/img/2024-04-08-power-automate-handson/지원서_원본.docx)

## 핸즈온 시나리오
### 동아리 운영진 업무 자동화하기
수십개의 지원서를 엑셀 파일로 확인해야 하는 상황에서, <br>
각 지원서를 PDF 문서로 한 장씩 깔끔하게 만들어서 확인해보자!

## 운영진들의 사전 준비

> 여기 "운영진들의 사전 준비" 부분은 세션에서는 사전에 미리 운영진들이 준비해두었던 부분이지만, <br>
> 혹시 다른 Teams (작업 환경)에서 이번 세션 핸즈온을 따라하고 싶으신 분들을 위해 준비한 부분입니다.
{: .prompt-info}

### 지원서 원본 파일을 Teams에 넣어두기
Teams의 파일 탭에서는 다음과 같이 파일을 드래그 앤 드랍으로 바로 업로드할 수 있습니다. <br>
위에 있는 지원서_원본.docx 파일을 Teams에 업로드합니다.

![img](/assets/img/2024-04-08-power-automate-handson/1.png){: w="600" }

### 원본 파일 열 추가하기
또한 미리 Teams에서 파일의 열을 추가해주어야 합니다.  <br>
지원서_원본.docx 파일을 자신의 Teams의 파일 탭에 넣고, 오른쪽 상단 쯤에 있는 +열 추가 버튼을 눌러줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/2.png){: w="600" }

|------|---|
|![img](/assets/img/2024-04-08-power-automate-handson/3.png)|텍스트를 선택해주고, 다음 버튼을 눌러줍니다.|


|------|---|
|![img](/assets/img/2024-04-08-power-automate-handson/4.png)|그럼 또 다음과 같이 열 만들기 창이 뜨게 됩니다. <br> 총 네 개의 열을 만들어주어야 하는데, 이름과 유형을 아래와 같이 설정합니다.|

|이름|유형|
|----|----|
|지원자 이름|한 줄 텍스트|
|지원자 학과|한 줄 텍스트|
|지원자 학번|한 줄 텍스트|
|지원 동기|여러 줄 줄 텍스트|

만약 4개의 열을 정상적으로 만든 뒤에도, 파일 탭에서 열이 잘 보이지 않다면, <br>
`+ 열 추가` 버튼을 누르면, `열 표시 또는 숨기기` 버튼을 확인할 수가 있습니다. 

![img](/assets/img/2024-04-08-power-automate-handson/5.png){: w="600" }


**지원자 이름 / 학과 / 학번 / 지원 동기** 열이 모두 보일 수 있도록 체크하고, <br>
적용 버튼을 누르면 열을 표시하도록 설정할 수 있습니다.

> 참고로, 열의 순서는 드래그 앤 드랍으로 변경할 수 있는데, 
> 실습의 편의성을 위해 모든 채널엔 이름, 학과, 학번, 지원 동기 순서대로 열을 배치해 두었습니다.
> ![img](/assets/img/2024-04-08-power-automate-handson/7.png){: w="500" }
{: .prompt-tip}

## 핸즈온 준비

### Teams 접속하기
팀즈의, 실습을 진행할 채널에 들어갑시다.

> 아래 실습용 이미지는 기본 세팅 채널, user1 등등의 채널에서 진행되고 있지만, <br>
> 실제로 따라하실 때는 위 사전 과정에 있는 **지원서_원본 파일이 세팅된 채널에서** 진행하셔야 한다는 점 유의하시길 바랍니다!
{: .prompt-warning}

### 새로운 폴더 추가
상단에 + 새로 만들기 버튼을 누르고 폴더 버튼을 눌러 폴더를 하나 추가해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/8.png){: w="600" }

폴더 이름은 문서라고 지어주겠습니다.

![img](/assets/img/2024-04-08-power-automate-handson/9.png){: w="600" }

- 다시 새로 만들기 버튼을 누르고, 폴더를 선택하여 또 다른 폴더를 하나 생성해주겠습니다. <br>
    폴더 이름은 복사라고 지어줍니다.

### Forms로 이동하기

좌측 상단에 `…` 버튼을 눌러 office 앱 시작 관리자로 이동해줍시다

![img](/assets/img/2024-04-08-power-automate-handson/11.png){: w="600" }

`Microsoft 365` 버튼을 눌러 다시 Microsoft 365 홈으로 이동해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/12.png){: w="300" }

똑같이 오른쪽 상단에 `……` 버튼을 눌러 앱 시작 관리자를 열어준 뒤, <br>
모든 앱 탐색 버튼을 눌러줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/13.png){: w="300" }

상단 검색창에서 Forms를 검색하여 Forms로 이동해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/14.png){: w="600" }

### 새 지원 양식 만들기

새로운 동아리 폼을 제작해봅시다. <br>
왼쪽 상단에 “새 양식” 부분을 클릭합니다.

![img](/assets/img/2024-04-08-power-automate-handson/15.png){: w="600" }

아래와 같이 이름, 학과, 학번, 지원 동기 입력란을 생성합니다.
- 왼쪽 하단의 새로 추가 버튼을 누른 뒤 텍스트 버튼을 클릭하면 생성할 수 있습니다.

![img](/assets/img/2024-04-08-power-automate-handson/16.png){: w="600" }

모든 문항은 필수로 체크해주고, 지원 동기란은 긴 답변을 체크합니다.

![img](/assets/img/2024-04-08-power-automate-handson/17.png){: w="400" }

## PowerAutomate

### 워크플로우 시나리오

![img](/assets/img/2024-04-08-power-automate-handson/18.png){: w="600" }

### Power Autoate 접속, 기본 세팅

다시 오른쪽 상단에 `……` 버튼을 눌러 앱 시작 관리자를 열어준 뒤, <br>
`모든 앱 탐색` 버튼을 눌러줍니다.

검색창에서 Power Automate를 검색하여 실행시켜줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/19.png){: w="600" }

왼쪽 상단에 `만들기` 버튼을 클릭한 뒤, `자동화된 클라우드 흐름`을 클릭해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/20.png){: w="600" }


흐름의 이름은 `동아리지원서`, 흐름의 트리거는 **`새 응답이 제출되는 경우`**로 선택합니다. <br>
모두 선택 되었다면, `만들기` 버튼을 클릭해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/21.png){: w="600" }

우측 상단에 새 디자이너 토글 버튼은 `off`로 설정해야, 동일한 환경에서 실습이 가능합니다.

![img](/assets/img/2024-04-08-power-automate-handson/22.png){: w="600" }

### 1) [새 응답이 제출되는 경우] 폼 응답 트리거 설정
새 응답이 제출되는 경우 프로세스에서 양식 선택 토글 버튼을 눌러주고, <br>
아까 만든 동아리 지원 폼을 선택해줍시다.
![img](/assets/img/2024-04-08-power-automate-handson/23.png){: w="600" }

### 2) [응답 세부 정보 가져오기]
새 단계 버튼을 눌러 새로운 단계를 추가해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/24.png){: w="450" }

위 검색창에서 응답을 검색하여 응답 세부 정보 가져오기를 선택합니다.

![img](/assets/img/2024-04-08-power-automate-handson/25.png){: w="500" }

응답 세부 정보 가져오기 프로세스에서도 양식 ID 토글을 눌러 동아리 지원 폼을 선택해줍니다.

![img](/assets/img/2024-04-08-power-automate-handson/26.png){: w="600" }

응답 ID 칸도 빈칸을 클릭하여 오른쪽에 동적 콘텐츠 창이 뜨면, <br>
`응답 ID` 를 클릭하여 넣어줍니다.

![img](/assets/img/2024-04-08-power-automate-handson/27.png){: w="600" }

2번 과정을 잘 따라왔다면, 다음과 같이 세팅이 완료됩니다.

![img](/assets/img/2024-04-08-power-automate-handson/28.png){: w="600" }

### 3) [파일 복사] 지원서 원본 파일을 “복사” 폴더로 복사하기

새 단계 버튼을 클릭한 뒤 상단 검색창에 SharePoint를 검색하여 나온 SharePoint를 클릭해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/29.png){: w="600" }

상단의 검색창에서 파일 복사를 검색하여 파일 복사를 선택해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/30.png){: w="600" }


현재 사이트 주소 옆 토글 버튼을 클릭하여 <br>
`4/4 MLSA Session - userN (자신의 번호)`를 선택해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/31.png){: w="600" }

> 여기서 작업 환경(Teams)가 다르신 분들은 **지원서_원본 파일이 세팅된 채널**로 설정해주시면 됩니다.
{: .prompt-tip}

복사할 파일 필드는 옆에 폴더 아이콘 버튼을 클릭하면 <br>
오른쪽에 파일 선택창이 뜹니다. 여기서 Shared Documents 옆 `콘텐츠 표시 버튼(>)`을 눌러줍니다.

![img](/assets/img/2024-04-08-power-automate-handson/32.png){: w="600" }

`userN (자신의 번호)` 옆에 있는 `콘텐츠 표시 화살표 버튼`을 또 누르면 <br>
`지원서_원본.docx`라는 문서를 확인할 수 있고, 이 파일을 선택해주시면 됩니다.

|--|--|
|![img](/assets/img/2024-04-08-power-automate-handson/33.png)|![img](/assets/img/2024-04-08-power-automate-handson/34.png)|

대상 사이트 주소는 현재 사이트 주소와 동일하게 설정해주고, <br>
대상 폴더는 필드 옆 폴더 아이콘을 클릭하여 Shared Documents 옆 `콘텐츠 표시 버튼(>)`을 아까와 동일하게 눌러줍니다.

![img](/assets/img/2024-04-08-power-automate-handson/35.png){: w="600" }

userN으로 들어간 뒤, `복사` 폴더를 선택해줍니다.

|--|--|
|![img](/assets/img/2024-04-08-power-automate-handson/36.png)|![img](/assets/img/2024-04-08-power-automate-handson/37.png)|

`다른 파일이 이미 있는 경우`는 `Replace`를 선택해줍니다.

![img](/assets/img/2024-04-08-power-automate-handson/38.png){: w="600" }

3번 프로세스를 정상적으로 세팅하였다면, 다음과 같이 설정됩니다.

![img](/assets/img/2024-04-08-power-automate-handson/39.png){: w="600" }

### 4) [파일 속성 업데이트] 복사된 파일의 세부 정보를 폼에 입력된 정보에 맞게 업데이트하기

이젠 파일의 세부 정보를 업데이트 해보겠습니다. <br>
새 단계 버튼을 클릭한 뒤 상단 검색창에 SharePoint를 검색하여 나온 `SharePoint`를 클릭합니다.

![img](/assets/img/2024-04-08-power-automate-handson/4-1.png){: w="600" }

업데이트를 상단 검색창에 입력하거나, 스크롤을 내려서 `파일 속성 업데이트`를 찾아 선택합니다.

![img](/assets/img/2024-04-08-power-automate-handson/4-2.png){: w="600" }

사이트 주소 옆 토글 버튼을 클릭하여 `4/4 MLSA Session - userN (자신의 번호)`를 선택합니다.

![img](/assets/img/2024-04-08-power-automate-handson/4-3.png){: w="600" }

라이브러리 이름 옆 토글 버튼을 클릭하여 `문서`를 선택합니다.

![img](/assets/img/2024-04-08-power-automate-handson/4-4.png){: w="600" }

ID 필드는 클릭하여 오른쪽에 동적 콘텐츠 창이 뜨면 `파일 복사에 있는 ItemId`를 선택합니다.

![img](/assets/img/2024-04-08-power-automate-handson/4-5.png){: w="600" }

> 3번 프로세스(파일 복사)에서 복사된 파일의 세부 정보를 수정해야하기 때문에 <br>
> **파일 복사에 있는 ItemId**를 선택해주어야 합니다!
{: .prompt-info}

순서대로
- 지원자 이름 필드에는 이름을 입력하세요,
- 지원자 학과 필드에는 학과을 입력하세요,
- 지원자 학번 필드에는 학번을 입력하세요,
- 지원 동기 필드에는 지원 동기를 입력하세요 
를 선택해줍니다.

모두 선택이 완료되면 다음과 같이 설정이 됩니다. <br>
(참고로 아래 사진에는 제목 필드에 무언가 설정해두었지만.. 무시해주시고, 이름, 학과, 학번 필드만 제대로 채우면 됩니다.)

![img](/assets/img/2024-04-08-power-automate-handson/4-6.png){: w="600" }

### 5) [파일 콘텐츠 가져오기] 세부 정보가 업데이트된 파일의 콘텐츠를 가져오기

다음으로 파일 콘텐츠를 가져와보겠습니다. <br>
동일하게 새 단계 버튼을 클릭하고 상단 검색창에 SharePoint를 검색하여 나온 SharePoint를 클릭해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/40.png){: w="600" }

상단의 검색창에서 콘텐츠를 검색하고, 파일 콘텐츠 가져오기를 선택해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/41.png){: w="450" }

사이트 주소는 이전과 동일하게 <br>
`4/4 MLSA Session - userN (자신의 번호)`로 선택해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/42.png){: w="600" }

파일 식별자 필드는 `파일을 선택합니다.` 부분을 클릭하면 오른쪽에 `동적 콘텐츠` 창이 뜹니다. <br>
스크롤을 내려서 `파일 복사 부분의 Id`를 선택해주시면됩니다.

> 여기서 ItemId가 아닌 Id를 선택해야하는 것에 유의해주세요!
{: .prompt-warning}

![img](/assets/img/2024-04-08-power-automate-handson/43.png){: w="600" }

모든 세팅이 완료되면 다음과 같이 설정됩니다.

![img](/assets/img/2024-04-08-power-automate-handson/44.png){: w="600" }

### 6) [파일 만들기] OneDrive에서 Word 파일 생성하기

> **😵 이 때, 왜 하필 OneDrive에서 Word 파일을 생성하나요?** <br>
> 추후 Word파일을 PDF 파일로 변경 하기 위해 OneDrive에서 Word 파일을 생성합니다! <br>
> PDF 파일 변경은 다른 간단한 방법도 있지만, 학교 계정이 power automate 프리미엄 권한이 없기 때문에, <br>
> OneDrive내에서 Word파일 생성 후, PDF로 변환하는 방식을 채택하였습니다.
{: .prompt-info}

다시 새 단계 버튼을 누르고, 상단에 OneDrive를 검색해줍시다. <br>
`OneDrive for Business` 버튼을 선택해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/45.png){: w="600" }

상단 검색창에 만들기 라고 검색하고, `파일 만들기`를 선택해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/46.png){: w="600" }

폴더 경로는 `Root`로 설정합니다.
- 나중에 생성된 파일은 OneDrive에서 지우기 용이하게 설정하기 위해 Root로 설정하였습니다.

![img](/assets/img/2024-04-08-power-automate-handson/47.png){: w="600" }

파일 이름은 **`학번을 입력하세요`_`이름을 입력하세요`.docx**로 설정해줍니다.
> 언더바와 끝에 .docx 확장자 작성에 유의해주세요!
{: .prompt-warning}

![img](/assets/img/2024-04-08-power-automate-handson/48.png){: w="600" }

파일 콘텐츠는 `“파일 콘텐츠 가져오기”의 파일 콘텐츠`를 선택해줍시다.
- 5번째 프로세스(파일 콘텐츠 가져오기)에서 가져온 파일 세부 정보가 업데이트 된 파일의 내용을 가져오겠다는 의미입니다!

![img](/assets/img/2024-04-08-power-automate-handson/49.png){: w="600" }

### 7) [파일 변환] Word파일을 PDF 파일로 변환하기

다시 새 단계 버튼을 누르고, 상단에 OneDrive를 검색해줍시다. <br>
`OneDrive for Business`버튼을 선택합니다.

![img](/assets/img/2024-04-08-power-automate-handson/45.png){: w="600" }

상단 검색창에 변환이라고 검색하고, `파일 변환(미리보기)`를 선택합니다.

![img](/assets/img/2024-04-08-power-automate-handson/50.png){: w="600" }

파일 필드에서는 이전에 만든 파일을 pdf 로 변환할 예정이므로, <br>
**파일 만들기에 있는 `Id`**를 선택해줍니다.
- 대상 유형은 `PDF`로 설정합니다.

![img](/assets/img/2024-04-08-power-automate-handson/51.png){: w="600" }

### 8) [파일 만들기] PDF로 변환된 파일을 “문서”폴더에 저장하기

이제 정말 마지막입니다. <br>
OneDrive에서 PDF로 변환된 파일을 다시 Teams로 옮겨봅시다.

동일하게 새 단계 버튼을 클릭하고 상단 검색창에 SharePoint를 검색하여 나온 SharePoint를 클릭해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/52.png){: w="600" }

상단 검색창에서 만들기를 검색하여 `파일 만들기`를 선택해줍니다.

![img](/assets/img/2024-04-08-power-automate-handson/53.png){: w="600" }

사이트 주소는 `4/4 MLSA Session userN (자신의 번호)`로 설정해줍니다.

![img](/assets/img/2024-04-08-power-automate-handson/54.png){: w="600" }

폴더 경로는 옆에 `폴더 아이콘`을 누른 뒤, Shared Documents 옆 화살표 모양의 `콘텐츠 표시` 버튼을 눌러주고,

![img](/assets/img/2024-04-08-power-automate-handson/55.png){: w="600" }

user1 으로 들어가 문서 폴더를 클릭해줍니다.

|--|--|
|![img](/assets/img/2024-04-08-power-automate-handson/56.png)|![img](/assets/img/2024-04-08-power-automate-handson/57.png)|


파일 이름과 파일 콘텐츠는 각각 7번째 프로세스인 "파일 변환"의 `파일 이름`과 `파일 콘텐츠`로 설정해줍시다.

![img](/assets/img/2024-04-08-power-automate-handson/58.png){: w="600" }

## 워크플로 테스트

### 저장하기
이렇게 간단한 워크플로가 생성이 되었습니다. 저장 버튼을 눌러 저장해줍시다.
![img](/assets/img/2024-04-08-power-automate-handson/59.png){: w="600" }

### 테스트 설정
자 이제 테스트를 해보겠습니다. <br>
상단의 테스트 버튼을 누르고 수동 설정을 해준 뒤, 테스트 버튼을 누릅시다.
![img](/assets/img/2024-04-08-power-automate-handson/60.png){: w="600" }

### 폼 응답 제출
다시, 아까 작업하던 폼 화면으로 돌아가서 미리 보기 버튼을 클릭합니다.

![img](/assets/img/2024-04-08-power-automate-handson/61.png){: w="600" }

미리 보기 화면으로 넘어가면 다음과 같이 폼을 작성할 수 있는데, <br>
수동 테스트를 위해 폼을 작성하고, 제출해봅시다.

![img](/assets/img/2024-04-08-power-automate-handson/62.png){: w="600" }

응답을 제출하고, 다시 테스트 화면으로 돌아오면 워크플로우가 정상적으로 진행이 되고 있는지 확인할 수 있습니다.

조금 기다리면 정상적으로 실행이 완료되며, <br>
자신의 문서 폴더에 들어가면 정상적으로 생성된 pdf 파일도 볼 수 있습니다.

실습이 전부 끝나면 OneDrive에서 실습 파일을 지우고 마무리하면됩니다.