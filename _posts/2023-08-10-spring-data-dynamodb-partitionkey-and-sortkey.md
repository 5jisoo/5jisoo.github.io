---
title: Partition Key와 Sort Key를 같이 사용할 때
date: 2023-08-10 18:20:00 +/-TTTT
categories: [Project, POCHAK]
tags: [spring, dynamodb, troubleshooting]     # TAG names should always be lowercase
---

## 문제 상황

POCHAK은 원테이블 설계로, `Partition Key`와 `Sort Key`를 모두 사용하여 설계하였습니다.

하지만 Spring Data DynamoDB와 연결하는 과정에서 `BeanCreationException`이 발생하였습니다.

이를 해결하기 위해 알아보는 과정에서 Partition Key와 Sort Key를 같이 사용하는 경우, **CrudRepository를 사용할 때 PK와 SK를 조합한 Id 클래스가 따로 있어야 한다는 점**을 알게되고 에러를 해결한 과정을 정리합니다!

## 문제 해결 과정

### UserId Class

-   UserId 클래스를 다음과 같이 설정합니다. User에서 설정한 PK와 SK의 조합으로 이루어집니다.
-   유의할 점은 UserId 클래스는 Serializable을 구현`implements`해야 합니다.

```java
public class UserId implements Serializable {
    private static final long serialVersionUID = 1L;

    private String handle;
    private String userSK = "USER#";

    @DynamoDBHashKey
    public String getHandle() {
        return handle;
    }

    public void setHandle(String handle) {
        this.handle = handle;
    }

    @DynamoDBRangeKey
    public String getUserSK() {
        return userSK;
    }

    public void setUserSK(String userSK) {
        this.userSK = userSK;
    }
}
```

### User Class

User의 PK, SK의 getter, setter를 ID에서 받아오고(getter), ID을 변경하도록(setter) 설정해줍니다.

- 참고로 보통 모든 Entity의 attribute는 SDK에서 사용하기 위해서 getter와 setter를 필수로 가지고 있어야 합니다. 다만, userId는 getter와 setter가 있어서는 안됩니다.
- 이 외에도 ID 클래스는 꼭 다음과 같이 **지연 생성**을 해주어야 합니다.

```java
@NoArgsConstructor
@DynamoDBTable(tableName = "pochakdatabase")
public class User extends BaseEntity {

    @Id // ID class should not have getter and setter.
    private UserId userId;

    private String handle; // PK

    private String userSK; // SK

    ...

    @DynamoDBHashKey(attributeName = "PartitionKey")
    public String getHandle() {
        return userId != null ? userId.getHandle() : null;
    }

    public void setHandle(String handle) {
        if (userId == null) {
            userId = new UserId();
        }
        userId.setHandle(handle);
    }

    @DynamoDBRangeKey(attributeName = "SortKey")
    public String getUserSK() {
        return userId != null ? userId.getUserSK(): null;
    }

    public void setUserSK(String userSK) {
        if (userId == null) {
            userId = new UserId();
        }
        userId.setUserSK(userSK);
    }
 ...
}
```

### UserCrudRepository interface

이렇게 설정하면 CrudRepository를 구현하는 UserCrudRepository 클래스는 다음과 같이 작성할 수 있습니다.

- 저장소는 꼭 ID 클래스 - 여기서는 UserId 클래스 - 를 사용해야 합니다.

```java
@EnableScan
public interface UserCrudRepository extends DynamoDBCrudRepository<User, UserId> {
}
```

## 참고 자료

-   [Use Hash Range Keys - derjust:spring-data-dynamodb](https://github.com/derjust/spring-data-dynamodb/wiki/Use-Hash-Range-keys)