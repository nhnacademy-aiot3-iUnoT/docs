# 1. [조직 생성] - Admin

## 기본 정보

| 항목     | 내용                                                         |
| ------ |------------------------------------------------------------|
| URL    | `/api/core/admin/organizations`                            |
| Method | `POST`                                                     |
| 설명     | 시스템 관리자(Admin)가 새로운 조직을 생성합니다. 조직 생성 후 Owner 초대 정보가 생성됩니다. |


---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| -- | -- | -- |
| -  | -  | 없음 |


### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| -- | -- | ----- | -- |
| -  | -  | -     | 없음 |


### Request Body

DTO: `OrgCreateRequest`

```json
{
  "businessNumber": "1234567890",
  "email": "owner@example.com",
  "name": "조직이름"
}
```

| 필드             | 타입     | 필수 여부 | 설명        |
| -------------- | ------ | ----- | --------- |
| businessNumber | String | O     | 사업자 번호    |
| name           | String | O     | 조직명       |
| ownerEmail     | String | O     | Owner 이메일 |


---

## Response

HTTP Status

```
201 Created
```


Response DTO: `OrgCreateResponse`

```json
{
  "id": 1,
  "name": "조직이름"
}
```

---

# 2. [조직 목록 조회] - Admin

## 기본 정보

| 항목     | 내용                                     |
| ------ | -------------------------------------- |
| URL    | `/api/core/admin/organizations`        |
| Method | `GET`                                  |
| 설명     | 조직 목록을 조회합니다. 조직 상태 및 이름으로 검색할 수 있습니다. |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| -- | -- | -- |
| -  | -  | 없음 |


### Query Parameter

DTO : `OrgSearchRequest` 

| 이름     | 타입      | 필수 여부 | 설명              |
| ------ | ------- | ----- | --------------- |
| status | String  | X     | 조직 상태 검색 조건     |
| name   | String  | X     | 조직명 검색 조건       |
| page   | Integer | X     | 페이지 번호 (기본값 0)  |
| size   | Integer | X     | 페이지 크기 (기본값 20) |
| sort   | String  | X     | 정렬 조건           |


### Request Body

없음

---

## Response

HTTP Status

```
200 OK
```


Response DTO: `PageResponse<OrgSearchResponse>`

```json
{
  "content": [
    {
      "organizationId": 1,
      "name": "조직이름",
      "businessNumber": "1234567890",
      "status": "ACTIVE",
      "createdAt": "2026-08-03T10:00:00"
    }
  ],
  "page": 0,
  "size": 15,
  "totalElements": 1,
  "totalPages": 1,
  "last" : true
}
```

# 3. [조직 상세 조회] - Admin


## 기본 정보
| 항목     | 내용                                               |
| ------ | ------------------------------------------------ |
| URL    | `/api/core/admin/organizations/{organizationId}` |
| Method | `GET`                                            |
| 설명     | 특정 조직의 상세 정보를 조회합니다.                             |

---

## Request

### Path Variable

| 이름             | 타입   | 설명        |
| -------------- | ---- | --------- |
| organizationId | Long | 조회할 조직 ID |



### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| -- | -- | ----- | -- |
| -  | -  | -     | 없음 |


### Request Body

없음

---

## Response

HTTP Status

```
200 OK
```


Response DTO: `AdminOrgDetailResponse>`

```json
{
  "organizationId": 1,
  "name": "조직이름",
  "businessNumber": "1234567890",
  "status": "ACTIVE",
  "ownerEmail": "owner@example.com",
  "createdAt": "2026-08-03T10:00:00"
}
```
# 4. [조직 삭제 처리] - Admin


## 기본 정보
| 항목     | 내용                                               |
| ------ | ------------------------------------------------ |
| URL    | `/api/core/admin/organizations/{organizationId}` |
| Method | `DELETE`                                         |
| 설명     | 조직을 삭제 처리합니다.                                    |

---

## Request

### Path Variable

| 이름             | 타입   | 설명        |
| -------------- | ---- |-----------|
| organizationId | Long | 삭제할 조직 ID |



### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| -- | -- | ----- | -- |
| -  | -  | -     | 없음 |


### Request Body

없음

---

## Response

HTTP Status

```
204 No Content

```
