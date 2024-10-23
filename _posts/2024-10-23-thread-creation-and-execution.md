---
title: 스레드의 생성과 실행
date: 2024-10-23 17:30:00 +/-TTTT
categories: [Java, Thread]
tags: [java, thread]     # TAG names should always be lowercase
image:
  path: /assets/img/2024-10-23-thread-creation-and-execution/2.png
  alt: 스레드의 생성과 실행 썸네일
---

## 스레드 생성 방법

스레드를 생성하는 방법에는 두가지가 존재한다.

1. `Thread`를 상속받는 방법
2. `Runnable` 인터페이스를 구현하는 방법

## 스레드 생성 - `Thread` 상속

```java
public class HelloThread extends Thread {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": run()");
    }
}
```
_HelloThread 클래스_

- `Thread` 클래스를 상속하고, 스레드가 실행할 코드를 run() 메소드에 재정의(override)함.
- `Thread.currentThread()` : 해당 코드를 실행하는 스레드 객체 조회

<br>

```java
public class HelloThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        HelloThread helloThread = new HelloThread();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 전");
        helloThread.start();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}
```
_main_

```
main: main() start
main: start() 호출 전
main: start() 호출 후
main: main() end
Thread-0: run()
```
```
main: main() start
main: start() 호출 전
main: start() 호출 후
Thread-0: run()
main: main() end
```
_실행 결과_
> 실행 결과는 스레드의 실행 순서에 따라 다를 수 있음.

- `HelloThread`라는 스레드 객체를 생성하고 `start()` 메서드를 호출함
- `start()`를 호출하면 `HelloThread`가 **`run()`**을 실행함.

> `run()`이 아니라 반드시 **`start()`**를 호출해야 함. 그래야 <u><b>별도의 스레드</b>에서 run() 코드가 실행</u>됨.
{: .prompt-warning}

## 실행 결과 분석

### 스레드 생성 전

![img](/assets/img/2024-10-23-thread-creation-and-execution/0.png){: w="800" }

- `main()` 메서드는 `main` 이라는 이름의 스레드가 실행함.

프로세스가 작동하기 위해 최소한 하나의 스레드가 필요하므로, <br>
자바는 실행 시점에 `main`이라는 스레드를 만들고 프로그램의 시작점인 `main()` 메서드를 실행함.

### 스레드 생성 후

![img](/assets/img/2024-10-23-thread-creation-and-execution/1.png){: w="800" }

1. `HelloThread` 스레드 객체를 생성한 다음에 `start()` 메서드를 호출하면 자바는 스레드를 위한 별도의 스택 공간을 할당함.
2. 스레드 객체를 생성하고, 반드시 **`start()` 를 호출**해야 **스택 공간을 할당 받고 스레드가 작동**함.
3. 스레드에 이름을 주지 않으면 자바는 스레드에 `Thread-0` , `Thread-1` 과 같은 임의의 이름을 부여함.
4. 새로운 `Thread-0` 스레드가 사용할 전용 스택 공간이 생김.
5. `Thread-0` 스레드는 <u>`run()` 메서드의 스택 프레임을 스택에 올리면서 `run()` 메서드를 시작</u>함.

### 호출 시간 확인

> `main` 스레드가 `run()` 메서드를 실행하는게 아니라 `Thread-0` 스레드가 `run()` 메서드를 실행한다. <br>
> 따라서 `main` 스레드는 `run()` 메서드의 실행이 끝나는 것을 기다리지 않는다!
{: .prompt-tip}

![img](/assets/img/2024-10-23-thread-creation-and-execution/2-1.png){: w="600" }

- `main` 스레드가 `HelloThread` 인스턴스를 생성. 
    - 이때 스레드에 이름을 부여하지 않으면 자바가 `Thread-0` , `Thread-1` 과 같은 임의의 이름을 부여함.
- `start()` 메서드를 호출하면, `Thread-0` 스레드가 시작되면서 `Thread-0` 스레드가 `run()` 메서드를 호출함.
    - 여기서 핵심은 **`main` 스레드가 `run()` 메서드를 실행하는게 아니라 `Thread-0` 스레드가 `run()` 메서드를 실행한다는 점**이다.
- `main` 스레드는 단지 `start()` 메서드를 통해 `Thread-0` 스레드에게 실행 지시만 내림.
- 이제 `main` 스레드와 `Thread-0` 스레드는 **동시에 실행**된다.
- `main` 스레드 입장에서 보면 **그림의 1, 2, 3번 코드를 멈추지 않고 계속 수행**한다. <br> **그리고 `run()` 메서드는 `main` 이 아닌 별도의 스레드에서 실행**된다.

### 스레드간 실행 순서는 보장되지 않는다!

스레드는 **동시에 실행**되기 때문에 **스레드 간 실행 순서는 얼마든지 달라질 수 있다.** 

