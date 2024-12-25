---
title: 스마트 컨트랙트 실행하기
date: 2024-12-16 12:22:00 +/-TTTT
categories: [Security, Ethereum Basics]
tags: [security, blockchain, geth, ethereum, smart-contract, troubleshooting]
math: true
image:
  path: /assets/img/2024-12-16-run-smart-contract/0.png
  alt: <스마트 컨트랙트 실행하기> 썸네일
---

## Solidity Compiler 설치하기

npm을 이용하여 설치해보자.

```shell
% sudo npm install -g solc
```

## 프로그래밍 예제

> VSC에서 `hello.sol` 파일을 만들고 프로그래밍 해보자.

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.4.24;

contract hello{
    string message = "Hello world!";
    function showMsg() public view returns (string memory){
        return message;
    }
}
```

## 컴파일

> npm을 이용하여 설치해주었기 때문에, 기본 명령어가 `solc` 가 아닌 `solcjs`이다!
{: .prompt-info}

```shell
% solcjs --abi --bin hello.sol
```

성공적으로 컴파일이 되면 해당 폴더 내 .abi, .bin파일이 생성된다.

- abi (Application Binary Interface) : 스마트 컨트랙트 코드의 설명이 담긴 JSON 파일 
    - 어떤 함수가 들어있는지 설명하는 것.
- bin : 컴파일 된 바이너리 파일 

## 실행하기

### 콘솔 접속

먼저, 미리 만들어둔 사설 네트워크의 콘솔에 접속한다.

> 필자는 포트가 다른 프로세스와 겹쳐 임시로 30000으로 설정하였다.

```shell
geth --datadir cslab --port 30000 --nodiscover console
```

콘솔에서 `bin`, `abi` 변수를 각각 설정한다.
- `bin` : 컴파일된 bin파일을 설정해줌.
- `abi` : json파일 객체 그대로 설정.

### 변수 설정

> bin 변수 설정

- bin파일을 그대로 복사하여 앞에 16진수를 의미하는 0x를 붙여 변수로 설정한다.
- 또한 문자열을 의미하는 "" 따옴표로 감싸서 변수 설정을 해준다.
```shell
> bin = "0x{컴파일된 16진수 bin파일 내용}"
```


> abi 변수 설정

- bin과 다르게 string 문자열이 아닌 객체 형태로 바로 설정한다.

```shell
> abi = [{"inputs":[],"name":"showMsg","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"}]

// 결과
[{
    inputs: [],
    name: "showMsg",
    outputs: [{
        internalType: "string",
        name: "",
        type: "string"
    }],
    stateMutability: "view",
    type: "function"
}]
```

### 이더리움에서 실행하기

`sendTransaction()` 함수는 주소를 반환한다.
- 여기서는 `tx`라는 변수에 주소를 저장한다.

```shell
> tx = eth.sendTransaction({from: eth.accounts[0], data: bin})
```

<br>

하지만 위 명령어를 실행하면 다음과 같이 Authentication 에러가 발생한다.

![img](/assets/img/2024-12-16-run-smart-contract/0.png){: w="400" }{: .shadow}

그러므로, 0번 계좌의 비밀번호를 풀어주는 과정이 필요하다.

```shell
> personal.unlockAccount(eth.accounts[0])
```

그리고 다시 `sendTransaction()` 을 실행시켜주면! 성공적으로 실행된다.

### 채굴

하지만 현재는 바이트코드가 블록에는 들어갔으나, 다른 블록체인과 연결되지는 못한 상태이다.

```shell
> eth.getTransactionReceipt(tx)
null
```

따라서 거래는 되었지만 블록체인에 연결이 되지 않아 결과값이 `null`인 것을 확인할 수 있다.

```shell
> miner.start()
> miner.stop()
```

위 명령어를 통해 채굴을 시작하면 블록체인에 연결된다. 몇 초 기다린 뒤, stop() 해주도록 하자.

그리고 다시 receipt(부가 트랜잭션 정보)를 확인하면? 다음과 같은 결과값을 얻을 수 있다.

```shell
> eth.getTransactionReceipt(tx)
```

```json
{
  blockHash: "0xa49a284293d151db5b61f8cf2b50b4fc61d58e01a46b1a6ea6f10ecebf040e9f",
  blockNumber: 7,
  contractAddress: "0x71776219f07d03224d3c615a6fc22ad450bb3753",
  cumulativeGasUsed: 3925000,
  from: "0xc71a5b838cf49fd8bd7db6e1493ae0222f8acf3c",
  gasUsed: 300000,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x0",
  to: null,
  transactionHash: "0xaaa6c78db34de837020fb61cbfb02c4e64c75b0f745b07b52a9caac32cba4f6f",
  transactionIndex: 3
}
```

특히 이 결과값에서 `contractAddress`를 주로 사용하게 될 건데, <br>`contractAddress`란? 바이너리 코드가 들어간 위치(**스마트 컨트랙트가 들어간 블록의 주소**)이다.

아래 명령어를 통해 contractAddress를 address라는 변수에 저장해주도록 하자.

```shell
> address = eth.getTransactionReceipt(tx).contractAddress
"0x71776219f07d03224d3c615a6fc22ad450bb3753"
```

### 계약 인스턴스 가져오기

스마트 컨트랙트를 다음과 같이 배포하고, eth.contract()를 사용하여 계약 인스턴스를 가져오고 메서드들을 호출할 수 있다.

```shell
> contractInterface = eth.contract(abi).at(address)
```

```json
{
  abi: [{
      inputs: [],
      name: "showMsg",
      outputs: [{...}],
      stateMutability: "view",
      type: "function"
  }],
  address: "0x71776219f07d03224d3c615a6fc22ad450bb3753",
  transactionHash: null,
  allEvents: function(),
  showMsg: function()
}
```

이 내용은 위에서 작성한 `hello.sol` 코드의 내용이다. 여기서 우리가 작성한 showMsg() 함수를 실행해보자.

### 함수 실행

```shell
> contractInterface.showMsg.call()
"Hello world!"
```

## 트러블 슈팅

### `Error: intrinsic gas too low`

아래와 같은 에러가 발생한다면 배포하거나 호출하는 트랜잭션에서 가스 한도를 너무 적게 설정했을 가능성이 크다!

```shell
> tx = eth.sendTransaction({from: eth.accounts[0], data: bin})
Error: intrinsic gas too low
    at web3.js:3143:20
    at web3.js:6347:15
    at web3.js:5081:36
    at <anonymous>:1:6
