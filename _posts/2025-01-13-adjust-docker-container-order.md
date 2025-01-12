---
title: 도커 컨테이너 실행 순서 조정하기
date: 2025-1-13 00:20:00 +/-TTTT
categories: [Infra, Docker]
tags: [docker, infra, troubleshooting]
math: true
---

## 에러 상황

스프링 부트 프로젝트 배포과정에서 발생한 에러입니다.

저는 먼저, 레디스와 스프링부트 api 서버를 각각 도커 컨테이너로 실행한 뒤, 스프링 서버에서 레디스에 연결하고자 하였습니다.

따라서 작성된 docker compose 파일은 다음과 같습니다.

> docker-compose.yml

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
    ports:
      - 6379:6379
  api:
    image: user/coaster:latest
    container_name: coaster
    environment:
      - TZ=Asia/Seoul
      - LANG=ko_KR.UTF-8
      - HTTP_PORT=8080
    ports:
      - '8080:8080'
    volumes:
      - /var/log/coaster:/var/log/coaster
```

<br>

하지만 스프링 도커 컨테이너가 계속 중단되었고, 로그를 확인해보니 다음과 같이 **레디스에 연결할 수 없다**는 오류 메세지를 확인할 수 있었습니다.

```shell
% docker logs {container-id}
```

> 에러 메세지

```shell
org.springframework.context.ApplicationContextException: Failed to start bean 'redisMessageListener'
	at org.springframework.context.support.DefaultLifecycleProcessor.doStart(DefaultLifecycleProcessor.java:326) ~[spring-context-6.2.1.jar!/:6.2.1]
	at org.springframework.context.support.DefaultLifecycleProcessor$LifecycleGroup.start(DefaultLifecycleProcessor.java:510) ~[spring-context-6.2.1.jar!/:6.2.1]
	at java.base/java.lang.Iterable.forEach(Iterable.java:75) ~[na:na]
	at org.springframework.context.support.DefaultLifecycleProcessor.startBeans(DefaultLifecycleProcessor.java:295) ~[spring-context-6.2.1.jar!/:6.2.1]
	at org.springframework.context.support.DefaultLifecycleProcessor.onRefresh(DefaultLifecycleProcessor.java:240) ~[spring-context-6.2.1.jar!/:6.2.1]
	at org.springframework.context.support.AbstractApplicationContext.finishRefresh(AbstractApplicationContext.java:1006) ~[spring-context-6.2.1.jar!/:6.2.1]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:630) ~[spring-context-6.2.1.jar!/:6.2.1]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:146) ~[spring-boot-3.4.1.jar!/:3.4.1]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:752) ~[spring-boot-3.4.1.jar!/:3.4.1]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:439) ~[spring-boot-3.4.1.jar!/:3.4.1]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:318) ~[spring-boot-3.4.1.jar!/:3.4.1]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1361) ~[spring-boot-3.4.1.jar!/:3.4.1]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1350) ~[spring-boot-3.4.1.jar!/:3.4.1]
	at com.coastee.server.ServerApplication.main(ServerApplication.java:10) ~[!/:0.0.1-SNAPSHOT]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:568) ~[na:na]
	at org.springframework.boot.loader.launch.Launcher.launch(Launcher.java:102) ~[app.jar:0.0.1-SNAPSHOT]
	at org.springframework.boot.loader.launch.Launcher.launch(Launcher.java:64) ~[app.jar:0.0.1-SNAPSHOT]
	at org.springframework.boot.loader.launch.JarLauncher.main(JarLauncher.java:40) ~[app.jar:0.0.1-SNAPSHOT]
Caused by: org.springframework.data.redis.listener.adapter.RedisListenerExecutionFailedException: org.springframework.data.redis.RedisConnectionFailureException: Unable to connect to Redis
	at org.springframework.data.redis.listener.RedisMessageListenerContainer.lazyListen(RedisMessageListenerContainer.java:383) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.listener.RedisMessageListenerContainer.start(RedisMessageListenerContainer.java:361) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.context.support.DefaultLifecycleProcessor.doStart(DefaultLifecycleProcessor.java:323) ~[spring-context-6.2.1.jar!/:6.2.1]
	... 20 common frames omitted
