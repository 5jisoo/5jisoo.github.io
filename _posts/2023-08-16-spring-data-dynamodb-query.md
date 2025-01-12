---
title: Spring Data DynamoDB 쿼리 작업 정리
date: 2023-08-16 18:20:00 +/-TTTT
categories: [Project, POCHAK]
tags: [spring, dynamodb]     # TAG names should always be lowercase
---

> 'POCHAK'을 개발하며 Spring Data DynamoDB를 사용해보았고,  
> 그 과정에서 새롭게 알게 된 점을 정리한 글입니다.  
>   
> ※ 이후 포착은 서비스 특성상 데이터베이스를 DynamoDB에서 MySQL로 변경 후 다시 개발하였습니다.  
>   
> - [Spirng Data DynamoDB 라이브러리](https://github.com/boostchicken/spring-data-dynamodb)  
> - [POCHAK GitHub Repository](https://github.com/SMWU-POCHAK/POCHAK-Server)
{: .prompt-info}

## Query

### 구현 목표

KeyConditionExpression 작동 확인하기

-   PK가 동일하고 SK가 USER#로 시작하는 데이터를 전부 가져오기:  
    **.withKeyConditionExpression("#PK = :val1 and begins\_with(#SK, :val2)")**

### 예시 코드

1.  PK, SK의 이름과, 각각 비교할 값들을 HashMap에 저장시킴.
2.  DynamoDBQueryExperession 클래스를 만들어서
    1.  KeyConditionExpression을 전달
    2.  PK, SK의 이름을 전달
    3.  그리고 각각 PK와 비교할 값과 SK가 시작할 값을 전달하기
3.  최종적으로 mapper에 쿼리를 날리면 해당하는 User List를 가져올 수 있다!

``` java
public User findUserByUserHandle(String userHandle) throws BaseException {

    HashMap<String, String> ean = new HashMap<>(); // attribute names
    ean.put("#PK", "PartitionKey");
    ean.put("#SK", "SortKey");

    Map<String, AttributeValue> eav = new HashMap<>(); // attribute value
    eav.put(":val1", new AttributeValue().withS(userHandle));
    eav.put(":val2", new AttributeValue().withS("USER#"));

    DynamoDBQueryExpression<User> query = new DynamoDBQueryExpression<User>()
            .withKeyConditionExpression("#PK = :val1 and begins_with(#SK, :val2)")
            .withExpressionAttributeValues(eav)
            .withExpressionAttributeNames(ean);

    List<User> users = mapper.query(User.class, query);

    if (users.isEmpty()) { // 결과값이 비어있다면 - 예외 처리
        throw new BaseException(INVALID_USER_HANDLE);
    }
    return users.get(0);
}
```

---

## Sorting

### 구현 목표

최신 순 정렬하기

-   SK(allowedDate)를 기준으로 게시글을 최신 순 정렬(내림차순 - desc)한 값을 가져오기  
    (기본은 오래된 순으로 정렬됨.):  
    **.withScanIndexForward(false); // desc**

### 예시 코드

``` java
public List<Tag> findTagsByUserHandle(String userHandle) throws BaseException {

    HashMap<String, String> ean = new HashMap<>();
    ean.put("#PK", "PartitionKey");
    ean.put("#SK", "SortKey");

    Map<String, AttributeValue> eav = new HashMap<>();
    eav.put(":val1", new AttributeValue().withS(userHandle));
    eav.put(":val2", new AttributeValue().withS("TAG#"));

    DynamoDBQueryExpression<Tag> query = new DynamoDBQueryExpression<Tag>()
            .withKeyConditionExpression("#PK = :val1 and begins_with(#SK, :val2)")
            .withExpressionAttributeValues(eav)
            .withExpressionAttributeNames(ean)
            .withScanIndexForward(false); // desc

    List<Tag> tags = mapper.query(Tag.class, query);

    return tags;
}
```

---

## Query with Filter

### 구현 목표

KeyCondition으로 나온 결과에 필터 적용하기

-   status가 PUBLIC인 데이터만 가져오기:  
    **.withFilterExpression("#STATUS = :val3") // filter - get only public publish**

### 예시 코드

```java
public List<Publish> findOnlyPublicPublishWithUserHandle(String userHandle) throws BaseException {

    HashMap<String, String> ean = new HashMap<>();
    ean.put("#PK", "PartitionKey");
    ean.put("#SK", "SortKey");
    ean.put("#STATUS", "status");

    Map<String, AttributeValue> eav = new HashMap<>();
    eav.put(":val1", new AttributeValue().withS(userHandle));
    eav.put(":val2", new AttributeValue().withS("PUBLISH#"));
    eav.put(":val3", new AttributeValue().withS(Status.PUBLIC.toString()));

    DynamoDBQueryExpression<Publish> query = new DynamoDBQueryExpression<Publish>()
            .withKeyConditionExpression("#PK = :val1 and begins_with(#SK, :val2)")
            .withFilterExpression("#STATUS = :val3") // filter - get only public publish
            .withExpressionAttributeValues(eav)
            .withExpressionAttributeNames(ean)
            .withScanIndexForward(false); // desc

    List<Publish> publishes = mapper.query(Publish.class, query);
    return publishes;
}
```

---

## 수정 쿼리 작성하기

> Java의 List에 add, remove 메서드를 사용해도 되지만, 데이터베이스 쿼리로 구현하는 방법도 찾아보았습니다.

### 구현 목표

-   팔로우 중인 상태면 팔로우 취소하기
-   반대 상태라면? 팔로우하기

### 구현 원리

먼저 기존에 List로 구현되어있던 팔로우/팔로잉 목록을 HashSet으로 변경해주었습니다.

-   일단 UserHandle이 겹쳐서는 안되기 때문에 Set을 사용하는 편이 로직에 맞았고, String Set이 구현에 더 편리합니다.

isFollow라는 현재 팔로우 상태를 알아보는 Boolean 값을 받았습니다.

-   이는 List의 contains를 사용해주어도 되고, 저는 별도의 쿼리를 사용하였습니다.
-   isFollow() 쿼리 로직은 위의 설명이 충분히 나와있으니 설명은 생략하겠습니다~~

isFollow를 통해 현재 action ADD 또는 DELETE를 결정해주고, UpdateItemRequest를 작성할 수 있습니다.

-   작성법은 그냥 query와 매우 유사하며, 특히 withAttributeUpdates에 들어가는 HashMap에 해당 action을 추가해주면 됩니다.

**awsDynamoDB.updateItem(updateItemRequest)**를 통해 쿼리를 전송할 수 있으며, 오류 처리도 해주었습니다.

### 예시 코드

```java
public String followOrCancelByIsFollow(
    String followedUserHandle, 
    String followingUserHandle, 
    Boolean isFollow
) throws BaseException {

    String result = "성공적으로 팔로우하였습니다."; // add
    AttributeAction action = ADD;
    if (isFollow) {
        result = "성공적으로 팔로우를 취소하였습니다."; // delete
        action = DELETE;
    }

    // add or delete follower
    // User SK가 "USER#" 이 아니라 다른것으로 바뀐다면 바꿔야 함.
    HashMap<String, AttributeValue> followerItemKey = new HashMap<>();
    followerItemKey.put("PartitionKey", new AttributeValue().withS(followedUserHandle));
    followerItemKey.put("SortKey", new AttributeValue().withS("USER#"));

    HashMap<String, AttributeValueUpdate> followerUpdateValue = new HashMap<>();
    followerUpdateValue.put("followerUserHandles", new AttributeValueUpdate()
            .withValue(new AttributeValue().withSS(followingUserHandle))
            .withAction(action));

    UpdateItemRequest addFollower = new UpdateItemRequest()
            .withKey(followerItemKey)
            .withTableName("pochakdatabase")
            .withAttributeUpdates(followerUpdateValue);

    // add or delete following
    HashMap<String, AttributeValue> followingItemKey = new HashMap<>();
    followingItemKey.put("PartitionKey", new AttributeValue().withS(followingUserHandle));
    followingItemKey.put("SortKey", new AttributeValue().withS("USER#"));

    HashMap<String, AttributeValueUpdate> followingUpdateValues = new HashMap<>();
    followingUpdateValues.put("followingUserHandles", new AttributeValueUpdate()
            .withValue(new AttributeValue().withSS(followedUserHandle))
            .withAction(action));

    UpdateItemRequest addFollowing = new UpdateItemRequest()
            .withKey(followingItemKey)
            .withTableName("pochakdatabase")
            .withAttributeUpdates(followingUpdateValues);

    try {
        amazonDynamoDB.updateItem(addFollower);
        amazonDynamoDB.updateItem(addFollowing);
        return result;
    } catch (ResourceNotFoundException e) {
        throw new BaseException(RESOURCE_NOT_FOUND);
    } catch (AmazonDynamoDBException e) {
        throw new BaseException(DATABASE_ERROR);
    }
}
```

---

## 쿼리메소드 사용하기

> Spring Data JPA와 마찬가지로 쿼리 메소드를 사용할 수 있습니다.

### 구현 목표

-   User를 찾을 때 PK인 Handle과 SK의 prefix인 "USER#" 를 사용하여   
    **#PK = :val1 and begins\_with(#SK, :val2)**   
    쿼리를 날리고자 할 때, 쿼리 메소드를 사용하여 구현해보기

### 구현된 코드

쿼리 메소드 사용은 JPA와 사용이 동일합니다. 자세한 사용방법은 [공식문서](https://docs.spring.io/spring-data/jpa/reference/#repositories.query-methods.details)를 참고하세요!

```java
@EnableScan
public interface UserCrudRepository extends DynamoDBCrudRepository<User, UserId> {
    Optional<User> findUserByHandleAndUserSKStartingWith(String handle, String prefix);
}
```

### 주의 사항

여기서 만약 이렇게 작성하면, PK인 UserId만 가지고 쿼리를 날리게 됩니다. 이 경우 DB 설계를 고려해보았을 때 User가 아닌, PK가 동일한 Tag, Publish 등이 함께 찾아질 수 있으므로 유의합니다.

-   참고로 최신순 정렬 (내림차순 정렬)을 하고 싶은 경우, **ScanIndexForward**를 false로 주어야 하므로, 수동 쿼리를 작성해야합니다.

```java
@EnableScan
public interface UserCrudRepository extends DynamoDBCrudRepository<User, UserId> {
    Optional<User> findUserByHandle(String handle);
}
```

-   로그: PK가 동일한 USER, TAG가 함께 찾아진 결과

```
// PK로 "dayeon"을 주었을 때
"{"TableName":"pochakdatabase","ConsistentRead":true,"KeyConditions":{"PartitionKey":{"AttributeValueList":[{"S":"dayeon"}],"ComparisonOperator":"EQ"}},"ScanIndexForward":true}"


// 결과 - 다음과 같이 USER 뿐만 아니라 TAG도 찾아짐.
"{"Count":3,"Items":
[
// TAG
{"PartitionKey":{"S":"dayeon"},"postPK":{"S":"POST#test124"},"SortKey":{"S":"TAG#2023-08-10T00:12:35.451Z"},"postImg":{"S":"https://~~"},"status":{"S":"PUBLIC"}},

// TAG
{"PartitionKey":{"S":"dayeon"},"postPK":{"S":"POST#test123"},"SortKey":{"S":"TAG#2023-08-10T00:16:35.451Z"},"postImg":{"S":"https://~~"},"status":{"S":"PUBLIC"}},{"createdDate":{"S":"2023-08-12T06:15:17.847Z"},"status":{"S":"PRIVATE"},

// USER - 원래 찾고자 했던 데이터
"PartitionKey":{"S":"dayeon"},"followerUserHandles":{"SS":["jisoo"]},"message":{"S":"[0xed][0x95][0x9c] [0xec][0xa4][0x84] [0xec][0x86][0x8c][0xea][0xb0][0x9c] 111"},"lastModifiedDate":{"S":"2023-08-13T15:03:19.531Z"},"email":{"S":"dayeon@naver.com"},"SortKey":{"S":"USER#"},"name":{"S":"testUser1"},"profileImage":{"S":"https://11~~"}}],

// 결과 - 3개 (찾고자 하는 데이터 외에 다른 데이터가 섞임)
"ScannedCount":3}"
```

---

## 참고 자료

-   [DynamoDBMapper 쿼리 및 스캔 작업](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/DynamoDBMapper.QueryScanExample.html)
-   [DynamoDB의 쿼리 작업](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Query.html)
-   [Update an item in a DynamoDB table using an AWS SDK](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/example_dynamodb_UpdateItem_section.html)
-   [Update expressions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.UpdateExpressions.html)
-   [Update DynamoDB Items with Java - Published by Emmanouil Gkatziouras](https://egkatzioura.com/2016/08/08/update-dynamodb-items-with-java/)
-   [stack overflow - Deleting Attribute in DynamoDB](https://stackoverflow.com/questions/9142074/deleting-attribute-in-dynamodb)