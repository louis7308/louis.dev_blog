---
title: DTO, VO, DAO란 무엇인가?
description: 막연하게 코드를 사용할 때 DTO만 사용하고 있었는데 VO, DAO라는 용어를 듣게 되어 정리해보는 글입니다.
date: 2023-11-27
update: 2023-11-27
tags:
  - "#spring"
  - dto
  - vo
  - dao
series: Spring
---
제가 개발을 하다 보니 DTO라는 명칭은 자주 사용하지만 VO, DAO 라는 명칭은 자주 사용하지 않아 공부 하다보니 용도에 대해 알고 싶어서 글을 작성해보려고 합니다.

## 혼용 사례
이러한 DTO와 VO가 왜 나왔을지에 대해 조금 찾아보니 이는 사실 J2EE Patterns 1판 초판에서는 데이터 전달용 객체를 VO로 정의하였고 2판에서는 데이터 전달용 객체를 TO로 정의 하여 발생한 문제였습니다.

현재는 두개를 각각 다른 용도로 사용하고 있습니다.
VO : 값 표현용
DTO : 데이터 전달용

이제 더 자세히 알아가봅시다!

## DTO란?
![](https://i.imgur.com/QKZEqmE.png)

Data Transfer Object로서 데이터를 담아서 전달하기 위해 사용하는 객체입니다.
주로 사용되는 예는 **계층간 데이터를 전달**하기 위할 때 사용됩니다.

DTO는 Getter/Setter를 가지고 있을 수 있습니다.
여기에서 Setter를 가지고 있지않은 DTO는 불변 DTO라고 부를 수 있고,
Setter를 가지고 있는 DTO는 가변 DTO라고 부를 수 있습니다.

!또한 DTO는 속성값이 모두 같다고 해서 같은 객체가 아닙니다 이를 주의하시기 바랍니다.

### 그렇다면 왜 Dto를 사용해야 할까요?
저희가 만약을 한번 가장 해보겠습니다.
![](https://i.imgur.com/1C60hg0.png)
이러한 요청에 DTO가 아닌 Domain(Entity)를 요청과 응답으로 처리한다고 생각해봅시다.
저희는 프로젝트를 진행하는 과정중 View단에 비즈니스 로직이 자주 바뀌는 것을 경험해보신적이 있으실겁니다.

View에서 요청을 받는 field값에 원래는 "이름": "값" 으로 받았지만, 요청 값에 성명, 영어이름이 추가된다면 저희는 요청 값을 변경해야 하는 상황이 발생합니다.

하지만 저희가 요청을 받는 부분은 Entity이기 때문에 이 부분에 값을 변경한다면 Entity구조 또한 같이 변경된다는 문제점이 발생하는 것이죠.

그러하여 저희는 꼭 View에 변경이 바뀔 때 마다 다른 클래스에 영향을 주지 않고 사용할 수 있는 DTO를 만들어 사용해야한다고 생각합니다!

## VO란?
Value Object
값 그 자체를 표현하는 객체를 뜻한다.
흔한 예시로 돈으로 비유를 해보겠습니다.
만원 짜리 지폐에 고유번호가 존재합니다.
하지만 저희는 이것을 다른 만원이야 라고 하면서 편의점에서 물건을 구매 못한 상황을 겪어 보신적은 없으시죠?
이렇듯 만원이라는 값 그 자체를 표현하기 때문에 만원이라는 값은 동등하므로 같은 만원이라고 표현하는 것입니다.

코드 관점으로 보자면 Dto와 달리 VO는 getter 외에 로직을 가질 수 있다. 하지만 setter는 가질 수 없다 그러하여 새로 객체를 만들어 사용하는 경우가 많다.

### 동일 vs 동등
제가 여기에서 동등이라는 표현을 조금 사용하였는데
동일이라는 표현도 있지만 동등이라는 표현을 사용한 이유는
동일은 객체의 참조 주소값이 동일 할 때 사용하는 것이고,
동등은 객체의 값이 같을 때 사용하기 때문에 동등이라는 표현을 사용하였습니다.

## DAO란
데이터 사용기능 담당 클래스입니다.
DB데이터 조회나, 수정, 입력, 삭제와 같은 로직을 처리하기 위해 사용됩니다.
DAOInterface/DAOImplement로 구분지어 구현 분리하며 개발합니다.
DB의 **Data에 접근하는 객체**이면서 **비즈니스 로직과, DB Access 로직을 분리**하기 위해 사용됩니다.
만약 Mybatis연동 때처럼 Interface만 필요한 경우 그냥 DAO 라고 명시할 수 있습니다.

-> Mybatis 예시로 코드를 가져와보겠습니다.
```kotlin
@MyMapper
interface UserMapper {
	fun selectUserById(userId: String): UserVO

	fun insertUser(userVO: UserVO)

	...
}

class UserDaoImplMapper(
	private val userMapper: UserMapper
) : UserDao {
	@Override
	fun insert(user: UserVO) {
		userMapper.insertUser(user)
	}

	...
}
```
위 예제에서 UserMapper는 DAO의 인터페이스 라고 보기는 조금 애매합니다.
하지만 UserDaoImplMapper는 DAO 인터페이스 메소드를 구현하고 데이터 액세스 작업을 하는 DAO가 맞습니다.

-> ORM 예시로 코드를 가져와 보겠습니다.
![](https://i.imgur.com/IlYx8gH.png)
```kotlin
interface CustomerRepository : CrudRepository<Customer, Long> {}

CrudRepository -> SimpleJpaRepository(구현체)
```
현재 CustomerRepository가 CrudRepository를 인터페이스를 확장 받는데 이 인터페이스가 구현 되어진 SimpleJpaRepository에서 DAO 코드가 구현 되어져 있기 때문에 우리는 ORM을 사용한다면 알아서 DAO가 적용된 CrudRepository만 상속 받아 사용하면 되어서 편리하다.

### 정리
DTO는 유연한 데이터 전송을 표현하고, 
VO는 추가 논리로 불변 값을 표현하며, 
DAO는 데이터 액세스 논리를 분리하여 소프트웨어 개발에서 모듈식이며 유지 관리 가능한 아키텍처를 촉진합니다.

