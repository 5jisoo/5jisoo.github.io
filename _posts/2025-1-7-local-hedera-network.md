---
title: 헤데라 로컬 네트워크 구축 및 스마트 컨트랙트 작성하여 배포하기
date: 2025-1-7 18:38:00 +/-TTTT
categories: [Security, Hedera Basics]
tags: [security, blockchain, hedera]
math: true
image:
  path: /assets/img/2025-1-7-local-hedera-network/6.png
  alt: <스마트 컨트랙트 변수 업데이트> 썸네일
---

## 헤데라 로컬 네트워크 구축

> [GitHub: hashgraph/hedera-local-node](https://github.com/hashgraph/hedera-local-node)를 참고하여 작성하였습니다.
{: .prompt-info}

### 사전 준비

Settings > General 에서 `VirtioFS` file sharing implementation 이 설정되어 있는지 확인합니다.

![img](/assets/img/2025-1-7-local-hedera-network/1.png)

Settings > Resources > Advanced에서 리소스를 설정합니다.

> 최소 아래와 같은 스펙으로 맞추어야 한다고 합니다.

- CPUs: 6
- Memory: 8 GB
- Swap: 1 GB
- Disk Image Size: 64 GB

![img](/assets/img/2025-1-7-local-hedera-network/2.png)

Settings > Resources > File sharing에서 hedera-node-folder를 설정합니다.
- 프로젝트 루트 디렉터리 파일 혹은 `npm root -g`를 실행시킨 결과값을 넣어줍니다.

```shell
$ npm root -g
/usr/local/lib/node_modules
```

![img](/assets/img/2025-1-7-local-hedera-network/3.png)


마지막으로 Settings > Advanced에 들어가 도커 소켓 사용`Allow the default Docker sockets to be used (requires password)`을 허용해줍니다.

![img](/assets/img/2025-1-7-local-hedera-network/4.png)


#### requirements

node.js, npm, docker, docker compose 버전과 RAM 용량은 다음과 같이 맞추어야 합니다.

- Node.js >= v20.11.0
- NPM >= v10.2.4
- Docker >= v27.3.1
- Docker Compose => v2.29.7
- Minimum 16GB RAM

### 헤데라 설치 및 시작

```shell
% npm install @hashgraph/hedera-local -g
% npm start
```

헤데라를 시작하면 다음과 같이 도커 이미지를 pull 받아오기 시작합니다.

![img](/assets/img/2025-1-7-local-hedera-network/0.png)

- 계정은 기본적으로 10개가 생성됩니다. `--accounts` 옵션을 통해 시작시 생성할 계정 수를 변경할 수 있습니다.
- `--h` / `--host` 옵션을 통해 호스트를 재정의할 수 있습니다.

```shell
% hedera start
[Hedera-Local-Node] INFO (StateController) [✔︎] Starting start procedure!
[Hedera-Local-Node] INFO (InitState) ⏳ Making sure that Docker is started and it is correct version...
[Hedera-Local-Node] INFO (DockerService) ⏳ Checking docker compose version...
[Hedera-Local-Node] INFO (DockerService) ⏳ Checking docker resources...
[Hedera-Local-Node] INFO (InitState) ⏳ Setting configuration with latest images on host 127.0.0.1 with dev mode turned off using turbo mode in single node configuration...
[Hedera-Local-Node] INFO (InitState) [✔︎] Local Node Working directory set to /Users/jisoo/Library/Application Support/hedera-local.
[Hedera-Local-Node] INFO (InitState) [✔︎] Hedera JSON-RPC Relay rate limits were disabled.
[Hedera-Local-Node] INFO (InitState) [✔︎] Needed environment variables were set for this configuration.
[Hedera-Local-Node] INFO (InitState) [✔︎] Needed bootsrap properties were set for this configuration.
[Hedera-Local-Node] INFO (InitState) [✔︎] Needed bootsrap properties were set for this configuration.
[Hedera-Local-Node] INFO (InitState) [✔︎] Needed mirror node properties were set for this configuration.
[Hedera-Local-Node] INFO (StartState) ⏳ Starting Hedera Local Node...
[Hedera-Local-Node] INFO (DockerService) ⏳ Pulling docker images...
[Hedera-Local-Node] INFO (StartState) ⏳ Detecting network...
[Hedera-Local-Node] INFO (StartState) [✔︎] Hedera Local Node successfully started!
[Hedera-Local-Node] INFO (NetworkPrepState) ⏳ Starting Network Preparation State...
[Hedera-Local-Node] INFO (NetworkPrepState) [✔︎] Imported fees successfully!
[Hedera-Local-Node] INFO (NetworkPrepState) [✔︎] Topic was created!
[Hedera-Local-Node] INFO (AccountCreationState) ⏳ Starting Account Creation state in synchronous mode ...
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------| Accounts list (ECDSA keys) |----------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |    id    |                            private key                            |  balance |
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1002 - 0x7f109a9e3b0d8ecfba9cc23a3614433ce0fa7ddcc80f2a8f10b222179a5a80d6 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1003 - 0x6ec1f2e7d126a74a1d2ff9e1c5d90b92378c725e506651ff8bb8616a5c724628 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1004 - 0xb4d7f7e82f61d81c95985771b8abf518f9328d019c36849d4214b5f995d13814 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1005 - 0x941536648ac10d5734973e94df413c17809d6cc5e24cd11e947e685acfbd12ae - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1006 - 0x5829cf333ef66b6bdd34950f096cb24e06ef041c5f63e577b4f3362309125863 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1007 - 0x8fc4bffe2b40b2b7db7fd937736c4575a0925511d7a0a2dfc3274e8c17b41d20 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1008 - 0xb6c10e2baaeba1fa4a8b73644db4f28f4bf0912cceb6e8959f73bb423c33bd84 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1009 - 0xfe8875acb38f684b2025d5472445b8e4745705a9e7adc9b0485a05df790df700 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1010 - 0xbdc6e0a69f2921a78e9af930111334a41d3fab44653c8de0775572c526feea2d - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1011 - 0x3e215c3d2a59626a669ed04ec1700f36c05c9b216e592f58bbfd3d8aa6ea25f9 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |--------------------------------------------------------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |------------------------------------------------| Accounts list (Alias ECDSA keys) |--------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |--------------------------------------------------------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |    id    |               public address               |                             private key                            | balance |
[Hedera-Local-Node] INFO (AccountCreationState) |--------------------------------------------------------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1012 - 0x67d8d32e9bf1a9968a5ff53b87d777aa8ebbee69 - 0x105d050185ccb907fba04dd92d8de9e32c18305e097ab41dadda21489a211524 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1013 - 0x05fba803be258049a27b820088bab1cad2058871 - 0x2e1d968b041d84dd120a5860cee60cd83f9374ef527ca86996317ada3d0d03e7 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1014 - 0x927e41ff8307835a1c081e0d7fd250625f2d4d0e - 0x45a5a7108a18dd5013cf2d5857a28144beadc9c70b3bdbd914e38df4e804b8d8 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1015 - 0xc37f417fa09933335240fca72dd257bfbde9c275 - 0x6e9d61a325be3f6675cf8b7676c70e4a004d2308e3e182370a41f5653d52c6bd - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1016 - 0xd927017f5a6a7a92458b81468dc71fce6115b325 - 0x0b58b1bd44469ac9f813b5aeaf6213ddaea26720f0b2f133d08b6f234130a64f - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1017 - 0x5c41a21f14cfe9808cbec1d91b55ba75ed327eb6 - 0x95eac372e0f0df3b43740fa780e62458b2d2cc32d6a440877f1cc2a9ad0c35cc - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1018 - 0xcdad5844f865f379bea057fb435aefef38361b68 - 0x6c6e6727b40c8d4b616ab0d26af357af09337299f09c66704146e14236972106 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1019 - 0x6e5d3858f53fc66727188690946631bde0466b1a - 0x5072e7aa1b03f531b4731a32a021f6a5d20d5ddc4e55acbb71ae202fc6f3a26d - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1020 - 0x29cbb51a44fd332c14180b4d471fbbc6654b1657 - 0x60fe891f13824a2c1da20fb6a14e28fa353421191069ba6b6d09dd6c29b90eff - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1021 - 0x17b2b8c63fa35402088640e426c6709a254c7ffb - 0xeae4e00ece872dd14fb6dc7a04f390563c7d69d16326f2a703ec8e0934060cc7 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) |--------------------------------------------------------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------| Accounts list (ED25519 keys) |----------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |    id    |                            private key                            |  balance |
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1022 - 0xa608e2130a0a3cb34f86e757303c862bee353d9ab77ba4387ec084f881d420d4 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1023 - 0xbbd0894de0b4ecfa862e963825c5448d2d17f807a16869526bff29185747acdb - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1024 - 0x8fd50f886a2e7ed499e7686efd1436b50aa9b64b26e4ecc4e58ca26e6257b67d - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1025 - 0x62c966ebd9dcc0fc16a553b2ef5b72d1dca05cdf5a181027e761171e9e947420 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1026 - 0x805c9f422fd9a768fdd8c68f4fe0c3d4a93af714ed147ab6aed5f0ee8e9ee165 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1027 - 0xabfdb8bf0b46c0da5da8d764316f27f185af32357689f7e19cb9ec3e0f590775 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1028 - 0xec299c9f17bb8bdd5f3a21f1c2bffb3ac86c22e84c325e92139813639c9c3507 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1029 - 0xcb833706d1df537f59c418a00e36159f67ce3760ce6bf661f11f6da2b11c2c5a - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1030 - 0x9b6adacefbbecff03e4359098d084a3af8039ce7f29d95ed28c7ebdb83740c83 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1031 - 0x9a07bbdbb62e24686d2a4259dc88e38438e2c7a1ba167b147ad30ac540b0a3cd - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) [✔︎] Accounts created succefully!
[Hedera-Local-Node] INFO (CleanUpState) ⏳ Initiating clean up procedure. Trying to revert unneeded changes to files...
[Hedera-Local-Node] INFO (CleanUpState) [✔︎] Clean up of consensus node properties finished.
[Hedera-Local-Node] INFO (CleanUpState) [✔︎] Clean up of mirror node properties finished.
```

### 헤데라 중지 및 재시작

로컬 네트워크 중지 및 재시작 명령어는 다음과 같습니다.

```shell
% hedera stop
% hedera restart
```

```shell
% sudo hedera stop
[Hedera-Local-Node] INFO (StateController) [✔︎] Starting stop procedure!
[Hedera-Local-Node] INFO (StopState) ⏳ Initiating stop procedure. Trying to stop docker containers and clean up volumes...
[Hedera-Local-Node] INFO (StopState) ⏳ Stopping the network...
[Hedera-Local-Node] INFO (StopState) [✔︎] Hedera Local Node was stopped successfully.
```

> 참고로 필자는 여기서 에러가 나 몇시간째 재시작이 되지 않았는데,, <br>
> 초반 로그에서 `Local Node Working directory set to /Users/jisoo/Library/Application Support/hedera-local.` 이렇게 나오는 working directory를 지우고, <br>
> image와 container까지 전부 삭제한 뒤 다시 `hedera start`를 하니 정상적으로 작동되었습니다.
{: .prompt-tip}


## 스마트 컨트랙트

> [Hedera 공식 문서: Smart Contract](https://docs.hedera.com/hedera/core-concepts/smart-contracts)를 참고하여 작성되었습니다.
{: .prompt-info}

### 스마트 컨트랙트 작성

`HelloHedera.sol` 파일을 만들어 간단한 solidity 코드를 작성합니다.

- `set_message()` : `message` 설정 함수
- `get_message()` : `message` 확인 함수

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.17;

contract HelloHedera {
    string message = "Hello Hedera!";

    function set_message(string memory message_) public {
        message = message_;
    }

    function get_message() public view returns (string memory) {
        return message;
    }
}
```

### 스마트 컨트랙트 컴파일

아래 명령어를 통해 `HelloHedera.sol` 파일을 컴파일합니다.
- `--abi`, `--bin` 옵션을 붙여 컴파일 결과로 abi, bin 파일을 확인할 수 있습니다.

```shell
% solcjs --abi --bin HelloHedera.sol
```

> [Hedera 공식 문서 (SDKs)](https://docs.hedera.com/hedera/sdks-and-apis/sdks)를 참고하여 작성하였습니다.
{: .prompt-info}

## 스마트 컨트랙트 배포 

### 프로젝트 세팅

먼저, 프로젝트 디렉토리 `hello-hedera` 를 만들고, 그 안에 solidity 코드, abi, bin 파일을 넣어주었습니다.

그리고 npm을 통해 [hedera javascript sdk](https://github.com/hashgraph/hedera-sdk-js)를 설치합니다.

```shell
% npm install --save @hashgraph/sdk
```

![img](/assets/img/2025-1-7-local-hedera-network/5.png){: .shadow }

<br>

javascript와 node를 통해 스마트 컨트랙트 배포 및 실행을 진행할 예정입니다.

프로젝트 디렉토리에 `hello-hedera.js`라는 파일을 만들어주고, 그 안에 코드를 작성합니다.


### client 설정

먼저 client를 설정합니다.

`mirror`, `node endpoint`는 따로 설정하지 않은 이상, 로컬에서는 각각 `localhost:5600`, `localhost:50211` 입니다.

`MY_ACCOUNT_ID`와 `MY_PRIVATE_KEY`는 위 로그에서 나오는 계정 중 아무거나 골라서 사용하면 되는데, 위에 다음과 같이 `(ECDSA keys)` 인지 `(ED25519 keys)` 인지 확인할 수 있습니다. <br>
계정을 고르고, private key가 ECDSA인지 ED25519인지 확인하고, `PrivateKey.fromStringECDSA()` 혹은 `PrivateKey.fromStringED25519()` 를 사용하면 됩니다.
- 아래 예시에서는 제일 처음인 `0.0.1002` 아이디의 `0x7f109a9e3b0d8ecfba9cc23a3614433ce0fa7ddcc80f2a8f10b222179a5a80d6` key를 사용해주었습니다.

> local network를 설정할 때 나온 account list

```shell
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------| Accounts list (ECDSA keys) |----------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) |    id    |                            private key                            |  balance |
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1002 - 0x7f109a9e3b0d8ecfba9cc23a3614433ce0fa7ddcc80f2a8f10b222179a5a80d6 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1003 - 0x6ec1f2e7d126a74a1d2ff9e1c5d90b92378c725e506651ff8bb8616a5c724628 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1004 - 0xb4d7f7e82f61d81c95985771b8abf518f9328d019c36849d4214b5f995d13814 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1005 - 0x941536648ac10d5734973e94df413c17809d6cc5e24cd11e947e685acfbd12ae - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1006 - 0x5829cf333ef66b6bdd34950f096cb24e06ef041c5f63e577b4f3362309125863 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1007 - 0x8fc4bffe2b40b2b7db7fd937736c4575a0925511d7a0a2dfc3274e8c17b41d20 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1008 - 0xb6c10e2baaeba1fa4a8b73644db4f28f4bf0912cceb6e8959f73bb423c33bd84 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1009 - 0xfe8875acb38f684b2025d5472445b8e4745705a9e7adc9b0485a05df790df700 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1010 - 0xbdc6e0a69f2921a78e9af930111334a41d3fab44653c8de0775572c526feea2d - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) | 0.0.1011 - 0x3e215c3d2a59626a669ed04ec1700f36c05c9b216e592f58bbfd3d8aa6ea25f9 - 10000 ℏ |
[Hedera-Local-Node] INFO (AccountCreationState) |-----------------------------------------------------------------------------------------|
```

<br>

> client를 설정하는 js 코드

```javascript
/**
 * build hedera client
 */
