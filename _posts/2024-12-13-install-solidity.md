---
title: Solidity 설치하기
date: 2024-12-13 19:19:00 +/-TTTT
categories: [Security, Blockchain]
tags: [security, blockchain, solidity, ethereum]
math: true
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
pragma solidity 0.8.28;

contract hello{
    string message = "Hello world!";
    function showMsg() public view returns (string memory){
        return message;
    }
}
```

