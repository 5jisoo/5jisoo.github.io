---
title: 스레드 제어와 생명 주기
date: 2024-10-25 20:16:00 +/-TTTT
categories: [Java, Thread]
tags: [java, thread, lifecycle]     # TAG names should always be lowercase
image:
  path: /assets/img/2024-10-25-thread-control-and-lifecycle/0.png
  alt: <스레드의 제어와 생명 주기> 썸네일
---

## 스레드 기본 정보

```java
public class ThreadInfoMain {
    public static void main(String[] args) {
        Thread mainThread = Thread.currentThread();
        log("mainThread = " + mainThread);
        log("mainThread.threadId() = " + mainThread.threadId());
        log("mainThread.getName() = " + mainThread.getName());
        log("mainThread.getPriority() = " + mainThread.getPriority());
        log("mainThread.getThreadGroup() = " + mainThread.getThreadGroup());
        log("mainThread.getState() = " + mainThread.getState());

        Thread myThread = new Thread(new HelloRunnable(), "myThread");
        log("mainThread = " + myThread);
        log("mainThread.threadId() = " + myThread.threadId());
        log("mainThread.getName() = " + myThread.getName());
        log("mainThread.getPriority() = " + myThread.getPriority());
        log("mainThread.getThreadGroup() = " + myThread.getThreadGroup());
        log("mainThread.getState() = " + myThread.getState());
    }
}
```

```
21:07:01.745 [     main] mainThread = Thread[#1,main,5,main] // 스레드 정보 출력
21:07:01.752 [     main] mainThread.threadId() = 1 // 스레드 아이디
21:07:01.752 [     main] mainThread.getName() = main // 스레드 이름
21:07:01.755 [     main] mainThread.getPriority() = 5 // 스레드 우선순위
21:07:01.755 [     main] mainThread.getThreadGroup() = java.lang.ThreadGroup[name=main,maxpri=10] // 스레드 그룹
21:07:01.755 [     main] mainThread.getState() = RUNNABLE // 스레드 상태
21:07:01.756 [     main] mainThread = Thread[#21,myThread,5,main]
21:07:01.756 [     main] mainThread.threadId() = 21
21:07:01.756 [     main] mainThread.getName() = myThread
21:07:01.757 [     main] mainThread.getPriority() = 5
21:07:01.757 [     main] mainThread.getThreadGroup() = java.lang.ThreadGroup[name=main,maxpri=10]
21:07:01.758 [     main] mainThread.getState() = NEW
```

### 스레드 출력
**`thread.toString()`** : 스레드 ID, 스레드 이름, 우선순위, 스레드 그룹을 반환한다.

### 스레드 아이디
**`thread id`** : 자바에서 자동 생성하여 부여한다. 또한 JVM 내에서 다른 스레드 아이디와 중복되지 않는다.
- 스레드 아이디는 직접 설정하는 것이 불가능하다.

### 스레드 이름
**`name`**
스레드 이름. 설정하지 않으면 자동으로 부여되며, 이름은 중복될 수 있다.

### 스레드 우선순위
**`priority`** 
(default : 5) : 우선순위 
- `setPriority()` 메서드를 사용하여 1(가장 낮음)부터 10(가장 높음)까지 설정할 수 있으며 우선순위가 높을 수록 더 많이 실행된다.
- 운영체제 스케줄링에 힌트를 주지만 우선순위가 높다고 **항상 더 많이 실행되는 것은 아님**. (운영체제마다 상이함)
- 보통 운영체제에서 알아서 조정하기 때문에, 자바 백엔드 개발할 때 직접 조정하는 일은 거의 없음. 

### 스레드 그룹
**`thread group`** : 스레드가 속한 그룹
- 스레드 그룹은 스레드를 그룹화하여 관리할 수 있는 기능을 제공한다.
    - 기본적으로 모든 스레드는 부모 스레드와 동일한 스레드 그룹에 속하며, 특정 작업(예: 일괄 종료, 우선순위 설정 등)을 그룹단위로 수행할 수 있다.
