---
title: 이더리움 플랫폼
date: 2024-12-13 14:01:00 +/-TTTT
categories: [Security, Blockchain]
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

<!--

## 이더리움 지갑의 일반 기능

# 디앱의 핵심 기능 : 스마트 컨트랙트


## 계정

## 이더

## 가스

## 콜과 트랜잭션

## EVM

# geth로 이더리움 네트워크에 접속

## geth 시작하기

## geth 대화형 콘솔 시작하기

## JSON-RPC 사용하기

## geth로 채굴하기

## 기타 클라이언트

# geth 계정 관리

## 이더리움 계정

## geth 명령어로 계정 관리

## geth 콘솔에서 Web3로 계정 관리

## JSON-RPC로 계정 관리

# 심플코인 컨트랙트 다시 보기

## SimpleCoin 컨트랙트 개선하기

## 개선된 코드 사용하기

## 이더리움 네트워크에서 코인 전송은 어떻게 실행되는가?

-->

---

> 「Building Ethereum Dapps 이더리움 디앱 개발」 (로베르토 인판테 저, 정종화 역)을 읽고 작성한 글입니다.
{: .prompt-tip}