따라서 아래와 같은 실행결과가 모두 나올 수 있다.

1. `main` 스레드가 빨리 실행된 경우
    ```
    main: main() start 
    main: start() 호출 전 
    main: start() 호출 후 
    main: main() end 
    Thread-0: run()
    ``` 
    `main` 스레드가 모든 로직을 다 수행한 뒤에 `Thread-0`이 수행됨.
2. `Thread-0` 스레드가 빨리 실행된 경우
    ```
    main: main() start 
    main: start() 호출 전 
    Thread-0: run() 
    main: start() 호출 후 
    main: main() end
    ```
    `Thread-0` 호출 지시 직후 `Thread-0`가 먼저 수행되고, `main`이 수행됨.
3. `main` 스레드 실행 중간에 `Thread-0` 스레드가 실행된 경우
    ```
    main: main() start 
    main: start() 호출 전
    main: start() 호출 후 
    Thread-0: run()
    main: main() end
    ```

> 스레드는 순서와 실행 기간을 모두 보장하지 않는다!
{: .prompt-tip}

## `start()` 대신 `run()`을 호출하기

```java
public class BadThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        HelloThread helloThread = new HelloThread();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 전");
        helloThread.run(); // run() 직접 실행
        System.out.println(Thread.currentThread().getName() + ": start() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}
```
_`run()`을 호출한 경우_

```
main: main() start
main: start() 호출 전
main: run()
main: start() 호출 후
main: main() end
```
_실행결과_

- 실행 결과를 잘 보면 별도의 스레드가 `run()` 을 실행하는 것이 아닌, `main` 스레드가 `run()` 메서드를 호출함.

![img](/assets/img/2024-10-23-thread-creation-and-execution/3.png){: w="800" }

- `main` 스레드는 `HelloThread` 인스턴스에 있는 `run()` 이라는 메서드를 호출한다.
- `main` 스레드가 `run()` 메서드를 실행했기 때문에 `main` 스레드가 사용하는 스택위에 `run()` 스택 프레임이 올라감.
    - 그러므로 `main` 스레드에서 모든 것을 처리하였고, (일반 메서드를 호출한 것과 다름이 없음.) <br> **`Thread-0`는 아무일도 하지 않는다**.

## 데몬 스레드

스레드의 종류: 사용자(user) 스레드 / 데몬(daemon) 스레드

### 사용자 스레드
> non-daemon 스레드

- 프로그램의 주요 작업을 수행함.
- 작업이 완료될 때까지 실행됨.
- 모든 사용자(user) 스레드가 종료되면 JVM도 종료.
    
### 데몬 스레드

사용자에게 직접적으로 보이지 않으면서 시스템의 백그라운드에서 작업을 수행하는 것을 **데몬 스레드, 데몬 프로세스**라 한다.

ex) 사용하지 않는 파일이나 메모리를 정리하는 작업

- 백그라운드에서 보조적인 작업을 수행.
- 모든 사용자(user) 스레드가 종료되면 데몬 스레드는 자동으로 종료.

> JVM은 데몬 스레드의 실행 완료를 기다리지 않고 종료된다. <br> 
> 데몬 스레드가 아닌 모든 스레드가 종료되면, 자바 프로그램도 종료된다. <br> 
> - ex) 위 예제에서 `main` 스레드가 종료된 후에도 아직 `Thread-0` 스레드가 수행중이라면 JVM은 종료되지 않는다. 이후 `Thread-0`가 종료되면 그 때 JVM이 종료된다.
{: .prompt-info}

### 데몬 스레드 생성

```java
public class DaemonThreadMain {

    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        DaemonThread daemonThread = new DaemonThread();
        daemonThread.setDaemon(true); // daemon thread 여부
        daemonThread.start();

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }

    static class DaemonThread extends Thread {

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ": run()");
            try {
                Thread.sleep(10000); // 10초간 실행
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println(Thread.currentThread().getName() + ": run() end");
        }
    }
}
```
_소스 코드_

- `setDaemon(true)` : 데몬 스레드로 설정 (기본값 : false)

```
main: main() start
main: main() end
Thread-0: run()

Process finished with exit code 0
```
_실행 결과_

- 유일한 user 스레드인 `main` 스레드가 종료되면서 자바 프로그램도 종료됨. 
    - 자바 프로그램은 데몬 스레드를 기다리지 않기 때문!
- 따라서 데몬 스레드의 `run() end` 가 출력되기 전에 프로그램이 종료됨.

### 만약 데몬 스레드가 아닌 사용자 스레드라면?

> `setDaemon(false)` 로 설정하고 실행

```
main: main() start
main: main() end
Thread-0: run()
Thread-0: run() end

Process finished with exit code 0
```
_실행 결과_