```

다음과 같이 `gas:3000000` 충분한 가스를 할당할 경우 해결할 수 있다.

```shell
> tx = eth.sendTransaction({from: eth.accounts[0], data: bin, gas:3000000})
INFO [12-16|18:23:07.052] Submitted contract creation              fullhash=0x7ec5f48701c964fc7563fa5f2dcc52b00f8ad5298942d444f6ce297eeac6c066 contract=0x8EC59dabc0d88eAbeed9cC869B81795Ac321673d
"0x7ec5f48701c964fc7563fa5f2dcc52b00f8ad5298942d444f6ce297eeac6c066"
```

#### 여기서 가스란?

Gas(가스)는 이더리움이라는 거대한 컴퓨터를 쓰기 위한 일종의 연료이다.
- 트랜잭션을 수행하는데에 있어 네트워크에 대한 **수수료**이다.

1. 스마트 컨트랙트를 배포할 때
2. 함수에서 상태 변수에 변화를 줄 때

등등 컨트랙트 내부에서 특정 코드를 실행할 때 발생되는 수수료이다.


#### 해결되지 않은 의문.....

가스 계산이 잘못된 것 같다. 내 코드에서는 상태 변수에 변화도 없고, 아주 간단하여 복잡성도 적다..

뭔가 세팅에서 잘못된 것 같은데.. 풀리지 않았다

```shell
> eth.getTransactionReceipt(tx)
{
  blockHash: "0xbb1dcbcf02bd1cdc064bfe6027e5021cc80ae9e542282091f9b58297f82f2976",
  blockNumber: 58,
  contractAddress: "0xa9d6fe280d6fbe013726e4f6cfb44871ff2d34b9",
  cumulativeGasUsed: 199384,
  from: "0xc71a5b838cf49fd8bd7db6e1493ae0222f8acf3c",
  gasUsed: 199384,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: null,
  transactionHash: "0x587427e8b4ada7cae90cc17dbb1e265df6bb3d5770fdb32db707f89dfbcc7a5e",
  transactionIndex: 0
}
```

이 트랜잭션에서는 199384의 gas를 사용하였는데, 왜이렇게 많이 사용했을까? 아니면 이게 일반적인 수치인가?

공부가 더 필요하다..

### Error: new BigNumber() not a base 16 number: 

```shell
> contractInterface.showMsg.call()
Error: new BigNumber() not a base 16 number: 
    at L (bignumber.js:3:2876)
    at bignumber.js:3:8435
    at a (bignumber.js:3:389)
    at web3.js:1110:23
    at web3.js:1634:20
    at web3.js:826:16
    at map (<native code>)
    at web3.js:825:12
    at web3.js:4080:18
```

뜬금없이 함수 호출할 때, 다음과 같이 `Error: new BigNumber() not a base 16 number: ` 에러가 발생하였다.

구글링해보니 web3.js 버그.. 배포 주소 버그.. 등등에 관련한 질문이 나왔는데 **결과적으로는 solidity 컴파일러 버전과 geth 버전의 차이때문에 발생한 문제였다.**

geth 버전은 공부하고 있는 책의 버전 대로 `1.8.13-stable`을, 하지만 solidity 컴파일러는 `0.8.x`대의 최신 버전을 사용하여 발생한 에러였다.

```bash
% npm uninstall solc
% npm install -g solc@0.4.24
```

위의 명령어를 통해 컴파일러를 삭제하고, geth 버전에 맞춘 `0.4.24` 버전으로 다운그레이드하여 해결하였다.

물론, solidity 코드도 `pragma solidity = 0.4.24;` 로 변경하여 특정 버전의 컴파일러를 선택하도록하였다.

Thanks to.. [힌트 주신 하나의 게시글](https://www.clien.net/service/board/cm_blockchain/13645133)