const mirror = "localhost:5600";
const node = { "localhost:50211": new AccountId(3) };

const MY_ACCOUNT_ID = AccountId.fromString("0.0.1002");
const MY_PRIVATE_KEY = PrivateKey.fromStringECDSA(
"0x7f109a9e3b0d8ecfba9cc23a3614433ce0fa7ddcc80f2a8f10b222179a5a80d6"
);

const client = Client.forNetwork(node)
.setMirrorNetwork(mirror)
.setOperator(MY_ACCOUNT_ID, MY_PRIVATE_KEY);

```

### file 생성 및 contract 생성 트랜잭션 실행

다음으로 위에서 작성한 smartcontract 파일을 컴파일한 bin 파일을 올리는 트랜잭션과, 올려진 파일을 통해 contract를 생성하는 트랜잭션을 실행해보도록 하겠습니다.

먼저, fs 라이브러리를 사용하여 bin 파일 내용을 읽어옵니다. `fs.readFileSync("HelloHedera_sol_HelloHedera.bin");`
- 또는 bin 파일 내용을 가져와 문자열로 넣어주어도 됩니다.

`new FileCreateTransaction().setContents(bytecode);` 를 통해 bytecode를 내용으로 설정한 `FileCreateTransaction` 인스턴스를 생성하고, client를 통해 실행`execute()`해줍니다.

실행 뒤, `getReceipt()`를 통해 receipt(부가 정보)를 가져오고 그 속에 `fileId`를 가져오면, 컨트랙트 생성 트랜잭션을 실행할 준비가 완료됩니다.


```javascript
  /**
   * create the transaction
   */
  const bytecode = fs.readFileSync("HelloHedera_sol_HelloHedera.bin");

  console.log("[1] create the transaction");
  const fileCreateTx = await new FileCreateTransaction().setContents(bytecode);
  const fileCreateTxRes = await fileCreateTx.execute(client);
  const fileCreateReceipt = await fileCreateTxRes.getReceipt(client);
  const fileId = fileCreateReceipt.fileId;
  console.log("[1] bytecode file id " + fileId);
