---
title: JPA 영속화 순서 변경 - Dirty Checking과 @DynamicUpdate 사용
date: 2024-06-24 18:20:00 +/-TTTT
categories: [Study, Spring]
tags: [jpa, dirty-checking, dynamic-update, spring, springboot]     # TAG names should always be lowercase
---

> Dirty Checking과 관련된 내용을 복습하며 영속화 순서에 대해 궁금했던 것들, 그리고 실험했던 기록을 작성합니다.

## 강의 내용

Parent Entity와 Child Entity가 1:N으로 연관관계가 맺어져 있음. (따라서 연관관계의 주인은 Child Entity.)

이 상태에서 아래 코드를 실행시킬 경우, 

```java
Child child1 = new Child();
child1.setName("child1");

Child child2 = new Child();
child2.setName("child1");

Parent parent = new Parent();
parent.setName("Parent1");
parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);
em.persist(child1);
em.persist(child2);
```

다음 사진과 같이 총 3번의 INSERT 쿼리가 나가게 됩니다.

|-----|-----|
|1.  Parent Entity INSERT &emsp; &emsp; &emsp; <br> 2.  (child1) Child Entity INSERT <br> 3.  (child2) Child Entity INSERT |![img](/assets/img/2024-06-24-dirty-checking-and-dynamic-update/0.png){: w="300" }|



---

## 1) 만약 Child와 Parent의 영속화 순서를 뒤집으면 어떻게 될까?

### 가정

연관관계의 주인(FK를 가지고 있는 쪽 - 여기서는 Child)을 가장 먼저 영속화 시켜주는 코드를 작성해보자

-   Parent를 영속화하기 전, Child를 먼저 영속화하였고, 이후에 Parent를 영속성 컨텍스트에 등록함.

```java
Child child1 = new Child();
child1.setName("child1");

Child child2 = new Child();
child2.setName("child1");

Parent parent = new Parent();
parent.setName("Parent1");
parent.addChild(child1);
parent.addChild(child2);

em.persist(child1); // child를 먼저 영속화
em.persist(child2);
em.persist(parent);

System.out.println("=========");
```

### 결과

이 경우, 쿼리는 다음과 같이 나가게 된다.

|----|----|
|1.  INSERT CHILD1 &emsp; &emsp; &emsp; <br> 2.  INSERT CHILD2 <br> 3.  INSERT PARENT <br> 4.  UPDATE CHILD1 <br> 5.  UPDATE CHILD2 |![img](/assets/img/2024-06-24-dirty-checking-and-dynamic-update/1.png){: w="350"}|

그렇다면 여기서 왜 **CHILD의 컬럼을 전부 업데이트하는 쿼리**가 등장하였을까?

### 이유 \[ Dirty Checking \]

#### "영속성 컨텍스트와 변경 감지" 복습

> 영속성 컨텍스트엔 1차 캐시가 있고, 이 안엔 ID, Entity, 스냅샷이 저장되어 있음.  
> 이 때, **스냅샷은 DB에서 값을 읽어왔을 때 (영속성 컨텍스트에 들어왔을 때)**의 상태를 저장해둔 것

persist() 할 때 (영속성 컨텍스트에 등록할 때) Child에 연관된 Parent는 영속성 컨텍스트에도 등록이 되어 있지 않고, DB에도 등록되어 있지 않음.

따라서 Child 엔티티가 영속성 컨텍스트에 등록될 시점엔 **PARENT\_ID (FK)가 NULL로 설정**된 버전이 스냅샷으로 저장되는 것!

이후에 Parent를 영속성 컨텍스트에 등록한 후에야 Child의 FK를 설정할 수 있기에,  
트랜잭션의 커밋 시점에 Dirty Checking을 통해 UPDATE 쿼리가 생성되면 이 때 PARENT\_ID, 즉 FK가 UPDATE 되는 것.

-   Dirty Checking 과정을 통해 **FK가 세팅된 엔티티와 아직 FK가 세팅되지 않은 스냅샷을 비교**하면서 UPDATE 쿼리를 날려준 것.

---

## 2) 하나의 엔티티에 두 번의 수정사항이 생긴다면, UPDATE는 두 번 나갈까?

### 가정

Child를 영속성 컨텍스트에 등록시킨 이후에 FK도 바꾸고, 이름도 바꿔준다면 UPDATE 쿼리는 두 번 나갈까?

1.  Parent를 Child보다 이후에 등록 ⇒ FK UPDATE
2.  child1의 이름을 변경

```java
Child child1 = new Child();
child1.setName("child1");

Child child2 = new Child();
child2.setName("child1");

Parent parent = new Parent();
parent.setName("Parent1");
parent.addChild(child1);
parent.addChild(child2);

em.persist(child1);
em.persist(child2);
em.persist(parent);

child1.setName("childA");

System.out.println("=========");
```

### 결과

Child1에 대한 **UPDATE쿼리는 한 번만 나가게 됨**!

