# Organization API

## API 목록

| 번호 | Method | URL | 권한 | 설명               |
| --- | --- | --- | --- |------------------|
| 1 | POST | `/api/core/admin/organizations` | Admin | 조직 생성            |
| 2 | GET | `/api/core/admin/organizations` | Admin | 조직 목록 조회         |
| 3 | GET | `/api/core/admin/organizations/{organizationId}` | Admin | 조직 상세 조회         |
| 4 | DELETE | `/api/core/admin/organizations/{organizationId}` | Admin | 조직 삭제 처리         |
| 5 | GET | `/api/core/organizations/me` | User | 내 조직 정보 조회       |
| 6 | PUT | `/api/core/organizations/me/setup` | Owner | 조직 최초 설정 (Owner) |
| 7 | PUT | `/api/core/organizations/me/status` | Owner | 조직 활성화/비활성화      |
| 8 | PUT | `/api/core/organizations/me` | Owner | 조직 정보 수정         |


---
# 1. [조직 생성] - Admin

## 기본 정보

| 항목     | 내용                                                                                                       |
| ------ |----------------------------------------------------------------------------------------------------------|
| URL    | `/api/core/admin/organizations`                                                                          |
| Method | `POST`                                                                                                   |
| 설명     | 시스템 관리자(Admin)가 새로운 조직을 생성합니다. <br/>조직 생성 후 OWNER 초대를 위한 Invitation이 생성됩니다.<br/>Owner의 이메일로 초대링크를 전송합니다. |


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
|----------------| ------ | ----- | --------- |
| businessNumber | String | O     | 사업자 번호    |
| email          | String | O     | Owner 이메일 |
| name           | String | O     | 조직명       |


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

| 이름     | 타입      | 필수 여부 | 설명                     |
| ------ | ------- | ----- |------------------------|
| status | OrganizationStatus  | X     | 조직 상태 검색 조건            |
| name   | String  | X     | 조직명 검색 조건              |
| page   | Integer | X     | 페이지 번호 (기본값 0)         |
| size   | Integer | X     | 페이지 크기 (기본값 10)        |


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

# 5. [내 조직 정보 조회] - User

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/core/organizations/me` |
| Method | `GET` |
| 설명 | 현재 로그인한 사용자가 속한 조직 정보를 조회합니다. |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| - | - | 없음 |


### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |


### Request Body

없음


---

## Response

HTTP Status

200 OK


Response DTO: `OrgDetailResponse`

예시

```json
{
    "id": 1,
    "name": "조직이름",
    "roadAddress": "서울특별시 강남구",
    "zipCode": "61682",
    "addressDetail": "101동 202호",
    "description": "조직 설명",
    "status": "ACTIVE",
    "createdAt": "2026-08-03T10:00:00"
}
```


---

# 6. [조직 최초 설정] - Owner

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/core/organizations/me/setup` |
| Method | `PUT` |
| 설명 | Admin이 생성한 PENDING 상태의 조직에 대해 Owner가 최초 조직 정보를 입력하고 조직을 활성화합니다. |


---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| - | - | 없음 |


### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |


### Request Body

DTO: `OrganizationSetupRequest`

예시

```json
{
    "zipCode": "06234",
    "roadAddress": "서울특별시 강남구",
    "addressDetail": "101동 202호",
    "description": "조직 설명"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- |-------| --- |
| zipCode | String | O     | 우편번호 |
| roadAddress | String | O     | 도로명 주소 |
| addressDetail | String | x     | 상세 주소 |
| description | String | X     | 조직 상세 설명 |

---

## Response

HTTP Status

204 No Content

---

# 7. [조직 상태 변경] - Owner

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/core/organizations/me/status` |
| Method | `PUT` |
| 설명 | Owner가 조직 상태를 변경합니다. |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| - | - | 없음 |


### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |


### Request Body

DTO: `OrgStatusUpdateRequest`

예시

```json
{
  "status": "INACTIVE"
}
```


| 필드 | 타입 | 필수 여부 | 설명                             |
| --- | --- | --- |--------------------------------|
| status | OrganizationStatus | O | 변경할 조직 상태 (ACTIVE or INACTIVE) |


---

## Response

HTTP Status

204 No Content

---

# 8. [조직 정보 수정] - Owner

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/core/organizations/me` |
| Method | `PUT` |
| 설명 | Owner가 조직 주소 및 상세 설명 정보를 수정합니다. |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| - | - | 없음 |


### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |


### Request Body

DTO: `OrgUpdateRequest`

예시

```json
{
    "zipCode": "06234",
    "roadAddress": "서울특별시 강남구",
    "addressDetail": "101동 202호",
    "description": "수정된 조직 설명"
}
```


| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- |-------| --- |
| zipCode | String | O     | 우편번호 |
| roadAddress | String | O     | 도로명 주소 |
| addressDetail | String | x     | 상세 주소 |
| description | String | X     | 조직 상세 설명 |

---

## Response

HTTP Status

204 No Content
