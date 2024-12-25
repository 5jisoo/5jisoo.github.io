---
title: Node.js에서 스마트 컨트랙트를 블록체인에 입력하기
date: 2024-12-20 9:51:00 +/-TTTT
categories: [Security, Ethereum Basics]
tags: [security, blockchain, ethereum, solidity, nodejs]
math: true
---

## 작업 환경

> [외부 Application에서 Geth에 연결](https://5jisoo.github.io/posts/geth-external-access/) 에서 이어지는 내용입니다. (Node.js 및 Web3 설정)
{: .prompt-info}

- masOS M1
- Geth Version: 1.10.17-stable
- Solidity Version: 0.8.19+commit.7dd6d404.Emscripten.clang
- Node.js Version: v20.17.0
- Web3 Version: v0.20.0

## Solidity

다음과 같이 `showMsg()`라는 간단한 solidity 코드를 node.js를 통해 실행시켜볼 예정이다.

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.19;

contract hello{
    string message = "Hello world!";
    function showMsg() public view returns (string memory){
        return message;
    }
}
```

먼저, 이 솔리디티 파일을 컴파일 해주어 abi, bin 파일을 만들어 준다.

```shell
% solcjs --abi --bin hello.sol
```

## Node.js 

Geth 콘솔에서 했던 작업을 그대로 Node.js 코드로 작성해보자.

- 콘솔을 7326 포트를 사용하기에 localhost:7326으로 설정해주었다.
- 각각 `abi`, `bin` 변수에 `abi`파일, `bin`파일 내용을 저장한다.

```javascript
const Web3 = require("web3");
const web3 = new Web3();
web3.setProvider(new web3.providers.HttpProvider("http://127.0.0.1:7326"));

const abi = [
  {
    inputs: [],
    name: "showMsg",
    outputs: [{ internalType: "string", name: "", type: "string" }],
    stateMutability: "view",
    type: "function",
  },
];
const bin =
  "0x{bin파일}";
```

## 계좌 잠금 해제 및 트랜잭션 전송

여기까지는 이전과 동일하다.

그 다음 password를 사용하여 `eth.accounts[0]`의 잠금을 푼 뒤, 트랜잭션을 전송해야 한다. <br>
하지만, 잠금을 푸는 데는 오랜 시간이 걸린다..!

```js
const result = web3.personal.unlockAccount(web3.eth.accounts[0], "1234");
tx = web3.eth.sendTransaction({
    from: web3.eth.accounts[0],
    data: bin,
    gas: "470000",
  });
```

`unlockAccount()` 함수가 완전히 끝난 뒤에 `sendTransaction()` 함수가 실행될 수 있도록 `await` 키워드를 사용해주어야 한다. 

> [Promise](https://ko.javascript.info/promise-basics)와 [async/await](https://ko.javascript.info/async-await) 참고 문서
{: .prompt-info}

간단하게 Promise 객체는 **어떤 작업에 관한 상태 정보**를 갖고 있는 객체이다. 작업의 결과가 Promise 객체에 저장되어 해당 객체를 보면 작업의 성공/실패 여부를 알 수 있다.

- 보통 시켜두고 언제 완료될지 모르는 로직(비동기 로직)을 Promise 객체에 작성한다.
- new Promise와 같이 객체가 생성되는 순간 바로 executor이라는 콜백 함수를 실행한다.

```js
const unlock = new Promise((resolve) => {
    const result = web3.personal.unlockAccount(web3.eth.accounts[0], "1234");
    resolve(result);
});
```

위 코드 에서는 unlockAccount() 비동기 로직이 완료되면 `resolve(result);`를 호출하는 코드이다.

그 뒤, sendTransaction은 이전의 promise가 끝날때까지 기다려야하므로, `await unlock;`이라는 키워드를 통해 기다려준다.

다만 await는 async함수 내에서만 사용할 수 있으므로, 코드 전체를 async함수로 감싸주어야 한다. 따라서 아래 코드와 같이 `async input()` 함수로 감싸주어 코드를 작성해준다.

```js
const Web3 = require("web3");
const web3 = new Web3();
web3.setProvider(new web3.providers.HttpProvider("http://127.0.0.1:7326"));

const abi = [
  {
    inputs: [],
    name: "showMsg",
    outputs: [{ internalType: "string", name: "", type: "string" }],
    stateMutability: "view",
    type: "function",
  },
];
const bin =
  "0x{bin파일}}";

input();

async function input() {
  const unlock = new Promise((resolve) => {
    const result = web3.personal.unlockAccount(web3.eth.accounts[0], "1234");
    resolve(result);
  });

  await unlock;

  tx = web3.eth.sendTransaction({
    from: web3.eth.accounts[0],
    data: bin,
    gas: "470000",
  });
}

```

## Receipt 확인하기

다음으로, 트랜잭션의 receipt 정보를 가져와서 contractAddress 값을 가져와야 한다. 이 내용은 아래의 코드와 같이 js로 작성할 수 있다.

```js
web3.eth.getTransactionReceipt(tx);
```

하지만 그 전에! 트랜잭션 전송 후 **마이닝**을 통해 블록을 블록체인에 연결한 뒤에 receipt 값이 생기면 contractAddress값을 가져올 수 있다.

> 블록이 블록체인에 연결되기 전에, receipt는 null이다. [참고: 블록 연결 상태 확인하기](https://5jisoo.github.io/posts/smart-contract-variable-update/#%EB%B8%94%EB%A1%9D-%EC%97%B0%EA%B2%B0-%EC%83%81%ED%83%9C-%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0)
{: prompt-tip}

따라서 1초마다 receipt 값을 검사하여 null아닐때 address를 return하도록 작성할 예정이다. `setInterval()`함수를 사용하여 작성해보자.

> [setInterval() 참고 자료](https://developer.mozilla.org/ko/docs/Web/API/Window/setInterval)
{: .prompt-info}

setInterval의 매개변수는 다음과 같다.
- `func` : `delay`마다 실행되는 function
- `delay` : 타이머가 지정된 함수 또는 코드 실행 사이에 지연해야 하는 밀리초(1/1000초) 단위의 시간
등등

따라서 
1. 1초마다 반복하여
2. `web3.eth.getTransactionReceipt(tx);`이 null인지 확인하고, 
3. null일 경우 기다린다는 로그를 출력하고, 
4. null이 아닐 경우 contractAddress를 꺼내 로그를 출력하고, 타이머의 반복작업을 취소하는 

코드를 작성해보자.

```js
const waitForConfirmation = setInterval(() => {
  const result = web3.eth.getTransactionReceipt(tx);
  if (result) {
    console.log(result.contractAddress);
    clearInterval(waitForConfirmation);
  } else {
    console.log("Wait for confirmation ... ");
  }
}, 1000); // 1sec
```

## 중간 확인

먼저 geth 콘솔을 외부 접근이 가능하도록 실행해주자.

- `--allow-insecure-unlock` : 원격 잠금 허용

```shell
% geth --datadir "data" \
--http  \
--http.addr "0.0.0.0" \
--http.port "7326" \
--http.api "web3,eth,personal,net" \
--http.corsdomain "*" \
--allow-insecure-unlock \
--nodiscover console
```

그 뒤, node 코드를 실행시킨다.

```shell
% node test.js
```

현재 js 코드는 다음과 같다.

```js
const Web3 = require("web3");
const web3 = new Web3();
web3.setProvider(new web3.providers.HttpProvider("http://127.0.0.1:7326"));

const abi = [
  {
    inputs: [],
    name: "showMsg",
    outputs: [{ internalType: "string", name: "", type: "string" }],
    stateMutability: "view",
    type: "function",
  },
];
const bin =
  "0x{bin파일}"; // 길어서 생략

input();

async function input() {
  const unlock = new Promise((resolve) => {
    const result = web3.personal.unlockAccount(web3.eth.accounts[0], "1234");
    resolve(result);
  });

  await unlock;

  tx = web3.eth.sendTransaction({
    from: web3.eth.accounts[0],
    data: bin,
    gas: "470000",
  });

  const waitForConfirmation = setInterval(() => {
    const result = web3.eth.getTransactionReceipt(tx);
    if (result) {
      console.log(result.contractAddress);
      clearInterval(waitForConfirmation);
    } else {
      console.log("Wait for confirmation ... ");
    }
  }, 1000);
}
```


그 뒤, console에서 채굴을 시작하면??

```shell
> miner.start()
> miner.stop()
```

다음과 같이 contractAddress를 확인할 수 있다.

![img](/assets/img/2024-12-20-smart-contract-nodejs/0.png){: w="500" }{: .shadow}
_정상 실행 확인 완료!_

## 스마트 컨트랙트 실행

geth에서 했던 방식과 동일하게 코드를 작성해준다.
- web3를 붙인다는 것만 기억해주면 됨.

address는 위에서 얻은 contractAddress 값을 그대로 가져와주었다.

```js
const Web3 = require("web3");
const web3 = new Web3();
web3.setProvider(new web3.providers.HttpProvider("http://127.0.0.1:7326"));

const abi = [
  {
    inputs: [],
    name: "showMsg",
    outputs: [{ internalType: "string", name: "", type: "string" }],
    stateMutability: "view",
    type: "function",
  },
];

const address = "0x33c37da4443c10badc724830d4c51c561d54c2df";

const helloInterface = web3.eth.contract(abi).at(address);
console.log(helloInterface.showMsg.call());
```

이렇게 작성하고, 실행하면?

```shell
% node run.js
```

다음과 같이 `Hello world!`가 출력되는 것을 확인할 수 있다.

![img](/assets/img/2024-12-20-smart-contract-nodejs/1.png){: w="400" }{: .shadow}
_정상 실행 확인 완료!_