Caused by: org.springframework.data.redis.RedisConnectionFailureException: Unable to connect to Redis
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory$ExceptionTranslatingConnectionProvider.translateException(LettuceConnectionFactory.java:1858) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory$ExceptionTranslatingConnectionProvider.getConnection(LettuceConnectionFactory.java:1789) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory$SharedConnection.getNativeConnection(LettuceConnectionFactory.java:1586) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory$SharedConnection.lambda$getConnection$0(LettuceConnectionFactory.java:1566) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory.doInLock(LettuceConnectionFactory.java:1527) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory$SharedConnection.getConnection(LettuceConnectionFactory.java:1563) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory.getSharedConnection(LettuceConnectionFactory.java:1249) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory.getConnection(LettuceConnectionFactory.java:1055) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.listener.RedisMessageListenerContainer$Subscriber.lambda$initialize$0(RedisMessageListenerContainer.java:1241) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.listener.RedisMessageListenerContainer$Subscriber.doInLock(RedisMessageListenerContainer.java:1455) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.listener.RedisMessageListenerContainer$Subscriber.initialize(RedisMessageListenerContainer.java:1235) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.listener.RedisMessageListenerContainer.doSubscribe(RedisMessageListenerContainer.java:428) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.listener.RedisMessageListenerContainer.lazyListen(RedisMessageListenerContainer.java:404) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.listener.RedisMessageListenerContainer.lazyListen(RedisMessageListenerContainer.java:374) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	... 22 common frames omitted
Caused by: io.lettuce.core.RedisConnectionException: Unable to connect to localhost/<unresolved>:6379
	at io.lettuce.core.RedisConnectionException.create(RedisConnectionException.java:63) ~[lettuce-core-6.4.1.RELEASE.jar!/:6.4.1.RELEASE/08096cf]
	at io.lettuce.core.RedisConnectionException.create(RedisConnectionException.java:41) ~[lettuce-core-6.4.1.RELEASE.jar!/:6.4.1.RELEASE/08096cf]
	at io.lettuce.core.AbstractRedisClient.getConnection(AbstractRedisClient.java:354) ~[lettuce-core-6.4.1.RELEASE.jar!/:6.4.1.RELEASE/08096cf]
	at io.lettuce.core.RedisClient.connect(RedisClient.java:219) ~[lettuce-core-6.4.1.RELEASE.jar!/:6.4.1.RELEASE/08096cf]
	at org.springframework.data.redis.connection.lettuce.StandaloneConnectionProvider.lambda$getConnection$1(StandaloneConnectionProvider.java:112) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at java.base/java.util.Optional.orElseGet(Optional.java:364) ~[na:na]
	at org.springframework.data.redis.connection.lettuce.StandaloneConnectionProvider.getConnection(StandaloneConnectionProvider.java:112) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory$ExceptionTranslatingConnectionProvider.getConnection(LettuceConnectionFactory.java:1787) ~[spring-data-redis-3.4.1.jar!/:3.4.1]
	... 34 common frames omitted
Caused by: io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: localhost/127.0.0.1:6379
Caused by: java.net.ConnectException: Connection refused
	at java.base/sun.nio.ch.Net.pollConnect(Native Method) ~[na:na]
	at java.base/sun.nio.ch.Net.pollConnectNow(Net.java:672) ~[na:na]
	at java.base/sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:946) ~[na:na]
	at io.netty.channel.socket.nio.NioSocketChannel.doFinishConnect(NioSocketChannel.java:336) ~[netty-transport-4.1.116.Final.jar!/:4.1.116.Final]
	at io.netty.channel.nio.AbstractNioChannel$AbstractNioUnsafe.finishConnect(AbstractNioChannel.java:339) ~[netty-transport-4.1.116.Final.jar!/:4.1.116.Final]
	at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:776) ~[netty-transport-4.1.116.Final.jar!/:4.1.116.Final]
	at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:724) ~[netty-transport-4.1.116.Final.jar!/:4.1.116.Final]
	at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:650) ~[netty-transport-4.1.116.Final.jar!/:4.1.116.Final]
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:562) ~[netty-transport-4.1.116.Final.jar!/:4.1.116.Final]
	at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:997) ~[netty-common-4.1.116.Final.jar!/:4.1.116.Final]
	at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) ~[netty-common-4.1.116.Final.jar!/:4.1.116.Final]
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) ~[netty-common-4.1.116.Final.jar!/:4.1.116.Final]
	at java.base/java.lang.Thread.run(Thread.java:833) ~[na:na]
```

## 호스팅 환경

Azure Virtual Machine 으로 호스팅하였습니다.

- Ubuntu 22.04.5 LTS
- Docker version 27.4.1


## 해결 과정

### health-check 조건 추가

먼저, 스프링부트 컨테이너가 레디스 컨테이너보다 먼저 빌드되어 실행될 경우, **아직 레디스가 실행되지 않았기 때문에** 발생하는 에러가 아닐까? 라고 먼저 생각해보았습니다.

따라서 `docker-compose.yml`에 먼저 레디스 컨테이너의 health-check을 해준다음, 스프링 컨테이너가 실행될 수 있도록 다음과 같이 조건을 추가하였습니다.

```yaml
  api: # springboot 
    depends_on:
      redis:
        condition: service_healthy
```

<br>

그리고, redis에서 제공하는 health-check 명령어, `ping`을 통해 컨테이너가 실행되는지 확인하라는 조건을 추가하였습니다.

```yaml
  redis:
    healthcheck:
    test: [ "CMD", "redis-cli", "ping" ]
    interval: 10s
    timeout: 5s
    retries: 5
```

> [공식문서](https://redis.io/docs/latest/commands/ping/)에 따르면, `PONG`이 반환되며 redis 컨테이너가 실행되는 걸 확인할 수 있습니다.
{: .prompt-info}


### 직접 health-check 확인하기

위 `redis-cli ping` 명령어가 실행되는지 확인하는 명령어는 다음과 같습니다.

```shell
$ docker exec -it {redis-container-name} redis-cli ping
PONG
```


### 최종 파일

조건을 추가하면, 최종 docker-compose.yml 파일을 다음과 같이 작성할 수 있었습니다.

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
```

<br>

실행시키면, 다음과 같이 레디스 컨테이너 실행 이후 스프링 컨테이너가 실행되는 것을 확인할 수 있습니다.

![img](/assets/img/2025-01-13-adjust-docker-container-order/0.png)
_레디스 컨테이너 실행을 기다리는 스프링 컨테이너_

![img](/assets/img/2025-01-13-adjust-docker-container-order/1.png)
_레디스 컨테이너 health checking 완료 후 실행되는 스프링 컨테이너_

## 결과

하지만.. 컨테이너 실행 순서를 조정해주었음에도 스프링 서버에 레디스가 연결되지 않았습니다.

따라서 다른 에러 원인을 찾아 해결하였고, 다음 포스트에서 이어서 작성해보도록 하겠습니다.

> 다음 포스트: [도커 컨테이너 내부에서 localhost에 접근하기](https://5jisoo.github.io/posts/connect-localhost-inside-container/)
{: .prompt-tip}