---
title: 외부 Application에서 Geth에 연결
date: 2024-12-17 11:54:00 +/-TTTT
categories: [Security, Blockchain]
tags: [security, blockchain, geth, ethereum]
math: true
---

## 작업 환경

- masOS M1
- Geth Version: 1.10.17-stable
- Solidity Version: 0.8.19+commit.7dd6d404.Emscripten.clang

## Geth 콘솔 설정하기

콘솔이 아닌 Node.js와 같은 외부 프로그램에서 Geth에 접근할 수 있게 하기 위해 Geth를 실행할 때 다음 옵션들을 추가해야 함.

- `--http` : HTTP-RPC 서버를 활성화 하여, 외부에서 블록체인 서버에 접근 가능하도록 함.
- `--http.addr` : 
- `--http.port` : 외부에서 접근하기 위한 포트 번호 지정
- `--http.api` : 외부에서 접근할 수 있는 명령어들을 지정
- `--http.corsdomain`
- `--allow-insecure-unlock` : 외부에서 계좌 잠금 해제를 할 수 있게 하기 위한 명령어

예를 들어 다음과 같이 수행한다.
```shell
% geth --datadir "data" --http --http.addr "0.0.0.0" --http.port "7326" --http.api "web3,eth,personal,net" --http.corsdomain "*" --allow-insecure-unlock --nodiscover console
```

> --http 옵션은 Geth v1.10.0 부터 지원하고 있다!
{: .prompt-tip}

## Node.js로 Geth에 접속

Node.js나 Javascript에서 geth로 접속하기 위해서는 Web3 모듈이 필요하다.

> Web3 최신 버전이 버그로 연결이 잘 되지 않아... 옛날 버전인 0.20.0을 사용해줄 예정.

```shell
% npm install web3@0.20.0
```

Node.js를 사용하여 자바스크립트 코드를 작성하자.

provider를 설정한 뒤, 마지막에 잘 연결되었는지 확인하기 위해 계정의 주소를 로그로 찍도록 설정하였다.

```js
const Web3 = require("web3");
const web3 = new Web3();

web3.setProvider(new web3.providers.HttpProvider("http://127.0.0.1:7326"));

console.log(web3.eth.accounts[0]);
```

> provider 주소는 같은 컴퓨터이기에 localhost인 `127.0.0.1`로, <br>
> 포트 번호는 geth 콘솔을 실행하는 포트와 동일한 `7326`번으로 설정해주어야 한다.
{: .prompt-info}

![img](/assets/img/2024-12-17-geth-external-access/0.png){: w="500" }{: .shadow }
_계정의 주소가 출력된다._

마찬가지로 geth콘솔에서 `eth.accounts[0]`를 통해 계정의 주소를 출력하면 Node.js로 확인했던 동일한 주소를 확인할 수 있다.

![img](/assets/img/2024-12-17-geth-external-access/1.png){: w="600" }
_동일한 account[0] 주소 확인 가능_

다른 명령어(ex 계좌 확인)도 web3를 앞에 붙여 명령어를 확인할 수 있다!

```js
console.log(web3.eth.getBalance(eth.accounts[0]));
```

```shell
% node test.js
Balance: 38000000000000000000
```