- `main` 스레드가 종료되어도, user 스레드인 `Thread-0` 가 종료될 때 까지 자바 프로그램이 종료되지 않음.
    - 자바 프로그램은 사용자 스레드인 `main`과 `Thread-0` 모두 기다린 뒤 종료함.
- 따라서 `Thread-0: run() end` 가 출력됨.

## 스레드 생성 - `Runnable` 인터페이스 구현

```java
public class HelloRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": run()");
    }
}
```
_Runnable을 구현한 스레드_

```java
public class HelloRunnableMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        HelloRunnable runnable = new HelloRunnable();
        Thread thread = new Thread(runnable);
        thread.start();

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}
```
_스레드 실행 코드_

```
main: main() start
main: main() end
Thread-0: run()
```
_실행 결과_

기존과 실행 결과는 동일함. 스레드와 해당 스레드가 실행할 작업이 서로 분리되어 있다는 점이 상이할 뿐!

`Thread thread = new Thread(runnable);`과 같이 스레드 객체를 생성할 때 **실행할 작업을 생성자로 전달**하면 됨.

## Thread 상속 vs Runnable 구현

> 스레드 사용할 때는 `Thread` 를 상속 받는 방법보다 `Runnable` 인터페이스를 구현하는 방식을 사용하자! <br>
> 스레드와 실행할 작업을 명확히 분리하고, 인터페이스를 사용하므로 더 유연하고 유지보수 하기 쉬운 코드를 만들 수 있다.
{: .prompt-tip}

### Thread 클래스 상속 방식

> 장점

- 간단한 구현: `Thread`클래스를 상속받아 `run()`메서드만 재정의하면 됨

> 단점

- 상속의 제한: 자바는 **단일 상속만을 허용**하므로 (다중 상속 불가능) 이미 다른 클래스를 상속받고 있는 경우 `Thread` 클래스를 상속 받을 수 없음.
- 유연성 부족: 인터페이스를 사용하는 방법에 비해 **유연성이 떨어짐**.


### Runnable 인터페이스 구현 방식

> 장점

- 상속의 자유로움: `Runnable` 인터페이스 방식은 다른 클래스를 상속받아도 문제없이 구현 가능.
- 코드의 분리: 스레드와 실행할 작업을 분리하여 코드의 가독성을 높임.
- 여러 스레드가 동일한 `Runnable` 객체를 공유할 수 있어 자원 관리에 효율적.

> 단점

- 코드가 약간 복잡해질 수 있음. 
    - `Runnable` 객체를 생성하고 이를 `Thread` 에 전달하는 과정이 추가됨!

## 여러 스레드 만들기

단순히 스레드 3개를 만들어 실행해보자.

```java
public class ManyThreadMainV1 {
    public static void main(String[] args) {
        log("main() start");

        HelloRunnable runnable = new HelloRunnable();
        Thread thread1 = new Thread(runnable);
        thread1.start();
        Thread thread2 = new Thread(runnable);
        thread2.start();
        Thread thread3 = new Thread(runnable);
        thread3.start();
        log("main() end");
    }
}
```
_소스 코드_

```
20:58:47.575 [     main] main() start
20:58:47.579 [     main] main() end
Thread-1: run()
Thread-0: run()
Thread-2: run()
```
_실행 결과_

![img](/assets/img/2024-10-23-thread-creation-and-execution/4.png)

- 세 스레드 모두 동일한 작업(`HelloRunnable` 인스턴스에 있는 `run()` 메서드)을 실행함.
- 당연히 실행 순서는 유동적임.

## Runnable을 만드는 다양한 방법

### 정적 중첩 클래스 사용
```java
public class InnerRunnableMainV1 {
    public static void main(String[] args) {
        log("main() start");
        MyRunnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
        log("main() end");
    }

    static class MyRunnable implements Runnable {
        @Override
        public void run() {
            log("run()");
        }
    }
}
```

### 익명 클래스 사용
```java
public class InnerRunnableMainV2 {
    public static void main(String[] args) {
        log("main() start");
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                log("run()");
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
        log("main() end");
    }
}
```

### 익명 클래스 변수 없이 직접 전달
```java
public class InnerRunnableMainV3 {
    public static void main(String[] args) {
        log("main() start");
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                log("run()");
            }
        });
        thread.start();
        log("main() end");
    }
}
```

### 람다
```java
public class InnerRunnableMainV3 {
    public static void main(String[] args) {
        log("main() start");
        Thread thread = new Thread(() -> log("run()"));
        thread.start();
        log("main() end");
    }
}
```


---

<br>

> 이 포스트는 ["김영한의 실전 자바 - 고급 1편, 멀티스레드와 동시성"](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-1) 강의를 듣고 작성하였습니다! <br>
> 실습에 사용된 모든 코드는 [이곳](https://github.com/5jisoo/Java-Study)에서 확인할 수 있습니다.
{: .prompt-info}