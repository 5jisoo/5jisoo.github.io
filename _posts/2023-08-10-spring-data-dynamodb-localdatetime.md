---
title: LocalDateTime 저장 에러
date: 2023-08-10 18:20:00 +/-TTTT
categories: [Spring, Spring Data DynamoDB]
tags: [spring, dynamodb, troubleshooting]     # TAG names should always be lowercase
---

## 문제 상황

LocalDateTime을 DynamoDB에 저장하려고 했으나, 다음과 같은 에러가 발생하였습니다.

```
InvalidDefinitionException: Joda date/time type `org.joda.time.LocalDateTime` not supported by default
```

## 해결 과정

LocalDateTime parsing 에러로 추측하였으며,

직접 LocalDateTime을 바꿔주는 Custom Converter를 작성하여 해당 에러를 해결해 줄 수 있었습니다.

```java
public static class LocalDateTimeConverter implements DynamoDBTypeConverter<Date, LocalDateTime> {
    @Override
    public Date convert(LocalDateTime source) {
        return Date.from(source.toInstant(ZoneOffset.UTC));
    }

    @Override
    public LocalDateTime unconvert(Date source) {
        return source.toInstant().atZone(TimeZone.getDefault().toZoneId()).toLocalDateTime();
    }
}
```

이후, 사용하고자 하는 Entity에 적용해주면 됩니다.

-   POCHAK의 경우, BaseEntity의 LocalDateTime attribute에 **@DynamoDBTypeConverted(converter = LocalDateTimeConverter.class)** 애노테이션을 추가해주었습니다.

```java
public class BaseEntity {
    
    ...

    @LastModifiedDate
    @DynamoDBAttribute
    @DynamoDBTypeConverted(converter = LocalDateTimeConverter.class)
    private LocalDateTime lastModifiedDate;

    ...
}
```

## 참고 자료

-   [Spring Boot에서 Repository로 DynamoDB 조작하기 - 우아한 기술 블로그](https://techblog.woowahan.com/2633/)