---
title: Geth 설치
date: 2024-12-13 17:36:00 +/-TTTT
categories: [Security, Ethereum Basics]
tags: [security, blockchain, geth, ethereum, dapp]
math: true
image:
  path: /assets/img/2024-12-13-install-geth/0.png
  alt: <Geth 설치> 썸네일
---

## 블록체인과 스마트 컨트랙트

### 블록체인
데이터를 안전하게 저장하기 위한 자료구조
- 데이터를 블록에 저장하고, 그 블록들이 연결된 구조를 블록체인이라고 한다.

### 스마트 컨트랙트

블록체인에 넣어서 실행시킬 수 있는 프로그램
- 블록체인에 저장되어 값이나 프로그램이 절대로 위/변조되지 않는다.
    - '무결성'을 보장함.
- 하지만, 프로그램을 넣거나 변숫값을 읽고 쓰기 위해서는 비용이 발생한다.
- 주언어: Solidity

## Geth (Go Ethereum)

고(Go)에서 구현된 전체 이더리움 노드를 실행하기 위한 다목적 명령 줄 도구.
- CLI 기반의 이더리움 네트워크를 제공함.

### 이더리움 네트워크
1. main 네트워크 : '진짜' 이더리움 (비싼 이더를 구매해야 함..)
2. 사설 네트워크 : 이더리움 네트워크를 개인이 운영하는 것.
    - 개인이 운영하기 위해 서버 / 계좌.. 등등의 리소스가 필요함
3. test 네트워크 : 연습용 네트워크
    - 미리 만들어둔 사설 네트워크

이 중 사설 네트워크를 구축하기 위해 Geth를 사용할 예정.

### Geth 설치 방법

홈페이지 다운
- https://geth.ethereum.org/downloads

(mac os) brew 이용
```zsh
% brew tap ethereum/ethereum
% brew install ethereum
```

## 트러블 슈팅

### Failed to register the Ethereum service

![img](/assets/img/2024-12-13-install-geth/0.png){: w="700" }

```
Fatal: Failed to register the Ethereum service: only PoS networks are supported, please transition old ones with Geth v1.13.x
```

brew 명령어를 사용하고 `geth`를 실행하였는데 다음과 같은 오류가 발생하였다.

따라서 uninstall 후 책과 동일한 버전을 사용하기 위해 1.8.13 버전을 홈페이지에서 수동으로 다운 받아주었다. (실행 파일은 /user/local/bin/ 위치로 옮겨주어야 한다.)

```zsh
% brew uninstall ethereum
```

만약 포트 중복 에러가 발생한다면 다음 명령어를 통해 열린 포트 목록을 확인하고, 직접 닫아주면 된다.

```zsh
% sudo lsof -PiTCP -sTCP:LISTEN
% sudo kill -9 {PID}
```