- **부모 스레드(Parent Thread)**: 새로운 스레드를 생성하는 스레드를 의미한다. <br> 스레드는 기본적으로 다른 스레드에 의해 생성된다. 이러한 생성 관계에서 새로 생성된 스레드는 생성한 스레드를 **부모**로 간주한다. 
    - 예를 들어 `myThread` 는 `main` 스레드에 의해 생성되었으므로 `main` 스레드가 부모 스레드!
- `main` 스레드는 기본으로 제공되는 `main` 스레드 그룹에 소속되어 있다. 따라서 `myThread` 도 부모 스레드인 `main` 스레드의 그룹인 `main` 스레드 그룹에 소속된다. 

### 스레드의 현재 상태
**`state`** : 스레드의 현재 상태
- `NEW`: 스레드가 아직 시작되지 않은 상태이다.
- `RUNNABLE`: 스레드가 실행 중이거나 실행될 준비가 된 상태이다. 
- `BLOCKED`: 스레드가 동기화 락을 기다리는 상태이다.
- `WAITING`: 스레드가 다른 스레드의 특정 작업이 완료되기를 기다리는 상태이다. 
- `TIMED_WAITING`: 일정 시간 동안 기다리는 상태이다.
- `TERMINATED`: 스레드가 실행을 마친 상태이다.

## 스레드의 생명 주기

![img](/assets/img/2024-10-25-thread-control-and-lifecycle/0.png){: w="600" }
_스레드의 상태_

> 자바에서 스레드의 **일시 중지 상태(Suspended State)**라는 상태는 없다. 스레드가 기다리는 상태들을 묶어서 쉽게 설명하기 위해 사용한 용어임!

<table>
    <tbody>
        <tr>
            <td rowspan="2"></td>
            <td>NEW 새로운 상태 : 생성되었으나, 시작되지 않은 상태</td>
        </tr>
        <tr>
            <td>Runnable 실행 가능한 상태 : 실행 중이거나 실행될 준비가 된 상태</td>
        </tr>
        <tr>
            <td rowspan="3">일시 중지 상태 <br> (Suspended State)</td>
            <td>Blocked 차단 상태 : 동기화 락을 기다리는 상태</td>
        </tr>
        <tr>
            <td>Waiting 대기 상태 : 다른 스레드의 작업을 무기한으로 기다리는 상태</td>
        </tr>
        <tr>
            <td>Time Waiting 시간 제한 대기 상태: 일정 시간 동안 다른 스레드의 작업을 기다리는 상태</td>
        </tr>
        <tr>
            <td></td>
            <td>Terminated 종료 상태: 실행이 완료된 상태</td>
        </tr>
    </tbody>
</table>

### New 새로운 상태

생성되고 아직 시작되지 않은 상태
- 이 상태에서는 `Thread` 객체가 생성되지만, `start()` 메서드가 호출되지 않은 상태

### Runnable 실행 가능 상태 (실행 상태)

스레드가 실행될 준비가 된 상태. 이 상태에서 스레드는 실제로 CPU에서 실행될 수 있다.
- `start()` 메서드가 호출되었을 때

Runnable 상태에 있는 스레드가 모두 동시에 실행되는 것이 아니다. 스케줄러의 실행 대기열에 포함되어 있다가 차례대로 CPU에서 실행한다.
- 운영체제의 스케줄러의 실행 대기열에 있는 상태와 현재 실행되고 있는 상태 모두 `RUNNABLE` 상태이다.
    - 자바에서 이 두 상태를 구분할 수는 없다.


### Blocked 차단 상태

스레드가 다른 스레드에 의해 동기화 락을 얻기 위해 기다리는 상태이다.

- 예를 들어, `synchronized` 블록에 진입하기 위해 **락을 얻어야 하는 경우** 이 상태에 들어간다.


### Waiting 대기 상태

