---
title: 솔리디티로 스마트 컨트랙트 프로그래밍하기
date: 2024-12-20 9:51:00 +/-TTTT
categories: [Security, Building Ethereum Dapps]
tags: [security, blockchain, ethereum, solidity]
math: true
---

# 고급 컨트랙트 구조 

## 컨트랙트 선언

선언할 수 있는 항목은 다음과 같다.
- 상태 변수
- 이벤트
- 열거형
- 구조체
- 함수
- 함수 제어자

```
// SPDX-License-Identifier: UNLICENSED

pragma solidity 0.8.19;

contract AuthorizedToken {
    // 열거형 정의
    enum UserType {TokenHolder, Admin, Owner}

    // 구조체 정의
    struct AccountInfo {
        address accouint;
        string firstName;
        string lastName;
        UserType UserType;
    }

    // 상태 변수 정의
    mapping (address => uint256) public tokenBalance;
    mapping (address => AccountInfo) public registeredAccount;
    mapping (address => bool) public frozenAccount;

    address public owner;

    uint256 public constant maxTransferLimit = 15000;

    // 이벤트 정의
    event Transfer(address indexed from, address indexed to, uint value);
    event FrozenAccount(address target, bool frozen);

    // 함수 제어자 정의
    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }

    // 생성자 정의
    constructor(uint256 _initialSupply) public {
        owner = msg.sender;
        mintToken(owner, _initialSupply);
    }

    // 함수 정의
    function transfer(address _to, uint256 _amount) public {
        require(checkLimit(_amount));
        // ...
        emit Transfer(msg.sender, _to, _amount);
    }

    function registerAccount(
        address account, 
        string firstName, 
        string lastName, 
        bool isAdmin
    ) public onlyOwner {
            // ...
    }

    function checkLimit(uint256 _amount) private returns (bool) {
        if (_amount < maxTransferLimit) 
            return true;
        return false;
    }

    function validateAccount(address _account) internal returns (bool) {
        if (frozenAccount[_account] && tokenBalance[_account] > 0)
            return true;
        return false;
    }

    // 함수 제어자를 활용하여 함수 정의
    function mintToken(
        address _recipient, 
        uint256 _mintedAmount
    ) onlyOwner public {
        tokenBalance[_recipient] += _mintedAmount;
        emit Transfer(owner, _recipient, _mintedAmount);
    }

    function freezeAccount(
        address target,
        bool freeze
    ) onlyOwner public {
        frozenAccount[target] = freeze;
        emit FrozenAccount(target, freeze);
    }

}
```

### 상태 변수

상태 변수는 컨트랙트의 상태를 저장한다.

매핑과 같은 일부 유형은 상태 변수에만 사용할 수 있다.

명시적, 암시적 접근 수준도 상태 변수를 선언에 포함한다.

### 이벤트

이벤트는 컨트랙트 멤버로 EVM 트랜잭션 로그와 상호작용한다.

이벤트가 발생하면 이벤트를 참조하는 클라이언트에 전달되고 관련된 콜백 함수를 호출한다.

### 열거형

지정된 허용값 집합. 사용자 정의 유형.

### 구조체

각각 다른 유형의 변수를 묶는 사용자 정의 유형.

### 함수

컨트랙트 로직을 요약하고 제어자로 변경할 수 있다.

상태 변수에 접근할 수 있으며 컨트랙트에 선언된 이벤트를 발생시킬 수 있다.

### 함수 제어자

함수의 동작을 제어한다. 보통 함수를 선언할 때 특정 값으로 입력을 제한하기 위해 사용한다.

하나의 컨트랙트에서도 몇몇 함수를 사용하기 위해 많은 제어자를 선언할 수 있다.

# 솔리디티의 주요 요소

## 값형

값형 변수는 EVM 스택에 저장되며 값을 보유한 단일 메모리 공간을 할당한다.

값형 변수가 다른 변수에 지정되거나 매개변수로 함수에 전달되면 해당 값이 **변수의 새로운 인스턴스에 복사**된다.

따라서 할당된 변숫값을 변경해도 **원래 변숫값에는 영향을 미치지 않는다**.

### 불형

`bool`로 선언된 변수는 `true` 또는 `false`이다.

```
bool isComplete = true;
```

> C, C++와 같이 true, false를 1, 0으로 정의하는 것은 솔리디티에서 불가능하다.
{: .prompt-tip}

### 정수형

정수형 변수는 `int`(부호 있음) 또는 `unit`(부호 없음) 으로 선언할 수 있다.

8비트에서 256비트까지 8의 배수로 정확한 크기를 지정할 수 있다. (지정하지 않으면 256비트로 설정된다.)
- `int32` : 부호있는 32비트 정수
- `uint128` : 부호없는 128비트 정수

