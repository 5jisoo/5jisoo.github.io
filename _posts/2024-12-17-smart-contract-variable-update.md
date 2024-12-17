---
title: 스마트 컨트랙트 변수 업데이트
date: 2024-12-17 10:04:00 +/-TTTT
categories: [Security, Blockchain]
tags: [security, blockchain, geth, ethereum, solidity]
math: true
---

## 스마트 컨트랙트 작성하기

다음 increase(), showCount() 함수를 작성해보자.
- val을 파라미터로 받아 count에 더해주고, 반환하는 `increase()` 함수
- count를 반환하는 `showCount()` 읽기 (view) 함수

```
// SPDX-License-Identifier : UNLICENSED

pragma solidity 0.4.24;

contract number {
    int8 private count = 0;

    function increase(int8 val) public returns (int8) {
        count += val;
        return count;
    }

    function showCount() public view returns (int8) {
        return count;
    }
}
```

### 컴파일

아래의 명령어를 통해 작성한 솔리디티 코드 파일을 컴파일하자.

```shell
% solcjs --abi --bin number.sol
```

## geth 콘솔

### 콘솔 접속

아래의 명령어를 통해 콘솔에 접속하자.

```shell
% geth --datadir cslab console
```

### bin, abi 변수 설정

컴파일한 바이너리(bin)파일과 abi 파일을 각각 bin, abi 변수에 저장한다.
- bin 변수는 문자열" " 앞에 16진수를 의미하는 0x를 붙여주어야 한다.

```shell
> abi = [{"constant":false,"inputs":[{"name":"val","type":"int8"}],"name":"increase","outputs":[{"name":"","type":"int8"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"showCount","outputs":[{"name":"","type":"int8"}],"payable":false,"stateMutability":"view","type":"function"}]
[{
    constant: false,
    inputs: [{
        name: "val",
        type: "int8"
    }],
    name: "increase",
    outputs: [{
        name: "",
        type: "int8"
    }],
    payable: false,
    stateMutability: "nonpayable",
    type: "function"
}, {
    constant: true,
    inputs: [],
    name: "showCount",
    outputs: [{
        name: "",
        type: "int8"
    }],
    payable: false,
    stateMutability: "view",
    type: "function"
}]
>
>
> bin = "0x608060405260008060006101000a81548160ff021916908360000b60ff16021790555034801561002e57600080fd5b506101558061003e6000396000f30060806040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680635136ed64146100515780635f0b84441461009b575b600080fd5b34801561005d57600080fd5b5061007f600480360381019080803560000b90602001909291905050506100cc565b604051808260000b60000b815260200191505060405180910390f35b3480156100a757600080fd5b506100b0610113565b604051808260000b60000b815260200191505060405180910390f35b6000816000808282829054906101000a900460000b0192506101000a81548160ff021916908360000b60ff1602179055506000809054906101000a900460000b9050919050565b60008060009054906101000a900460000b9050905600a165627a7a723058201760e3467082b3ba5d7d6a40b4c47fbc667a90713dddfb7c19838f6c81b8e51e0029"
"0x608060405260008060006101000a81548160ff021916908360000b60ff16021790555034801561002e57600080fd5b506101558061003e6000396000f30060806040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680635136ed64146100515780635f0b84441461009b575b600080fd5b34801561005d57600080fd5b5061007f600480360381019080803560000b90602001909291905050506100cc565b604051808260000b60000b815260200191505060405180910390f35b3480156100a757600080fd5b506100b0610113565b604051808260000b60000b815260200191505060405180910390f35b6000816000808282829054906101000a900460000b0192506101000a81548160ff021916908360000b60ff1602179055506000809054906101000a900460000b9050919050565b60008060009054906101000a900460000b9050905600a165627a7a723058201760e3467082b3ba5d7d6a40b4c47fbc667a90713dddfb7c19838f6c81b8e51e0029"
```

### 계좌 잠금 해제

acoounts[0] 잠금을 해제해주자.

```shell
> personal.unlockAccount(eth.accounts[0])
```

### 트랜잭션 발생시키기

0번 계좌에 바이너리 파일을 데이터로 포함시켜 트랜잭션을 발생시킨다.

```shell
> tx = eth.sendTransaction({from: eth.accounts[0], data: bin})
```

### 영수증(Receipt) 가져오기

트랜잭션이 블록체인에 잘 들어갔는지 확인하기 위해서는 receipt를 확인해야 한다.