여기서 UPDATE 쿼리가 CHILD1과 CHILD2 모두 name과 PARENT\_ID를 모두 업데이트 시키고 있다는 것을 확인해야 함.

-   CHILD1은 name, PARENT\_ID 모두 UPDATE하고 있지만,
-   CHILD2는 PARENT\_ID만 바뀐 상황임에도 name까지 모두 업데이트 되고 있음.

즉, 한 엔티티의 레코드 중 무엇이 변경되었는지를 일일히 확인하고 있는 것이 아닌, **엔티티의 모든 레코드를 업데이트 하고 있다는 점**!

![img](/assets/img/2024-06-24-dirty-checking-and-dynamic-update/2.png){: w="350"}

### 이유 \[@DynamicUpdate\]

하이버네이트는 엔티티의 변경 사항을 감지할 때, 변경된 필드만을 대상으로 하는 '부분 업데이트'를 실행하지 않고, 엔티티의 모든 필드를 포함하는 '전체 업데이트'를 실행함.

@DynamicUpdate 애노테이션을 사용하면 하이버네이트가 변경된 필드에 대해서만 업데이트 SQL을 생성하도록 할 수 있으나, 신중하게 사용해야 하며 모든 상황에서 성능 개선을 보장하지는 않음.

> [출처: 인프런 질문](https://www.inflearn.com/questions/1194412/%EB%A9%A4%EB%B2%84%EC%99%80-%ED%8C%80%EC%9D%98-%EC%98%81%EC%86%8D%ED%99%94-%EC%88%9C%EC%84%9C%EB%A5%BC-%EB%92%A4%EC%A7%91%EC%97%88%EC%9D%84-%EB%95%8C-%EC%83%9D%EC%84%B1%EB%90%98%EB%8A%94-query%EC%97%90-%EB%8C%80%ED%95%9C-%EC%A7%88%EB%AC%B8%EC%9E%85%EB%8B%88%EB%8B%A4)

---

## 3) @DynamicUpdate를 사용하면 UPDATE 쿼리는 어떻게 변할까?

### 가정

다음과 같이 Child에 @DynamicUpdate를 추가한 뒤, 2번의 수정사항이 발생하면 어떤 UPDATE 쿼리가 나가게 될까?

```java
@Entity
@DynamicUpdate // 애노테이션 추가
public class Child {

    @Id
    @GeneratedValue
    private Long id;
    
    
    ...
    
}
```

### 결과

![img](/assets/img/2024-06-24-dirty-checking-and-dynamic-update/3.png){: w="350"}

CHILD1

-   name과 PARENT\_ID가 전부 변경되었으므로, name과 PARENT\_ID가 전부 UPDATE 되는 쿼리가 생성됨.

```sql
Hibernate: 
    /* update
        for hellojpa.Child */update Child 
    set
        name=?,
        PARENT_ID=? 
    where
        id=?
```

CHILD2

-   FK(PARENT\_ID)만 변경되었으므로, PARENT\_ID만 UPDATE 되는 쿼리가 생성됨.

```sql
Hibernate: 
    /* update
        for hellojpa.Child */update Child 
    set
        PARENT_ID=? 
    where
        id=?
```

---

## @DynamicUpdate를 신중하게 사용해야 하는 이유

### PreparedStatement의 특징

미리 컴파일된 SQL 문을 포함하고 있어 PreparedStatment가 실행될 때 **DBMS가 먼저 컴파일할 필요 없이 바로 실행**할 수 있도록 함.

-   여기서 **?**는 파라미터를 나타내는데, 동적으로 값을 넣을 수 있음.

ex)

```java
String updateString =
  "update COFFEES set SALES = ? where COF_NAME = ?";


PreparedStatement updateSales = con.prepareStatement(updateString);

updateSales.setInt(1, 80);
updateSales.setString(2, "MAXIM");
updateSales.executeUpdate();
```

### 항상 성능에 유리하지는 않다!

JPA는 위와 같이 애플리케이션 로딩 시점에 PreparedStatement 스타일로 해당 엔티티의 UPDATE 쿼리를 만듦.

따라서 성능만 생각하면

1.  해당 PreparedStatement의 컬럼에 값을 전부 넘기는 것과
2.  @DynamicInsert를 사용하여 하나만 UPDATE를 하는 것

위 두가지가 큰 차이가 없음.

오히려 2번 보다 1번처럼 **전체 컬럼을 PreparedStatement 스타일의 SQL을 반복해서 사용**하는 게 더 속도가 빠를 수도 있는 것!

(물론 컬럼이 너무 많거나, 길이가 너무 길거나, 데이터가 크다면 상황은 달라짐.)

> 출처  
> [\[Oracle docs\] JDBC Basics - Using Prepared Statement](https://docs.oracle.com/javase%2Ftutorial%2F/jdbc/basics/prepared.html)  
> [인프런 질문 - "업데이트 고견 구합니다."](https://www.inflearn.com/questions/16697/%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8-%EA%B3%A0%EA%B2%AC-%EA%B5%AC%ED%95%A9%EB%8B%88%EB%8B%A4)