스레드가 다른 스레드의 특정 작업이 완료되기를 무기한 기다리는 상태이다.
- `wait()` , `join()` 메서드가 호출될 때

스레드는 다른 스레드가 `notify()` 또는 `notifyAll()` 메서드를 호출하거나, `join()` 이 완료될 때까지 기다린다.

### Timed Waiting (시간 제한 대기 상태)

스레드가 특정 시간 동안 다른 스레드의 작업이 완료되기를 기다리는 상태이다.
- `sleep(long millis)` , `wait(long timeout)` , `join(long millis)` 메서드가 호출될 때

주어진 시간이 경과하거나 다른 스레드가 해당 스레드를 깨우면 이 상태에서 벗어난다.

### Terminated (종료 상태)

스레드의 실행이 완료된 상태이다.
- 스레드가 정상적으로 종료되었거나 예외가 발생하여 종료된 경우
- 스레드는 한 번 종료되면 **다시 시작할 수 없다**.

### 코드

```java
package thread.control;

import static util.MyLogger.log;

public class ThreadStateMain {
    public static void main(String[] args) throws InterruptedException {
        Thread myThread = new Thread(new MyRunnable(), "myThread");
        log("myThread.state1 = " + myThread.getState()); // NEW
        log("myThread.start()");
        myThread.start();
        Thread.sleep(1000);
        // myThread TIMED_WAITING 상태 -> 자신의 상태 출력 불가능
        // main 스레드에서 상태를 출력
        log("myThread.state3 = " + myThread.getState()); // TIMED_WAITING
        Thread.sleep(4000);
        log("myThread.state5 = " + myThread.getState()); // TERMINATED
        log("end");
    }

    static class MyRunnable implements Runnable {

        @Override
        public void run() {
            try {
                log("start");
                log("myThread.state2 = " + Thread.currentThread().getState()); // RUNNABLE
                log("sleep() start");
                Thread.sleep(3000);
                log("sleep() end");
                log("myThread.state4 = " + Thread.currentThread().getState()); // RUNNABLE
                log("end");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}

```
_소스코드_

```
22:56:54.248 [     main] myThread.state1 = NEW
22:56:54.251 [     main] myThread.start()
22:56:54.251 [ myThread] start
22:56:54.252 [ myThread] myThread.state2 = RUNNABLE
22:56:54.252 [ myThread] sleep() start
22:56:55.257 [     main] myThread.state3 = TIMED_WAITING
22:56:57.257 [ myThread] sleep() end
22:56:57.259 [ myThread] myThread.state4 = RUNNABLE
22:56:57.259 [ myThread] end
22:56:59.260 [     main] myThread.state5 = TERMINATED
22:56:59.261 [     main] end
```
_실행 결과_

```
22:56:55.257 [     main] myThread.state3 = TIMED_WAITING
```

위와 같이 main에서 myThread의 상태를 찍은 이유는 무엇일까?

- myThread는 `sleep()`때문에 `TIMED_WAITING` 상태이다. <br>
따라서 다음과 같이 myThread에서 sleep() 이후 상태를 출력하고자 하면, **`TIMED_WAITING` 상태에선 대기 상태이기 때문에 출력이 불가능**하고, 출력 결과는 이후에 대기 상태가 끝난 뒤 `RUNNABLE` 상태에서 상태를 출력하므로, 정확한 상태 출력이 불가능하다.
    ```java
    log("sleep() start");
    Thread.sleep(3000);
    log("myThread.state3 = " + Thread.currentThread().getState()); // RUNNABLE 이 출력됨.
    log("sleep() end");
    ```

## 체크 예외 재정의
```java
 public interface Runnable {
     void run();
}
```


## join


---

<br>

> 이 포스트는 ["김영한의 실전 자바 - 고급 1편, 멀티스레드와 동시성"](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-1) 강의를 듣고 작성하였습니다! <br>
> 실습에 사용된 모든 코드는 [이곳](https://github.com/5jisoo/Java-Study)에서 확인할 수 있습니다.
{: .prompt-info}