```

<br>

`ContractCreateTransaction` 인스턴스에 `gas`를 설정하고, `setBytecodeFileId()` 메소드를 통해 위에서 나온 `fileId`를 설정해줍니다.

이 트랜잭션 역시도 `client`로 실행하고, receipt를 확인하면 `contractId`를 확인할 수 있습니다.

추후 **`contractId(아래 코드에서는 newContractId 변수)`는 스마트 컨트랙트를 실행할 때 계속 필요한 값**이니, 기억해두면 좋습니다.


```javascript
  const contractTx = await new ContractCreateTransaction()
    .setGas(3000000)
    .setBytecodeFileId(fileId);
  const modifyTransactionFee = contractTx.setMaxTransactionFee(new Hbar(16));
  const contractResponse = await modifyTransactionFee.execute(client);
  const contractReceipt = await contractResponse.getReceipt(client);
  const newContractId = contractReceipt.contractId;
  console.log("[1] smart contract ID: " + newContractId);
  console.log(
    "[1] The transaction consensus status is " + contractReceipt.status
  );
```

### 스마트 컨트랙트 함수 실행

#### set_message 함수

solidity로 작성한 `set_message()` 함수는 다음과 같습니다.

```
function set_message(string memory message_) public {
    message = message_;
}
```

이를 토대로 위 함수를 호출하는 코드는 다음과 같이 작성할 수 있습니다.

`ContractExecuteTransaction` (함수 호출 트랜잭션) 인스턴스를 만들고, 방금 전 스마트 컨트랙트 아이디인 `contractId`를 설정해줍니다.

호출하고나 하는 함수 이름과, 함께 전달할 매개변수를 `setFunction("함수명", 매개변수)` 형태로 작성합니다.
- `set_message` 함수는 문자열을 전달해야 하므로, `addString()` 을 통해 매개변수도 함께 전달해주었습니다.

> 여기서 주의해야할 점은, 문자열로 작성해야하는 **함수명**과 **매개변수 타입**에 유의해야 합니다. <br>
> **int32일 경우 `.addInt32`로**, **uint128일 경우, `.addUint128`로** 타입에 잘 맞게 매개변수를 설정해주어야 합니다. <br>
> ~~왜이렇게 당연한 소리를 하냐구요? 필자는 당연한 걸 놓쳐 n시간을 헤맸습니다!~~
{: .prompt-warning}

이 트랜잭션 역시도 `client`로 실행하고, receipt를 확인하면 `status` 상태를 확인할 수 있습니다.
- `recipt`의 `status`값이 **`22`**일 경우 성공적으로 실행된 것입니다!


```javascript
// set message
console.log("\n[2] call smartcontract function");
var transaction = await new ContractExecuteTransaction()
.setContractId(newContractId)
.setGas(100000)
.setFunction(
    "set_message",
    new ContractFunctionParameters().addString("Hello Hedera again!!")
);