```shell
> eth.getTransactionReceipt(tx)
null
```

현재는 블록에만 들어가있는 상태이고, 블록이 블록체인에 연결되지 않은 상태이기에,
채굴을 시작하여 블록체인에 연결해주어야 한다.

```shell
> miner.start()
> miner.stop()
```

이후에 다시 getTransactionReceipt()를 실행하면 다음과 같은 정보를 확인할 수 있다.

```shell
> eth.getTransactionReceipt(tx)
{
  blockHash: "0xcf3d84ecc789eac169e9b25483ec5a9b7cf0c7a8d3fba8d4e444becf57d96cc8",
  blockNumber: 13,
  contractAddress: "0x25b3c9798ea6d2140e6a687161a62b105f91ec00",
  cumulativeGasUsed: 90000,
  from: "0xc71a5b838cf49fd8bd7db6e1493ae0222f8acf3c",
  gasUsed: 90000,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x0",
  to: null,
  transactionHash: "0x125df14bd55b2e98f5ba5b5fd8b7e47c4e1afd5ef741c5ce3b6966612749c510",
  transactionIndex: 0
}
```

### 주소 저장

영수증에 나온 정보 중 `contractAddress`를 address 변수에 저장한다.

- `contractAddress` : 스마트컨트랙트의 위치

```shell
> address = eth.getTransactionReceipt(tx).contractAddress
"0x25b3c9798ea6d2140e6a687161a62b105f91ec00"
```

### 인터페이스 생성

ni (number interface) 를 생성하자.
- abi: 스마트 컨트랙트 코드의 설명이 담긴 json 파일
- address: 스마트 컨트랙트의 위치

이더리움 네트워크에 배포된 스마트 컨트랙트는 바이트 코드이기에, 해당 내용을 **실행**하려면 **abi를 사용하여 인코딩**을 해주어야 한다.

따라서, abi 파일 내용을 포함한 하나의 contract 객체를 만들어준 것.

```shell
> ni = eth.contract(abi).at(address)
{
  abi: [{
      constant: false,
      inputs: [{...}],
      name: "increase",
      outputs: [{...}],
      payable: false,
      stateMutability: "nonpayable",
      type: "function"
  }, {
      constant: true,
      inputs: [],
      name: "showCount",
      outputs: [{...}],
      payable: false,
      stateMutability: "view",
      type: "function"
  }],
  address: "0x25b3c9798ea6d2140e6a687161a62b105f91ec00",
  transactionHash: null,
  allEvents: function(),
  increase: function(),
  showCount: function()
}

```

## 실행하기

### showCount()

`showCount()`는 지난번 `showMsg()`와 동일한 형태로 `call()` 함수를 사용하여 실행시킬 수 있다.

```shell
> ni.showCount.call()
0
```

### increase()

그렇다면 인수 전달이 필요한 `increase()` 함수는 어떻게 호출할까?

아래처럼 호출하면 `invalid address` 에러가 발생한다.

```shell
> ni.increase(12)
Error: invalid address
    at web3.js:3930:15
    at web3.js:3756:20
    at web3.js:5025:28
    at map (<native code>)
    at web3.js:5024:12
    at web3.js:5050:18
    at web3.js:5075:23
    at web3.js:4137:16
    at apply (<native code>)
    at web3.js:4223:16
```

블록체인 안에 있는 변수를 업데이트 하기 위해서는 **`gas`**가 필요하다.

from, gas를 설정한 객체를 넣어 increase() 함수를 아래 명령어와 같이 실행시켜보자.

```shell
> result = ni.increase(12, {from: eth.accounts[0], gas: "470000"})
INFO [12-17|11:14:54.745] Submitted transaction                    fullhash=0xa25d919de99ba18437e7d25aa375e5393df34f18f39663fa8a23f1573f9ac257 recipient=0x25b3c9798EA6D2140E6a687161a62b105f91Ec00
"0xa25d919de99ba18437e7d25aa375e5393df34f18f39663fa8a23f1573f9ac257"
```

하지만, 트랜잭션이 만들어졌음에도 아직 count는 변경되지 않는다.

```shell
> ni.showCount.call()
0
```

블록은 만들어졌지만, 블록체인에 연결되지 않아서 그런 것! 다시 채굴을 해주면 블록체인에 연결된다.

```shell
> miner.start()
> miner.stop()
> ni.showCount.call()
12
```