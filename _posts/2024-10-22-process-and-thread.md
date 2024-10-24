---
title: 프로세스와 스레드
date: 2024-10-22 18:20:00 +/-TTTT
categories: [Java, Thread]
tags: [os, process, thread]     # TAG names should always be lowercase
---

# 멀티태스킹과 멀티프로세싱

## 멀티태스킹

> CPU 코어가 **한개**라고 가정해보자.

### 시분할(Time Sharing, 시간 공유) 기법

프로그램A를 먼저 수행하고, 프로그램B를 수행하는 것이 아닌 <br>
각 프로그램의 실행 시간을 분할해서 마치 동시에 실행되는 것 처럼 하는 기법이다.

⇒ 현대의 CPU는 초당 수십업번 이상의 연산을 수행할 수 있기 때문!

이렇게 하나의 컴퓨터 시스템이 동시에 여러 작업을 수행하는 능력을 **멀티태스킹** 이라고 한다.

> <U>CPU에 어떤 프로그램이 얼마만큼 실행될지</U>는 운영체제가 결정하는데 이것은 **스케줄링(Scheduling)**이라고 함!
{: .prompt-info }

## 멀티프로세싱
> CPU에 두 개의 코어가 들어있다고 가정해보자.

CPU 코어가 2개일때는 **물리적**으로 "완전히" 동시에 2개의 프로그램을 처리할 수 있다.

이렇게 컴퓨터 시스템에서 둘 이상의 프로세서(CPU 코어)를 사용하여 여러 작업을 동시에 처리하는 기술을 **멀티프로세싱**이라고 한다.

<br>

|------|---|
|![img](/assets/img/2024-10-22-process-and-thread/0.png)|![img](/assets/img/2024-10-22-process-and-thread/1.png)|

만약 여기서 하나의 프로세서가 시분할기법을 통해 하나의 프로그램A만 수행하는 것이 아닌, 여러개의 프로그램(A, B, C)을 돌아가면서 수행한다면, 실제로는 2개보다 더 많은 프로그램을 실행할 수 있다. <br>
(물론 코어 1개일 때보다 속도는 빨라짐!)

이런 경우를 여러 CPU 코어를 사용하기 때문에 **멀티프로세싱**이며, 동시에 각각의 CPU 코어에 여러 작업을 분할하여 수행하기 때문에 **멀티태스킹**이라고도 말할 수 있는 것이다.


## 멀티태스킹 vs 멀티프로세싱

**멀티프로세싱**
- 여러 CPU(여러 CPU 코어)를 사용하여 동시에 여러 작업을 수행하는 것을 의미함.
- **(물리적인) 하드웨어 기반**으로 성능을 향상시킴.
- 예: 다중 코어 프로세서를 사용하는 현대 컴퓨터 시스템

**멀티태스킹**
- 단일 CPU(단일 CPU 코어)가 여러 작업을 동시에 수행하는 것처럼 보이게 하는 것. 
- **소프트웨어 기반**으로 CPU 시간을 분할하여 각 작업에 할당함.
- 예: 현대 운영 체제에서 여러 애플리케이션이 동시에 실행되는 환경

# 프로세스와 스레드

![img](/assets/img/2024-10-22-process-and-thread/2.png)

> 프로세스엔 코드/데이터/힙 등의 메모리와 하나 이상의 스레드로 구성되어 있으며, <br>
> 스레드는 프로세스가 제공하는 메모리(그림의 초록색 부분)를 공유한다. <br>
> 스레드는 각각의 스택(그림의 노란색)을 관리한다. (또한, 다른 스레드의 스택에 접근할 수 없다.)
{: .prompt-tip}

## 프로세스

프로그램을 실행하면 프로세스가 만들어지고 프로그램이 실행된다. 
이렇게 **운영체제 안에서 실행중인 프로그램**을 **프로세스**라고 한다.

프로세스는 실행 중인 프로그램의 **인스턴스**이다.

- 프로세스는 메모리 공간, 파일 핸들, 시스템 자원(네트워크 연결) 등의 실행 환경을 제공한다. 
- 즉 **컨테이너 역할**을 한다는 것!

### 프로세스의 특징
- 각 프로세스는 독립적인 메모리 공간을 가진다. 
<br>따라서 서로 간섭하지 않으며 서로의 메모리에 직접 접근할 수 없다.
- 운영체제에서 별도의 작업 단위로 분리해서 관리된다.

격리되어 관리되기 때문에 하나의 프로세스가 충돌해도 다른 프로세스에 영향을 미치지 않는다. <br> 
따라서 특정 프로세스에 에러가 발생하면 해당 프로세스만 종료되고, 다른 프로세스에 영향을 주지 않는다.

### 프로세스의 메모리 구성
- **코드 섹션**: 실행할 프로그램의 코드가 저장되는 부분
- **데이터 섹션**: 전역 변수 및 정적 변수가 저장되는 부분
- **힙 (Heap)**: 동적으로 할당되는 메모리 영역
- **스택 (Stack)**: 메서드(함수) 호출 시 생성되는 지역 변수와 반환 주소가 저장되는 영역
    - **스레드**가 스택을 관리함.