var txResponse = await transaction.execute(client);
var receipt = await txResponse.getReceipt(client);
console.log("[2] receipt :: " + receipt);
var transactionStatus = receipt.status;
console.log("[2] receipt status :: " + transactionStatus);
```

#### get_message (view) 함수

`get_message` 함수는 다음과 같이 변수의 상태를 변경하지 않고, 단순히 값을 읽어오는 `view`함수 였습니다.

```
function get_message() public view returns (string memory) {
    return message;
}
```

따라서 call query를 만드는 `ContractCallQuery` 인스턴스에 
1. `setContractId()` 메소드를 통해 `contractId`를 설정하고, 
2. `setFunction()` 메소드를 통해 `get_message` 함수를 호출함을 설정합니다.

실행 뒤 나온 `contractCallResult`값에 getString(0)을 해주면 방금 전, set_message 함수를 통해 설정한 `"Hello Hedera again!!"` 이라는 문자열을 확인할 수 있습니다.

```javascript
// get message
console.log("\n[3] call smartcontract function");
const query = await new ContractCallQuery()
.setContractId(newContractId)
.setGas(100000)
.setFunction("get_message");
const contractCallResult = await query.execute(client);
const result = contractCallResult.getString(0);
console.log("[3] contract message: " + result);
```

### 최종 javascript 코드

따라서 지금까지 내용을 모두 종합한 javascript 코드는 다음과 같습니다.

```javascript
const {
  Client,
  PrivateKey,
  Hbar,
  ContractCallQuery,
  AccountId,
  FileCreateTransaction,
  ContractExecuteTransaction,
  ContractCreateTransaction,
  ContractFunctionParameters,
} = require("@hashgraph/sdk");
const fs = require("fs");

