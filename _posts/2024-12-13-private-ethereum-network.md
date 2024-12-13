---
title: 사설 Ethereum 네트워크 생성하기
date: 2024-12-13 18:40:00 +/-TTTT
categories: [Security, Blockchain]
tags: [security, blockchain, geth, ethereum]
math: true
---


### 계좌 만들기

```shell
% geth --datadir {경로} account new
```

### 만들어진 계좌 확인하기

```shell
% geth --datadir {경로} account list
```

![img](/assets/img/2024-12-13-private-ethereum-network/0.png){: w="700" }


### Genesis Block을 위한 설정

> **제네시스 블록**이란? <br>
> 블록체인에서 첫번째로 생성된 블록을 말합니다. <br>
> private network를 만들기 위해서는 네트워크를 시작하는 블록에 대한 정보가 필요하고, 이때 시작하는 블록이 제네시스 블록!
{: .prompt-info}

```shell
% puppeth
```

이를 실행하면 여러 옵션을 선택해야 한다.

```shell
jisoo@jisooui-MacBookPro ~ % puppeth
+-----------------------------------------------------------+
| Welcome to puppeth, your Ethereum private network manager |
|                                                           |
| This tool lets you create a new Ethereum network down to  |
| the genesis block, bootnodes, miners and ethstats servers |
| without the hassle that it would normally entail.         |
|                                                           |
| Puppeth uses SSH to dial in to remote servers, and builds |
| its network components out of Docker containers using the |
| docker-compose toolset.                                   |
+-----------------------------------------------------------+

// 네트워크 이름 설정

Please specify a network name to administer (no spaces or hyphens, please)
> hello

Sweet, you can set this via --network=hello next time!

INFO [12-13|18:52:30.925] Administering Ethereum network           name=hello
WARN [12-13|18:52:30.933] No previous configurations found         path=/Users/jisoo/.puppeth/hello

// 새로운 genesis 파일 생성

What would you like to do? (default = stats)
 1. Show network stats
 2. Configure new genesis
 3. Track new remote server
 4. Deploy network components
> 2

// POW 합의 알고리즘 선택

Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
> 1

// 선입금 계정 설정 - 그냥 enter 누르면 기본 설정.

Which accounts should be pre-funded? (advisable at least one)
> 0x

// 체인 / 네트워크 id 설정 - 그냥 enter 눌러 기본 설정.

Specify your chain/network ID if you want an explicit one (default = random)
> 
INFO [12-13|18:57:01.828] Configured new genesis block 

// genesis 파일 관리 (2번) 선택

What would you like to do? (default = stats)
 1. Show network stats
 2. Manage existing genesis
 3. Track new remote server
 4. Deploy network components
> 2

// 생성한 genesis 파일 내보내기

 1. Modify existing fork rules
 2. Export genesis configuration
 3. Remove genesis configuration
> 2

// 기본 그대로 enter 누름.

Which file to save the genesis into? (default = hello.json)
>      
INFO [12-13|19:00:41.038] Exported existing genesis block 

```

이렇게 실행하면 hello.json 파일이 생성됨.

<br>


```shell
%mv hello.json firstgeth
```

위 명령어를 통해 json파일을 데이터폴더로 이동시켜주자.

> 최신 버전의 geth를 사용하는 경우 중간에 폴더 선택하는 과정이 존재한다. <br>
> 버전에 따라 선택 과정이 상이한 것 같으니 유의하자!
{: .prompt-tip}


### Genesis Block 생성하기

```shell
% geth --datadir {경로} init {경로/파일이름.json}
```

### Ethereum 실행하기

```shell
% geth --datadir {경로} console
```

콘솔을 실행하면 명령어를 입력할 수 있다.

- 보통의 명령어는 `eth.`로 시작한다.

```shell
> eth.accounts
```

![img](/assets/img/2024-12-13-private-ethereum-network/1.png){: w="500" }{: .shadow }