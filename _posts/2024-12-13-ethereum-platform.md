---
title: 이더리움 플랫폼
date: 2024-12-13 14:01:00 +/-TTTT
categories: [Security, Building Ethereum Dapps]
tags: [security, blockchain, ethereum, dapp]     # TAG names should always be lowercase
math: true
---

# 이더리움 지갑으로 이더리움 플랫폼에 접속하기

- 미스트 : 이더리움 디앱을 위한 브라우저
- 이더리움 지갑 : 특정 버전의 미스트를 기반으로 하는 지갑 디앱

**이더리움 지갑**의 주 목적? 이더를 보관하거나 보내고 받기 위함.

## 이더리움 지갑 시작하기

먼저, [github.com/ethereum/mist/releases/tag/v0.11.1](https://github.com/ethereum/mist/releases/tag/v0.11.1) 에 접속하여 이더리움 지갑을 다운받는다.


세팅은 다음과 같이 설정한다.

<table>
    <tr>
        <th>
        개발 > 네트워크
        </th>
        <th>
        개발 > sync mode (동기화 모드)
        </th>
    </tr>
    <tr>
        <td>
            <img alt="img" width="400" src="/assets/img/2024-12-13-ethereum-platform/1.png"><br>
            <b>Ropsten - Test network</b> 로 설정
        </td>
        <td>
            <img alt="img" width="400" src="/assets/img/2024-12-13-ethereum-platform/1.png"><br>
            <b>Fast</b> 또는 <b>Full</b> 로 설정
        </td>
    </tr>
</table>

동기화 모드의 차이는 다음과 같다.

|동기화 모드|특징|예상 소요시간|
|---|---|---|
|**Light** (default)| 피어 전체 노드의 상태 트리만 가져와서 동기화함. | 상대적으로 적게 소요 |
|**Fast**| 전체 블록체인을 다운로드하지만 동기화를 시작하기 전 <br> 혹은 새 블록이 생성되기 전 64개 블록만 검증 <br> (디스크 1GB정도 필요) | 2-4시간 |
|**Full**| 전체 블록체인을 다운로드하고 모든 블록을 로컬에서 검증 <br> (디스크 100GB 정도 필요) | 하루-이틀 |

> 이더를 전송하거나 스마트 컨트랙트를 배포하는 등의 쓰기 작업을 수행하려는 경우 블록 전체를 동기화해야 한다. 따라서 **Fast** 또는 **Full** 동기화를 선택하자.
{: .prompt-tip}

> 안타깝게도ㅎㅎ😅 저는 여기서 무한로딩으로 실습이 진행되지 않아.. 개념만 정리하였다........

# 디앱의 핵심 기능 : 스마트 컨트랙트

## 계정

계정에는 두가지 유형이 존재함.

### 외부 소유 계정 (EOA) 또는 외부 계정
> Externally Owned Account / External Account

- 비공식적으로 사용자 계정이라고도 부름
- 공개키를 사용해 공개적으로 식별할 수 있으나 개인키를 가지고 있을 때만 작동 가능
- 이더를 구입하면 외부 계정에 저장됨.
- EOA에 있는 스마크 컨트랙트에서 트랜잭션이 시작됨.

### 컨트랙트 계정
> Contract Account

- 스마트 컨트랙트가 실제로 실행되는 계정
- 계정 주소는 배포할 때 생성되고 이 주소로 블록체인상에 컨트랙트 확인 가능.

### 계정 비교

속성 | EOA | 컨트랙트 계정
---|---|---
이더 잔액이 있는가? | 예 | 예
트랜잭션 메시지를 실행할 수 있는가? | 예 | 아니요
메시지를 부를 수 있는가 | 아니요 | 예
프로그램 코드가 있는가 | 아니요 | 예


## 이더

이더란, 이더리움 블록체인을 기반으로 한 암호화폐.
- 주 목적: 플랫폼을 통해 거래되는 서비스 및 상품에 금전적 가치를 부여하는 것!

또한 이러한 이더는 거래 수수료를 지불할 때도 사용되는데, <br>
거래 수수료는 하나의 트랜잭션을 실행하기 위해 소비되는 연산 자원을 가스라는 단위로 계산함.
- 그러나 실제 수수료는 가스를 이더로 변환하여 이더로 정상되는 것!

### 이더의 수명 주기
1. 이더 발행
2. 이더 전송
3. 이더 보관
4. 이더 교환

#### 이더 발행
이더는 채굴 프로세스를 통해 생성된다.

- 채굴자는 새로운 블록체인 블록에 트랜잭션을 모으고 추가하기 위해 경쟁함.
- 채굴에 성공하면 채굴자는 일정량의 이더로 보상을 받게 됨.

#### 이더 전송
이더가 발행되면 **채굴자의 외부 계정에 할당**된다.

채굴자는 이더리움 지갑이나 프로그래밍을 통해 이더를 다른 외부 게정이나 스마트 컨트랙트로 전송할 수 있다.

#### 이더 보관

채굴, 스마트 컨트랙트 거래, 이더 구입을 통해 이더를 얻으면 하나의 계정에 입금된다.

다양한 데스크톱, 모바일, 온라인 등등의 지갑에 보관한다.

#### 이더 교환

이더는 가치가 있기 때문에 보통 무료로 전송되지 않는다. 보통 개인이나 **암호화폐 거래소**를 통해 거래한다.

## 가스

가스란 이더리움 플랫폼에 부과되는 트랜잭션 수수료를 측정하는 단위이다.

트랜잭션 하나를 완료하는 데 사용되는 가스의 양은 트랜잭션을 실행하는 동안 EVM이 소비하는 연산 자원의 양에 따라 상이하다.

- 가스를 통해 거래 수수료를 부과하면 DoS 공격(무한루프와 같이 대량의 트랜잭션을 실행시키는 것.)을 막을 수 있다.

```
트랜잭션 수수료(이더) = 사용된 가스량 * 가스 단위당 가격(이더)
```

트랜잭션이 실행되는 동안 EVM은 가스를 소비하며, 트랜잭션이 **성공적으로 완료**되거나 **완료되기 전에 가스를 모두 소모한 경우** 트랜잭션이 종료된다.

## 콜과 트랜잭션

> 계정은 두 가지 유형의 메세지 콜, 트랜잭션을 통해 서로 상호작용한다.
{: .prompt-info}

### 콜

블록체인에 저장되지 않고 다음곽 같은 특성을 가진 메세지를 통해 콜을 보낸다.

- 블록체인의 상태를 변경하지 않는 읽기 전용 작업만 수행한다.
- 가스를 소비하지 않는다.
- 동기적으로 처리된다.
- 반환값을 즉시 돌려준다.
- 이더를 다른 컨트랙트 계정으로 옮길 수 없다.

보통 컨트랙트 멤버 변수를 직접 호출하거나 컨트랙트 상태를 변경하지 않는 상수 함수를 호출함.

### 트랜잭션

채굴 과정에서 직렬화되고 블록체인에 저장되는 메세지를 통해 전송됨.

|트랜잭션의 필드|
----|-----
| Sender Address | 송신자 주소 |
| Recipient Address | 수신자 주소 |
| Value | 전송할 이더 수량 (Wei 단위). <br> 메세지가 이더 전송에 사용되는 경우 |
| Data | 입력값. 메세지가 함수 호출에 사용되는 경우 |
| StartGa | 메시지를 실행하기 위해 소모되는 최대 가스양. <br> 이를 초과하는 경우 EVM은 예외를 발생시키고 상태를 롤백함. |
| Digital signature | 거래 송신자 신원 증명 |
| GasPrice | 트랜잭션을 실행하기 위해 필요한 가스 비용 (이더 단위) |

- 쓰기 작업을 할 수 있다.
- 가스를 소모하고 이더로 비용을 지불해야 한다.
- 비동기적으로 처리된다. <br>
    채굴 과정에서 실행되며 새로운 블록에 포함된 네트워크에 전파된다.
- 트랜잭션 ID를 반환하고 다른 값은 반환하지 않는다.
- 이더를 컨트랙트 계정으로 전송할 수 있다.

## EVM

EVM (이더리움 가상 머신)은 스택 기반의 추상 컴퓨터 머신이다.

컴퓨터가 이더리움 앱을 실행할 수 있도록 하여 두 개의 메모리 영역으로 구성된다.

- 휘발성 메모리 또는 메모리
    - 메모리는 단어-주소 바이트 배열.
    - 메세지를 호출할 때맘다 컨트랙트에 할당됨.
    - 읽기 작업은 256비트 단어에 접근하며, 쓰기 작업은 8비트 또는 256비트 크기로 수행함.
- 스토리지
    - 키와 값의 크기가 모두 256비트인 키-값 저장소.
    - 각 계정에 할당되며 블록체인에 저장됨.
    - 컨트랙트 계정은 자기 자신의 스토리지만 접근 가능함.

# Geth

## 콘솔 시작하기

### 버전 정보 표시

```shell
> web3.version
```

콘솔에서 자바스크립트 문법 사용이 가능하다.

```shell
> var apiVersion = web3.version.api
> var nodeVersion = web3.version.node
> console.log(apiVersion)
0.20.1
> console.log(nodeVersion)
Geth/v1.10.17-stable-25c9b49f/darwin-amd64/go1.18
```

### 연결 상태 확인

`web3.net` 객체에서 클라리언트 연결 정보를 확인할 수 있다.

```shell
> net
{
  listening: true,
  peerCount: 0,
  version: "1",
  getListening: function(callback),
  getPeerCount: function(callback),
  getVersion: function(callback)
}
```

노드에 대한 자세한 정보는 `web3.admin` 객체를 사용하여 호출한다.
```shell
> admin.nodeInfo
{
  enode: "enode://b6135ca5222bc4...",
  enr: "enr:-Jy4QNx1AnlsHclrTFp6chIh1...",
  id: "828...",
  ip: "192....",
  listenAddr: "[::]:30303",
  name: "Geth/v1.10.17-stable-25c9b49f/darwin-amd64/go1.18",
  ...
}
```

피어 속성은 연결된 피어에 대해 자세한 정보를 제공한다.

```shell
> admin.peers
```

### 블록체인에 접속하기

`web.eth` 객체는 클라이언트와 블록체인의 실시간 정보를 가져온다.

`eth.blockNumber` 속성을 통해 가장 최신 블록 번호를 확인할 수 있다.

```shell
> eth.blockNumber
25
> eth.getBlock(25)
{
  difficulty: 478278,
  extraData: "0xd783010a1184676...",
  gasLimit: 4816062,
  gasUsed: 0,
  hash: "0x8269d29eb5d649f5...",
  logsBloom: "0x0000000...",
  miner: "0x93c5a1a2fe437c36993bdeec39e677d38d3bdc51",
  mixHash: "0xbab1c2d2924s...",
  nonce: "0x15150000eed0a1bb",
  number: 25,
  ...
}
```

다음과 같이 최신 블록에 저장된 가장 첫번째 트랜잭션 역시도 가져올 수 있다.

```shell
> eth.getTransactionFromBlock(25, 0)
```

# 심플코인 컨트랙트 다시 보기

이전에 작성한 SimpleCoin 컨트랙트

```
// SPDX-License-Identifier: UNLICENSED

pragma solidity 0.8.19;

contract SimpleCoin {
    mapping (address => uint256) public coinBalance;

    constructor() public {
        coinBalance[] = 10000;
    }

    function transfer(address _to, uint _amount) public {
        coinBalance[msg.sender] -= _amount;
        coinBalance[_to] += amount;
    }
}
```

## SimpleCoin 컨트랙트 개선하기

### 생성자에 매개변수 추가

생성자와 전송 함수 둘 다 개선해보자!

먼저, 생성자에 매개 변수를 추가하자.

```
constructor(uint256 _initialSupply) public {
    coinBalance[msg.sender] = _initialSupply;
}
```

`msg.sender`는 메시지 발신자(함수 호출자)의 주솟값을 가지고 있는 특수 속성!

즉 **메세지 발진자가 컨트랙트의 소유자**이다.

### 전송 함수를 견고하게 만들기

```
function transfer(address _to, uint _amount) public {
    require(coinBalance[msg.sender] >= _amount); // 잔액 확인
    require(coinBalance[_to] + _amount >= coinBalance[_to]); // 산술 오버플로우 확인
    coinBalance[msg.sender] -= _amount;
    coinBalance[_to] += _amount;
}
```

`require` 특별 함수는 조건이 충족되지 않으면 예외를 발생시킨다. `throw`로 직접 발생시킬 수는 있으나, 현재엔 잘 사용되지 않는 예전 문법임!

### 이벤트 발생시키기

컨트랙트는 어떤 함수로도 발생시킬 수 있는 이벤트를 선언할 수 있다.

컨트랙트 상태를 모니터링하는 클라이언트가 이벤트를 처리한다.

```
event Transfer(address indexed from, address indexed to, uint256 value);
```

예를 들어 SimpleCoin이 전송된 뒤, 발생하는 이벤트를 위와 같이 선언할 수 있다.

```
function transfer(address _to, uint _amount) public {
    ...
    emit Transfer(msg.sender, _to, _amount);
}
```

그리고 전송 함수 마지막에 다음과 같이 이벤트를 호출한다.

## 개선된 코드 사용하기

최종 코드는 다음과 같다.

```
// SPDX-License-Identifier: UNLICENSED

pragma solidity 0.8.19;

contract SimpleCoin {
    mapping (address => uint256) public coinBalance;
    event Transfer(address indexed from, address indexed to, uint256 value);

    constructor(uint256 _initialSupply) public {
        coinBalance[msg.sender] = _initialSupply;
    }

    function transfer(address _to, uint _amount) public {
        require(coinBalance[msg.sender] >= _amount); // 잔액 확인
        require(coinBalance[_to] + _amount >= coinBalance[_to]); // 산술 오버플로우 확인
        coinBalance[msg.sender] -= _amount;
        coinBalance[_to] += _amount;
        emit Transfer(msg.sender, _to, _amount);
    }
}
```

### 수정된 생성자 테스트

먼저 ACCOUNT에서 특정 계정을 하나 선택해준 뒤, Deploy를 실행한다.

![img](/assets/img/2024-12-13-ethereum-platform/2.png){: w="400" }{: .shadow}
_Deploy 생성자 변수 전달_

이제 배포 작업을 할 때 다음과 같이 생성자의 입력 변수 `_initialSupply`를 입력받는다.

<br>

선택해준 계정의 `coinBalance`를 확인해보자. 잔액이 10000임을 정상 확인할 수 있다.

![img](/assets/img/2024-12-13-ethereum-platform/3.png){: w="400" }{: .shadow}
_잔액: 10000_

### 수정된 전송자 테스트

토큰이 없는 계정에서 먼저 SimpleCoin 토큰을 이체해보자.

다른 계정을 선택하고, transfer 150을 실행시켜보면? 잔액이 부족하기에 require함수의 조건에 통과하지 못해 다음과 같은 오류가 발생한다.

![img](/assets/img/2024-12-13-ethereum-platform/4.png){: w="600" }{: .shadow}
_실패_

그리고 다시 원래 coinBalance가 10000인 계정을 선택해서 150 토큰을 전송하고자 시도한다면? 다음과 같이 성공적인 실행을 확인할 수 있다.


![img](/assets/img/2024-12-13-ethereum-platform/5.png){: w="600" }{: .shadow}
_성공_

## 이더리움 네트워크에서 코인 전송은 어떻게 실행되는가?

![img](/assets/img/2024-12-13-ethereum-platform/6.png){: w="500" }{: .shadow}
_SimpleCoin 전송 트랜잭션의 수명주기_

SimpleCoin 지갑은 이더리움 네트워크의 로컬 노드에서 SimpleCoin 스마트 컨트랙트의 **`transfer()` 함수를 호출하여 전송 트랜잭션을 생성**한다.

채굴 노드가 새로운 블록을 블록체인에 포함할 때까지 **네트워크 전체에서 검증되고 전파**된다. 새로운 블록이 네트워크 전체에 전파되면 마지막으로 로컬 노드로 돌아간다.

---

> 「Building Ethereum Dapps 이더리움 디앱 개발」 (로베르토 인판테 저, 정종화 역)을 읽고 작성한 글입니다.
{: .prompt-tip}