#### 암시적 및 명시적 정수형 변환

서로 다른 정수 유형으로 선언된 변수 사이에서는 의미 있는 형 변환만 가능하다.

일반적으로 할당받는 변수의 유형이 덜 제한적이거나 더 커야한다. 이런 경우 암시적 변환이 발생한다.

```
contract IntConversions {
    int256 bigNumber = 150000000000;
    int32 mediumNegativeNumber = -450000;
    uint16 smallPositiveNumber = 15678;

    int16 newSmallNumber = bigNumber; // 1. 컴파일 에러
    uint64 newMediumPositiveNumber = mediumNegativeNumber; // 2. 컴파일 에러
    uint32 newMediumNumber = smallPositiveNumber;  // 3. 암시적 변환
    int256 newBigNumber = mediumNegativeNumber; // 4. 암시적 변환
}
```

1. 컴파일 에러 : newSmallNumber가 너무 작아 bigNumber를 포함할 수 없어 컴파일 에러 발생
2. 컴파일 에러 : uint는 양수만 저장할 수 있으므로 음수 저장시 컴파일 에러 발생
3. 암시적 변환 : uint32로 암시적으로 변환됨. `newMediumNumber = 15,678`
4. 암시적 변환 : int256으로 변환됨. `newBigNumber = -450,000`

<br>

암시적인 변환이 허용되지 않는 경우에도 명시적인 변환을 수행할 수 있다. 사용자는 이때 전환이 유효한지 확인해야 한다.
- 명시적인 정수 변환은 직관적이지 않다!!

### 고정 바이트 배열

1에서 32 사이의 크기로 고정 크기의 바이트 배열을 선언할 수 있다.
- bytes8, bytes12

### 주소

주소`address` 객체는 0x로 시작하고 최대 40자리의 16진수로 구성되는 문자열로 선언한다. 주소 객체는 20바이트를 사용한다.

balance 속성을 조회하면 주소의 이더 잔액을 Wei 단위로 확인할 수 있다.

```
function getBalance(address _address) public view returns (uint) {
    uint balance = _address.balance;
    return balance;
}
```

이더를 전송하기 위한 주소 유형으로 다양한 함수를 제공한다. 

> 보안 문제 때문에 `send()` 및 `call()`은 더 이상 사용되지 않는다.

- `transfer()`
    - Wei 단위로 이더를 전송한다.
    - 트랜잭션이 실패하면 발신자에게 예외가 발생하며, 가스는 환불되지 않지만 지불 내용은 자동으로 환불된다.
    - 최대 2300개의 가스를 소비할 수 있다.
- `send()`
    - Wei 단위로 이더를 전송한다.
    - 수신하는 곳에서 트랜잭션이 실패하면 발신자에게 false값이 반환되지만 지불 내용은 환불되지 않는다.
    - 최대 2300개의 가스를 소비할 수 있다.
- `call()`
    - 전체 가스 예산을 발신자에서 호출된 함수로 전송한다.
    - 실패하면 false를 반환하므로 `send()`와 마찬가지로 실패를 처리해야 함.

#### transfer()

`destinationAddress`로 10Wei를 전송한다.

```
destinationAddress.transfer(10);
```

#### send()

`send()` 함수로 이더를 보내는 경우 이더를 잃지 않으려면 오류를 처리해야 한다.

```
if (!destinationAddress.send(10))
    revert();
```

`send()`가 실패하면 false를 반환한다. false를 반환할 경우 후속작업 rever()를 실행한다.

#### call()

`call()`을 사용하면 외부 컨트랙트 함수를 호출할 수 있다.

```
destinationContractAddress.call("contractName", "functionName");
```

다음과 같이 외부 호출을 하면서 이더를 보낼 수 있다.

```
if (!destinationContractAddress.call.value(10)("contractName", "functionName"))
    revert();
```

`value()` 함수의 Wei 이더 수량만큼 이더를 외부 `call()`로 전송한다.

`send()`와 마찬가지로 상태를 되돌리기위해 외부 함수 호출 오류 역시도 처리해야 한다.

### 열거형

명명된 값으로 구성되는 집합으로 사용자 지정 데이터 형식이다.

```
enum InvestmentLevel {High, Medium, Low} // 선언

InvestmentLevel level = InvestmentLevel.Low; // 정의
```

> 아직 작성하고 있는 게시글입니다!

<!-- ## 참조형

## 글로벌 네임스페이스

## 상태 변수

## 함수

## 함수 제어자

## 변수 선언과 초기화 및 할당

## 이벤트

## 조건문

-->