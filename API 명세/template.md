# API 명세 작성 가이드

## 작성 필수 항목
- URL
- HTTP Method
- Request DTO
- Response DTO

※ 상세 비즈니스 로직 및 예외 처리는 필요 시 추가 작성


---

# RESTful API 작성 규칙

## 1. URL 작성 규칙

### 기본 원칙
- URL에는 행위(동사)가 아닌 리소스(명사)를 사용한다.
- 리소스는 복수형 사용을 권장한다.
- URL은 kebab-case(`-`) 사용을 권장한다.
- 계층 관계는 `/` 로 표현한다.

### 권장 예시

회원 목록 조회

```
GET /members
```

회원 상세 조회

```
GET /members/{member-id}
```

조직의 저장소 조회

```
GET /organizations/{organization-id}/storages
```

재고 등록

```
POST /inventories
```


### 지양하는 예시

```
GET /getMember
POST /createMember
POST /deleteInventory
```

이유:
- HTTP Method가 행위를 표현하기 때문에 URL에 동사를 사용하지 않는다.


---

# 2. HTTP Method 사용 기준

| Method | 사용 목적 | 예시 |
|---|---|---|
| GET | 조회 | 회원 조회 |
| POST | 생성 | 회원 가입, 재고 등록 |
| PUT | 수정 | 회원 정보 수정 |
| DELETE | 삭제 | 데이터 삭제 |

- PATCH 사용하지 않기로 함 (PUT으로 통일)
---

# 3. Path Variable / Query Parameter 사용 기준

## Path Variable

특정 리소스를 식별할 때 사용

예:

```
GET /members/{member-id}
```

특정 회원 조회


## Query Parameter

조회 조건, 검색, 필터링, 정렬 등에 사용

예:

```
GET /medicines?category=antibiotic
```


---

# API 명세 템플릿


# [API 이름]


## 기본 정보

| 항목 | 내용 |
|---|---|
| URL | |
| Method | |
| 설명 | |


---

## Request


### Path Variable

| 이름 | 타입 | 설명 |
|---|---|---|
| | | |


### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| | | |


### Request Body

DTO:

```json
{
}
```


---

## Response

HTTP Status

```
200 OK
```


Response DTO:

```json
{
}
```


---

# 작성 예시


# 회원 상세 조회


## 기본 정보

| 항목 | 내용 |
|---|---|
| URL | `/members/{member-id}` |
| Method | GET |
| 설명 | 회원 상세 조회 |


---

## Request


### Path Variable

| 이름 | 타입 | 설명 |
|---|---|---|
| member-id | Long | 회원 ID |


---

## Response

HTTP Status

```
200 OK
```


Response DTO:

```json
{
  "memberId": 1,
  "name": "홍길동",
  "email": "test@test.com"
}
```