async function main() {
  /**
   * build hedera client
   */
  const mirror = "localhost:5600";
  const node = { "localhost:50211": new AccountId(3) };

  const MY_ACCOUNT_ID = AccountId.fromString("0.0.1002");
  const MY_PRIVATE_KEY = PrivateKey.fromStringECDSA(
    "0x7f109a9e3b0d8ecfba9cc23a3614433ce0fa7ddcc80f2a8f10b222179a5a80d6"
  );

  const client = Client.forNetwork(node)
    .setMirrorNetwork(mirror)
    .setOperator(MY_ACCOUNT_ID, MY_PRIVATE_KEY);

  /**
   * create the transaction
   */

  const bytecode = fs.readFileSync("HelloHedera_sol_HelloHedera.bin");

  console.log("[1] create the transaction");
  const fileCreateTx = await new FileCreateTransaction().setContents(bytecode);
  const fileCreateTxRes = await fileCreateTx.execute(client);
  const fileCreateReceipt = await fileCreateTxRes.getReceipt(client);
  const fileId = fileCreateReceipt.fileId;
  console.log("[1] bytecode file id " + fileId);

  const contractTx = await new ContractCreateTransaction()
    .setGas(3000000)
    .setBytecodeFileId(fileId);
  const modifyTransactionFee = contractTx.setMaxTransactionFee(new Hbar(16));
  const contractResponse = await modifyTransactionFee.execute(client);
  const contractReceipt = await contractResponse.getReceipt(client);
  const newContractId = contractReceipt.contractId;
  console.log("[1] smart contract ID: " + newContractId);
  console.log(
    "[1] The transaction consensus status is " + contractReceipt.status
  );

  /**
   * call smartcontract function
   */

  // set message
  console.log("\n[2] call smartcontract function");
  var transaction = await new ContractExecuteTransaction()
    .setContractId(newContractId)
    .setGas(100000)
    .setFunction(
      "set_message",
      new ContractFunctionParameters().addString("Hello Hedera again!!")
    );

  var txResponse = await transaction.execute(client);
  var receipt = await txResponse.getReceipt(client);
  console.log("[2] receipt :: " + receipt);
  var transactionStatus = receipt.status;
  console.log("[2] receipt status :: " + transactionStatus);

  // get message
  console.log("\n[3] call smartcontract function");
  const query = await new ContractCallQuery()
    .setContractId(newContractId)
    .setGas(100000)
    .setFunction("get_message");
  const contractCallResult = await query.execute(client);
  const result = contractCallResult.getString(0);
  console.log("[3] contract message: " + result);
}
void main();

```

### 실행 결과

```shell
% node hello-hedera.js 
```

위 명령어를 통해 실행하면 다음과 같은 결과를 확인할 수 있습니다.

![img](/assets/img/2025-1-7-local-hedera-network/6.png){: .shadow }

```shell
[1] create the transaction
[1] bytecode file id 0.0.1032
[1] smart contract ID: 0.0.1033
[1] The transaction consensus status is 22

[2] call smartcontract function
[2] receipt :: {"status":"SUCCESS","accountId":null,"filedId":null,"contractId":"0.0.1033","topicId":null,"tokenId":null,"scheduleId":null,"exchangeRate":{"hbars":1,"cents":12,"expirationTime":"1963-11-25T17:31:44.000Z","exchangeRateInCents":12},"topicSequenceNumber":"0","topicRunningHash":"","totalSupply":"0","scheduledTransactionId":null,"serials":[],"duplicates":[],"children":[],"nodeId":"0"}
[2] receipt status :: 22

[3] call smartcontract function
[3] contract message: Hello Hedera again!!
```