## 스레드
> 프로세스는 하나 이상의 스레드를 반드시 포함한다. <br>
> 그래야 프로그램이 실행될 수 있기 때문!
{: .prompt-info}

**스레드**는 **프로세스 내에서 실행되는 작업의 단위 (실제 CPU에 의해 실행되는 단위)**이다.
- 한 프로세스 내에서 **여러 스레드가 존재**할 수 있으며, 이들은 프로세스가 제공하는 **동일한 메모리 공간(코드, 힙 등)을 공유**한다.
- 스레드는 프로세스보다 단순하므로 <U>생성 및 관리가 단순하고 가볍다</U>.

여기서 한 프로세스 내에 하나의 스레드만 존재하는 것을 **단일 스레드**, <br>
한 프로세스 내에 여러 스레드가 존재하는 것을 **멀티 스레드**라고 한다.

### 스레드의 특징
- **공유 메모리**: 같은 프로세스의 <U>코드 섹션, 데이터 섹션, 힙(메모리)</U>은 프로세스 안의 모든 스레드가 **공유**한다.
- **개별 스택**: 각 스레드는 <U>자신의 스택을 갖고 있다</U>. 또한 다른 스레드의 스택에 간섭할 수 없다.

### 프로그램의 실행
1. 프로그램을 실행한다.
2. 운영체제는 먼저 디스크에 있는 파일 덩어리인 프로그램을 메모리로 불러오면서 프로세스를 만든다.
3. 프로세스 안에 있는 코드가 한 줄씩 실행된다.

여기서 코드를 하나씩 실행하면서 내려가는 것이 바로 **"스레드"**!

> 프로세스는 실행 환경과 자원을 제공하는 **컨테이너 역할**을 하고, <br>
> 스레드는 CPU를 사용해서 **코드를 하나하나 실행**한다.
{: .prompt-tip}

### 멀티스레드

> 왜 필요할까?
하나의 프로그램도 그 안에서 동시에 여러 작업이 필요하다.
- ex) 유튜브로 영상을 보면서 댓글을 동시에 작성한다.

# 스레드와 스케줄링

> CPU 코어가 스레드를 실행하는 과정을 살펴보자.

![img](/assets/img/2024-10-22-process-and-thread/3.png){: w="500" }

1. 프로세스A에 있는 A1을 실행한다.
2. A1의 실행을 잠시 멈추고 프로세스B에 있는 스레드 B1을 실행한다.
3. B1의 실행을 잠시 멈추고 프로세스B에 있는 스레드 B2를 실행한다.
4. 이 과정을 반복한다.

## 단일/멀티 코어 스케줄링

### 단일 코어 스케줄링

![img](/assets/img/2024-10-22-process-and-thread/4.png){: w="500" }

운영체제는 내부에 스케줄링 큐를 가지고 있으며, 실행해야 하는 스레드가 스케줄링 큐에서 대기한다.
- 그림에서는 스레드A1, B1, B2가 스케줄링 큐에서 대기하고 있다.

![img](/assets/img/2024-10-22-process-and-thread/5.png){: w="500" }

운영체제는 큐에 가장 앞에 있는 스레드A1을 큐에서 꺼내고 CPU를 통해 실행한다. <br>
그리고 A1을 잠시 멈추고, 스케줄링 큐에 다시 넣는다.

![img](/assets/img/2024-10-22-process-and-thread/6.png){: w="500" }

다시 가장 앞에 있는 스레드B1을 큐에서 꺼내서 CPU를 통해 실행한 뒤, 다시 멈추고 스케줄링 큐에 넣는다.

이 과정을 반복하여 수행한다.

### 멀티 코어 스케줄링

![img](/assets/img/2024-10-22-process-and-thread/7.png){: w="500" }

멀티 코어 스케줄링의 경우에도, 스케줄링 큐에서 대기하고 있는 스레드를 코어가 각각 꺼내서 병렬로 실행한다.

그리고 단일 코어와 동일하게 스레드 수행을 잠시 멈추고, 스케줄링 큐에 다시 넣은 뒤에 대기 중인 다른 스레드를 꺼내서 CPU를 통해 실행한다.

# 컨텍스트 스위칭

> 멀티스레딩은 대부분 효율적이다. <br>
> 하지만 컨텍스트 스위칭 과정이 필요하므로 ✌️항상✌️ 효율적인 것은 아니다!
{: .prompt-tip}

운영체제가 스레드A를 작업하고, 잠시 멈춘뒤, 스레드B를 작업한다고 가정하자.<br>
스레드B를 작업한 뒤에 다시 스레드A로 돌아가고자 한다면 바로 작업을 시작할 수 있을까?

스레드A가 어디까지 수행되었는지, 계산하던 변수들의 값은 무엇인지 등 **스레드A를 멈춘 시점에 CPU에서 사용한 값들을 메모리에 저장해두어야**한다. <br>
그리고 다시 해당 스레드A를 실행할 때 **메모리에 저장한 값을 CPU에 다시 불러와야**한다.

이런 과정을 **컨텍스트 스위칭**이라고 한다.

---

<br>

> 이 포스트는 ["김영한의 실전 자바 - 고급 1편, 멀티스레드와 동시성"](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-1) 강의를 듣고 작성하였습니다!
{: .prompt-info}