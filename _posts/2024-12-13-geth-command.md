---
title: Geth 기본 명령어
date: 2024-12-13 20:42:00 +/-TTTT
categories: [Security, Blockchain]
tags: [security, blockchain, geth]
math: true
---

## Ethereum 실행하기

```shell
% geth --datadir {경로} console
```

## Geth에서 자주 쓰이는 명령어

### 계좌 정보

모든 계좌 확인하기
```
> eth.accounts
```

특정 계좌 확인하기
```
> eth.accounts[숫자]
```

잔고 확인하기
```
> eth.getBalance(eth.accounts[숫자])
```

### 계좌 잠금을 풀 때

```
> personal.unlockAccount(eth.accounts[숫자])
> personal.unlockAccount(eth.accounts[숫자], "기존 비밀번호")
```

### 채굴

```
> miner.start()
> miner.stop()
```

채굴 전엔 잔액이 0이었으나, 채굴 후엔 아래 사진과 같이 잔액이 늘어난 것을 확인할 수 있음.

![img](/assets/img/2024-12-13-geth-command/0.png){: w="500" }{: .shadow }

- 0번째 계좌만 잔액이 늘어난 이유는 기본 계좌가 0번 계좌로 설정되어 있기 때문임.