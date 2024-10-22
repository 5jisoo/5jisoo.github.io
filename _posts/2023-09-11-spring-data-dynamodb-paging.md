---
title: Spring Data DynamoDB 페이징 정리
date: 2023-09-11 18:20:00 +/-TTTT
categories: [Spring, Spring Data DynamoDB]
tags: [spring, dynamodb]     # TAG names should always be lowercase
---

## DDB에서 페이징이 필요한 이유

[공식 문서](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Query.html)에도 언급되어 있지만, DynamoDB의 단일 쿼리 작업은 최대 1MB의 데이터만 가져올 수 있습니다. 그렇기에 우리 어플리케이션에서 프로필 탭에서 자신이 찍힌 게시글과 찍은 게시글을 조회할 때 다음과 같은 페이징이 필요하다고 판단하여 리팩토링을 진행하게 되었습니다!

-   물론 성능 측면도 고려해보았을 때, 페이징은 언젠가 해야되리라 생각하고 있었음 ..

## 페이징 하는 방법 - CLI

다음과 같이 page-size를 지정하고 쿼리를 보내면?

```
aws dynamodb query --table-name Movies \
    --projection-expression "title" \
    --key-condition-expression "#y = :yyyy" \
    --expression-attribute-names '{"#y":"year"}' \
    --expression-attribute-values '{":yyyy":{"N":"1993"}}' \
    --page-size 5 \
    --debug
```

만약 5개 결과가 결과 그룹의 전부가 아니면 다음과 같이 **LastEvaluatedKey**가 반환됩니다.

-   LastEvaluatedKey는 다음 Query 요청에 대한 ExclusiveStartKey로 사용함으로써 다음 그룹을 가져오는 데 사용합니다!
-   만약 다음 결과값이 없으면 LastEvaluatedKey에 **null 값**이 반환됩니다.

```
2017-07-07 11:13:15,603 - MainThread - botocore.parsers - DEBUG - Response body:
b'{"Count":5,"Items":[{"title":{"S":"A Bronx Tale"}},
{"title":{"S":"A Perfect World"}},{"title":{"S":"Addams Family Values"}},
{"title":{"S":"Alive"}},{"title":{"S":"Benny & Joon"}}],
"LastEvaluatedKey":{"year":{"N":"1993"},"title":{"S":"Benny & Joon"}},
"ScannedCount":5}'
```

## 페이징하기 - Java

```java
DynamoDBQueryExpression<Publish> query = new DynamoDBQueryExpression<Publish>()
        .withKeyConditionExpression("#PK = :val1 and begins_with(#SK, :val2)")
        .withFilterExpression("#STATUS = :val3")
        .withExpressionAttributeValues(eav)
        .withExpressionAttributeNames(ean)
        .withLimit(12) // paging size
        .withExclusiveStartKey(exclusiveStartKey) // paging 시작 키 설정
        .withScanIndexForward(false); // desc
```

-   limit : 12개로 한정
-   exclusiveStartKey : 시작키 설정. null로 설정할 경우, 처음부터 가져옴.

exclusiveStartKey는 다음과 같이 Map 형태로 이루어져 있음.

```java
Map<String, AttributeValue> exclusiveStartKey
```

## 참고 자료

-   [테이블 쿼리 결과 페이지 매김](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Query.Pagination.html)