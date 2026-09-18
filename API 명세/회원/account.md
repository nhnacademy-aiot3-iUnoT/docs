# 회원·인증 API 명세

> 2026-09-17 Account `develop`의 `b39666c757e8ca8bc12a8102e1bac97b5845a109` 소스 기준이다. Front·Gateway와의 구분 및 다른 모듈의 기준 커밋은 [회원 전체 흐름](../../기능%20명세/1.회원/0.회원%20전체%20흐름.md)에 기록했다. 테스트·실행 검증은 수행하지 않았다.

## API 목록

| 번호 | Method | URL | 권한 | 설명 |
| --- | --- | --- | --- | --- |
| 1 | POST | `/api/auth/login` | 공개 | 로그인 |
| 2 | POST | `/api/auth/refresh` | 공개 | 토큰 갱신 |
| 3 | POST | `/api/auth/logout` | 공개 | 로그아웃 |
| 4 | POST | `/api/accounts` | 공개 | 초대 기반 회원가입 |
| 5 | POST | `/api/accounts/check-email` | 공개 | 이메일 사용 가능 여부 |
| 6 | GET | `/api/accounts/me` | User | 내 계정 조회 |
| 7 | PUT | `/api/accounts/me` | User | 내 이름 변경 |
| 8 | PUT | `/api/accounts/me/pwd` | User | 내 비밀번호 변경 |
| 9 | POST | `/api/accounts/check-pwd` | User | 새 비밀번호 재사용 확인 |
| 10 | DELETE | `/api/accounts/me` | User | 내 계정 탈퇴 |
| 11 | POST | `/api/accounts/pwd` | 공개 | 비밀번호 재설정 메일 요청 |
| 12 | POST | `/api/accounts/pwd/reset/{token}` | 공개 | 링크 토큰으로 비밀번호 재설정 |
| 13 | POST | `/api/accounts/me/reactivation/verification` | User | 재활성화 메일 요청 |
| 14 | POST | `/api/accounts/me/reactivation/confirm` | User | 재활성화 토큰 확인 |
| 15 | GET | `/api/accounts/admin` | Admin | 관리자 회원 목록 |
| 16 | GET | `/api/accounts/admin/{uuid}` | Admin | 관리자 회원 상세 |
| 17 | POST | `/api/accounts/admin` | Admin | 관리자 계정 생성 |
| 18 | PUT | `/api/accounts/admin/{uuid}` | Admin | 관리자 회원 이름 수정 |
| 19 | PUT | `/api/accounts/admin/{uuid}/pwd` | Admin | 관리자 비밀번호 재설정 |
| 20 | PUT | `/api/accounts/admin/{uuid}/status` | Admin | 관리자 회원 상태 변경 |
| 21 | DELETE | `/api/accounts/admin/{uuid}` | Admin | 관리자 회원 탈퇴 |
| 22 | GET | `/api/auth/.well-known/jwks.json` | 공개 | 공개키 목록 |
| 23 | GET | `/api/accounts/internal/search` | 내부 서비스 | 내부 이메일 검색 |
| 24 | GET | `/api/accounts/internal` | 내부 서비스 | 내부 UUID 목록 조회 |
| 25 | DELETE | `/api/accounts/internal` | 내부 서비스 | 내부 단건 탈퇴 |
| 26 | POST | `/api/accounts/internal/bulk-delete` | 내부 서비스 | 내부 일괄 탈퇴 |

## 공통 규칙

- 요청·응답 JSON은 `Content-Type: application/json`을 사용한다. body가 없는 응답은 별도 표시한다.
- 보호 API는 `Authorization: Bearer {accessToken}`이 필요하다. Front의 `access_token`·`refresh_token` 쿠키를 Account 요청 DTO에 임의로 추가하지 않는다.
- 공개 API는 로그인·갱신·로그아웃, 가입, 이메일 확인, 비밀번호 복구, JWKS다. 관리자 경로는 ADMIN 역할과 DB의 활성 관리자 여부를 확인한다.
- `/api/accounts/internal/**`는 Account 설정상 인증을 요구하지 않는 내부 계약이다. Gateway의 공개 경로 목록과는 다르며 일반 사용자 API로 분류하지 않는다.
- Gateway에는 `/api/accounts/check-pwd`가 공개 경로로 등록되어 있지만 Account에서는 Bearer 인증이 필요하다. 최종 서비스의 인증 요구를 따른다.
- JWT 검증은 서명 알고리즘·만료·발급자·audience·UUID Subject·kid를 확인한다. Access Token 인증 진입점은 누락·유효하지 않은 토큰을 `401 AU002`로 응답한다.
- 이메일은 서비스에서 `trim()`과 소문자 변환을 적용한다. DTO의 이메일 형식 검증은 서비스 진입 전에 수행된다.
- JSON 예시의 계정·토큰 값은 설명용 가상 값이다.

## 공통 Response

`ApiResponse<T>`의 필드는 `success: boolean`, `data: T`, `error: ErrorDetail 또는 null`, `timestamp: LocalDateTime`이다. `ApiResponse<Void>`의 성공 `data`는 null이다.

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "G001",
    "message": "입력값이 올바르지 않습니다.",
    "fieldErrors": []
  },
  "timestamp": "2026-09-17T12:00:00"
}
```

현재 Account 검증 처리기는 필드 오류를 수집하지만 응답에는 전달하지 않아 `fieldErrors`는 빈 배열이다. 내부 계정 조회·삭제와 JWKS는 성공 응답에 `ApiResponse`를 사용하지 않는다.

### 응답 DTO

| DTO | 필드·타입 |
| --- | --- |
| `AccountResponse` | `uuid: UUID`, `name: String`, `email: String`, `accountRole: USER/ADMIN`, `accountStatus: ACTIVE/LOCKED/INACTIVE/WITHDRAWN`, `createdAt: LocalDateTime`, `updatedAt: LocalDateTime`, `withdrawnAt: LocalDateTime 또는 null` |
| `LoginResponse` | `accessToken: String`, `refreshToken: String` |
| `EmailAvailabilityResponse` | `available: boolean` |
| `PasswordReuseCheckResponse` | `available: boolean` |
| `InternalAccountInfoResponse` | `accountUuid: UUID`, `name: String`, `email: String` |

`AccountResponse`는 내부 ID·비밀번호 해시를 제외한다. `WITHDRAWN`이면 응답의 `name`, `email`을 null로 가린다. 내부 응답 DTO는 별도 변환이므로 외부 응답의 개인정보 가림 규칙을 그대로 적용한다고 가정하지 않는다.

## 공통 오류 코드

| HTTP | 코드 | 의미·사용 범위 |
| --- | --- | --- |
| 400 | `G001` | DTO 검증 실패·읽을 수 없는 JSON 등 입력 오류 |
| 403 | `G002` | 일반 권한 없음으로 정의 |
| 500 | `G003` | 처리되지 않은 내부 예외 |
| 502 | `G004` | Inventory 연동 실패. upstream 메시지를 전달할 수 있으나 코드는 Account 기준으로 변환 |
| 404 | `A001` | 존재하지 않는 회원 |
| 409 | `A002` | 사용 중인 이메일 |
| 403 | `A003` | 비활성 회원으로 정의되어 있으나 현재 로그인은 INACTIVE에 토큰을 발급 |
| 403 | `A004` | 잠긴 계정의 토큰 발급 거부 |
| 403 | `A005` | 탈퇴 계정의 토큰 발급 거부 |
| 409 | `A006` | 허용되지 않은 계정 상태·전이 |
| 400 | `A007` | 현재와 동일한 새 비밀번호 |
| 400 | `A008` | 현재 비밀번호 불일치 |
| 400 | `A009` | 복구 토큰 소유자 불일치·만료·소비 완료 등 |
| 401 | `AU001` | 이메일·비밀번호 불일치 |
| 401 | `AU002` | Access Token 인증 실패 |
| 401 | `AU003` | Refresh Token 유효성·소비 실패 등 |
| 401 | `AU004`, `AU005` | 만료·미지원 토큰으로 정의. 현재 Access Token 진입점은 AU002로 통합 |
| 403 | `AU006` | 관리자 자격·접근 권한 없음 |

아래 각 API의 예외 외에 공통 입력·인증·내부 오류가 발생할 수 있다. 코드 정의만으로 모든 경로가 해당 코드를 반환한다고 보장하지 않는다.

---

# 1. [로그인] - 공개

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/auth/login` |
| Method | `POST` |
| 설명 | 로그인 |
| 인증·권한 | 공개. 제출하는 토큰·입력값은 별도 검증 |

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

DTO: `LoginRequest`

```json
{
  "email": "user@example.com",
  "password": "ExamplePwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `email` | String | O | 공백 불가·이메일 형식·최대 254자 |
| `password` | String | O | 공백 불가·6~64자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<LoginResponse>`

```json
{
  "success": true,
  "data": {
    "accessToken": "example-access-token",
    "refreshToken": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  },
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

이메일·비밀번호를 확인하고 두 토큰을 발급한다. `Cache-Control: no-store`. INACTIVE도 발급 가능하다. 잘못된 자격 증명은 `401 AU001`, 잠금은 `403 A004`다. 탈퇴 계정은 이메일 제거로 AU001이 발생할 수 있으며 상태 검사에 도달하면 A005다. Access Token 기본 TTL은 30분, Refresh Token은 14일이다.

---

# 2. [토큰 갱신] - 공개

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/auth/refresh` |
| Method | `POST` |
| 설명 | 토큰 갱신 |
| 인증·권한 | 공개. 제출하는 토큰·입력값은 별도 검증 |

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

DTO: `RefreshTokenRequest`

```json
{
  "refreshToken": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `refreshToken` | String | O | 공백 불가·최대 128자, 정규식 `^[0-9a-f]{32}\.[0-9a-f]{64}$` |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<LoginResponse>`

### 처리 규칙 및 예외

Redis의 기존 토큰을 `getAndDelete()`로 소비하고 DB 계정 상태를 확인한 뒤 두 토큰을 새로 발급한다. `Cache-Control: no-store`. 형식 오류는 `G001`, 미등록·만료·재사용 토큰은 `401 AU003`, 잠금·탈퇴는 `A004`·`A005`. 기존 토큰 소비 이후 실패해도 토큰은 복구되지 않는다.

---

# 3. [로그아웃] - 공개

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/auth/logout` |
| Method | `POST` |
| 설명 | 로그아웃 |
| 인증·권한 | 공개. 제출하는 토큰·입력값은 별도 검증 |

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

DTO: `RefreshTokenRequest`

```json
{
  "refreshToken": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `refreshToken` | String | O | 공백 불가·최대 128자, 정규식 `^[0-9a-f]{32}\.[0-9a-f]{64}$` |

갱신과 동일한 `RefreshTokenRequest`를 JSON body로 전달한다.

---

## Response

HTTP Status

```text
204 No Content
```

Response DTO: 본문 없음

### 처리 규칙 및 예외

제출한 Refresh Token의 Redis 키를 삭제한다. 형식이 유효하면 이미 삭제된 토큰도 성공 처리한다. 다른 세션의 Refresh Token이나 이미 발급한 Access Token을 일괄 폐기하지 않는다.

---

# 4. [초대 기반 회원가입] - 공개

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts` |
| Method | `POST` |
| 설명 | 초대 기반 회원가입 |
| 인증·권한 | 공개. 제출하는 토큰·입력값은 별도 검증 |

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

DTO: `CreateAccountRequest`

```json
{
  "inviteToken": "00000000-0000-4000-8000-000000000001",
  "name": "홍길동",
  "email": "user@example.com",
  "password": "ExamplePwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `inviteToken` | UUID | O | 조직 초대 토큰 |
| `name` | String | O | 공백 불가·최대 100자 |
| `email` | String | O | 공백 불가·이메일 형식·최대 254자 |
| `password` | String | O | 공백 불가·6~64자 |

---

## Response

HTTP Status

```text
201 Created
```

Response DTO: `ApiResponse<Void>`

```json
{
  "success": true,
  "data": null,
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

Inventory 초대 사용 후 USER·ACTIVE 계정을 저장한다. 생성된 회원 객체를 응답하지 않는다. 이메일 중복은 `409 A002`, 초대 사용 등 Inventory 연동 실패는 `502 G004`. Account 저장 실패 시 초대 사용 보상을 시도한다.

---

# 5. [이메일 사용 가능 여부] - 공개

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/check-email` |
| Method | `POST` |
| 설명 | 이메일 사용 가능 여부 |
| 인증·권한 | 공개. 제출하는 토큰·입력값은 별도 검증 |

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

DTO: `EmailAvailabilityRequest`

```json
{
  "email": "user@example.com"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `email` | String | O | 공백 불가·이메일 형식·최대 254자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<EmailAvailabilityResponse>`

```json
{
  "success": true,
  "data": {
    "available": true
  },
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

정규화한 이메일이 없으면 true, 있으면 false다. 이 확인으로 이메일을 예약하지 않는다.

---

# 6. [내 계정 조회] - User

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/me` |
| Method | `GET` |
| 설명 | 내 계정 조회 |
| 인증·권한 | Bearer JWT, 본인 UUID는 `sub`에서 추출 |

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

DTO: 없음. Request Body를 사용하지 않는다.

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

```json
{
  "success": true,
  "data": {
    "uuid": "00000000-0000-4000-8000-000000000001",
    "name": "홍길동",
    "email": "user@example.com",
    "accountRole": "USER",
    "accountStatus": "ACTIVE",
    "createdAt": "2026-09-17T12:00:00",
    "updatedAt": "2026-09-17T12:00:00",
    "withdrawnAt": null
  },
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

JWT UUID에 해당하는 계정이 없으면 `404 A001`.

---

# 7. [내 이름 변경] - User

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/me` |
| Method | `PUT` |
| 설명 | 내 이름 변경 |
| 인증·권한 | Bearer JWT, 본인 UUID는 `sub`에서 추출 |

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

DTO: `UpdateAccountNameRequest`

```json
{
  "name": "김회원"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `name` | String | O | 공백 불가·1~100자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

### 처리 규칙 및 예외

대상이 ACTIVE여야 한다. 계정 없음 `A001`, 상태 오류 `A006`. 비밀번호는 별도 API를 사용한다.

---

# 8. [내 비밀번호 변경] - User

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/me/pwd` |
| Method | `PUT` |
| 설명 | 내 비밀번호 변경 |
| 인증·권한 | Bearer JWT, 본인 UUID는 `sub`에서 추출 |

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

DTO: `ChangeOwnPasswordRequest`

```json
{
  "currentPassword": "ExamplePwd2026!",
  "newPassword": "ChangedPwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `currentPassword` | String | O | 공백 불가·6~64자 |
| `newPassword` | String | O | 공백 불가·6~64자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

### 처리 규칙 및 예외

ACTIVE 계정만 가능하다. 현재 비밀번호 불일치 `A008`, 새 비밀번호 재사용 `A007`, 계정 없음 `A001`, 상태 오류 `A006`. 전체 Refresh Token 폐기는 없다.

---

# 9. [새 비밀번호 재사용 확인] - User

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/check-pwd` |
| Method | `POST` |
| 설명 | 새 비밀번호 재사용 확인 |
| 인증·권한 | Bearer JWT, 본인 UUID는 `sub`에서 추출 |

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

DTO: `PasswordReuseCheckRequest`

```json
{
  "newPassword": "ChangedPwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `newPassword` | String | O | 공백 불가·6~64자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<PasswordReuseCheckResponse>`

```json
{
  "success": true,
  "data": {
    "available": true
  },
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

기존 값과 다르면 true, 같으면 `400 A007`이다. false 응답이나 실제 비밀번호 변경은 없다. 계정 없음 `A001`.

---

# 10. [내 계정 탈퇴] - User

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/me` |
| Method | `DELETE` |
| 설명 | 내 계정 탈퇴 |
| 인증·권한 | Bearer JWT, 본인 UUID는 `sub`에서 추출 |

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

DTO: `WithdrawAccountRequest`

```json
{
  "password": "ExamplePwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `password` | String | O | 본인 현재 비밀번호, 공백 불가·6~64자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<Void>`

```json
{
  "success": true,
  "data": null,
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

비밀번호 확인 후 Inventory 조직 탈퇴를 먼저 호출한다. 성공하면 상태를 WITHDRAWN으로 바꾸고 이메일·해시를 제거한다. 이름은 DB에 유지된다. 비밀번호 불일치 `A008`, 계정 없음 `A001`, Inventory 거부·연결 실패 `G004`. Front `DELETE /withdraw`는 별도로 204를 응답한다.

---

# 11. [비밀번호 재설정 메일 요청] - 공개

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/pwd` |
| Method | `POST` |
| 설명 | 비밀번호 재설정 메일 요청 |
| 인증·권한 | 공개. 제출하는 토큰·입력값은 별도 검증 |

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

DTO: `ResetPasswordTokenRequest`

```json
{
  "email": "user@example.com"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `email` | String | O | 공백 불가·이메일 형식·최대 254자 |

---

## Response

HTTP Status

```text
202 Accepted
```

Response DTO: `ApiResponse<Void>`

```json
{
  "success": true,
  "data": null,
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

미등록 이메일이나 1분 이내 재요청도 같은 접수 응답이다. 메일 링크 토큰은 5분 유효하다. 숫자 코드·방식 선택 필드는 없다. 요청 단계에서는 ACTIVE 여부를 검사하지 않는다.

---

# 12. [링크 토큰으로 비밀번호 재설정] - 공개

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/pwd/reset/{token}` |
| Method | `POST` |
| 설명 | 링크 토큰으로 비밀번호 재설정 |
| 인증·권한 | 공개. 제출하는 토큰·입력값은 별도 검증 |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| `token` | String | 발급받은 링크 토큰 |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |

### Request Body

DTO: `ResetPasswordRequest`

```json
{
  "password": "ChangedPwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `password` | String | O | 공백 불가·6~64자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<Void>`

### 처리 규칙 및 예외

토큰 소비 → 이메일로 계정 조회 → ACTIVE 확인 → 비밀번호 저장. 무효 토큰 `A009`, 계정 없음 `A001`, 상태 오류 `A006`. 기존 비밀번호와 동일한지 검사하지 않으며 토큰 소비와 DB 변경은 원자적이지 않다.

---

# 13. [재활성화 메일 요청] - User

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/me/reactivation/verification` |
| Method | `POST` |
| 설명 | 재활성화 메일 요청 |
| 인증·권한 | Bearer JWT, 본인 UUID는 `sub`에서 추출 |

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

DTO: 없음. Request Body를 사용하지 않는다.

JWT UUID로 계정을 식별한다.

---

## Response

HTTP Status

```text
202 Accepted
```

Response DTO: `ApiResponse<Void>`

```json
{
  "success": true,
  "data": null,
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

INACTIVE만 가능하며 다른 상태는 `409 A006`, 계정 없음은 `A001`. 계정별 발급 간격 30초, 토큰 TTL 10분. 제한 시간 내 재요청은 추가 발급 없이 202.

---

# 14. [재활성화 토큰 확인] - User

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/me/reactivation/confirm` |
| Method | `POST` |
| 설명 | 재활성화 토큰 확인 |
| 인증·권한 | Bearer JWT, 본인 UUID는 `sub`에서 추출 |

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

DTO: `ReactivationConfirmRequest`

```json
{
  "token": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `token` | String | O | 공백 불가, 정규식 `^[0-9a-f]{64}$` |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

### 처리 규칙 및 예외

Lua로 소유 계정을 확인한 뒤 토큰을 삭제하고 INACTIVE → ACTIVE로 변경한다. 소유자 불일치·만료·사용 완료 `A009`, 상태 오류 `A006`, 계정 없음 `A001`. Front는 성공 후 현재 세션을 종료해 새 로그인을 요구한다.

---

# 15. [관리자 회원 목록] - Admin

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/admin` |
| Method | `GET` |
| 설명 | 관리자 회원 목록 |
| 인증·권한 | Bearer JWT의 ADMIN 권한 + DB의 요청자가 ADMIN·ACTIVE |

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

DTO: 없음. Request Body를 사용하지 않는다.

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<List<AccountResponse>>`

```json
{
  "success": true,
  "data": [
    {
      "uuid": "00000000-0000-4000-8000-000000000001",
      "name": "홍길동",
      "email": "user@example.com",
      "accountRole": "USER",
      "accountStatus": "ACTIVE",
      "createdAt": "2026-09-17T12:00:00",
      "updatedAt": "2026-09-17T12:00:00",
      "withdrawnAt": null
    }
  ],
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

전체 목록이며 Account의 검색·페이지 API가 아니다. Front에서 필터·정렬·페이징한다. 탈퇴 계정도 포함한다.

---

# 16. [관리자 회원 상세] - Admin

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/admin/{uuid}` |
| Method | `GET` |
| 설명 | 관리자 회원 상세 |
| 인증·권한 | Bearer JWT의 ADMIN 권한 + DB의 요청자가 ADMIN·ACTIVE |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| `uuid` | UUID | 대상 계정 UUID |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |

### Request Body

DTO: 없음. Request Body를 사용하지 않는다.

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

```json
{
  "success": true,
  "data": {
    "uuid": "00000000-0000-4000-8000-000000000001",
    "name": "홍길동",
    "email": "user@example.com",
    "accountRole": "USER",
    "accountStatus": "ACTIVE",
    "createdAt": "2026-09-17T12:00:00",
    "updatedAt": "2026-09-17T12:00:00",
    "withdrawnAt": null
  },
  "error": null,
  "timestamp": "2026-09-17T12:00:00"
}
```

### 처리 규칙 및 예외

대상 없음 `404 A001`.

---

# 17. [관리자 계정 생성] - Admin

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/admin` |
| Method | `POST` |
| 설명 | 관리자 계정 생성 |
| 인증·권한 | Bearer JWT의 ADMIN 권한 + DB의 요청자가 ADMIN·ACTIVE |

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

DTO: `CreateAdminAccountRequest`

```json
{
  "name": "관리자",
  "email": "admin@example.com",
  "password": "ExamplePwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `name` | String | O | 공백 불가·최대 100자 |
| `email` | String | O | 공백 불가·이메일 형식·최대 254자 |
| `password` | String | O | 공백 불가·6~64자 |

초대 토큰 없음.

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

### 처리 규칙 및 예외

ADMIN·ACTIVE로 생성한다. 중복 이메일 `409 A002`.

---

# 18. [관리자 회원 이름 수정] - Admin

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/admin/{uuid}` |
| Method | `PUT` |
| 설명 | 관리자 회원 이름 수정 |
| 인증·권한 | Bearer JWT의 ADMIN 권한 + DB의 요청자가 ADMIN·ACTIVE |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| `uuid` | UUID | 대상 계정 UUID |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |

### Request Body

DTO: `UpdateAccountNameRequest`

```json
{
  "name": "김회원"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `name` | String | O | 공백 불가·1~100자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

### 처리 규칙 및 예외

대상 ACTIVE 필요. 대상 없음 `A001`, 상태 오류 `A006`. 이메일·권한·비밀번호는 수정하지 않는다.

---

# 19. [관리자 비밀번호 재설정] - Admin

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/admin/{uuid}/pwd` |
| Method | `PUT` |
| 설명 | 관리자 비밀번호 재설정 |
| 인증·권한 | Bearer JWT의 ADMIN 권한 + DB의 요청자가 ADMIN·ACTIVE |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| `uuid` | UUID | 대상 계정 UUID |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |

### Request Body

DTO: `AdminResetPasswordRequest`

```json
{
  "password": "ChangedPwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `password` | String | O | 공백 불가·6~64자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

### 처리 규칙 및 예외

대상 ACTIVE 필요. 현재 비밀번호와 동일 여부를 검사하지 않는다. 대상 없음 `A001`, 상태 오류 `A006`.

---

# 20. [관리자 회원 상태 변경] - Admin

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/admin/{uuid}/status` |
| Method | `PUT` |
| 설명 | 관리자 회원 상태 변경 |
| 인증·권한 | Bearer JWT의 ADMIN 권한 + DB의 요청자가 ADMIN·ACTIVE |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| `uuid` | UUID | 대상 계정 UUID |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |

### Request Body

DTO: `ChangeAccountStatusRequest`

```json
{
  "action": "LOCK",
  "reason": "운영 확인을 위한 잠금"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `action` | Enum | O | `LOCK`, `UNLOCK`, `DEACTIVATE`, `REACTIVATE` |
| `reason` | String | O | 공백 불가 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<AccountResponse>`

### 처리 규칙 및 예외

허용 전이는 ACTIVE→LOCKED, LOCKED→ACTIVE, ACTIVE→INACTIVE, INACTIVE→ACTIVE다. 그 외 `409 A006`. 사유는 검증하지만 저장하지 않는다. 요청자 자신의 상태 변경을 별도로 차단하지 않는다.

---

# 21. [관리자 회원 탈퇴] - Admin

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/admin/{uuid}` |
| Method | `DELETE` |
| 설명 | 관리자 회원 탈퇴 |
| 인증·권한 | Bearer JWT의 ADMIN 권한 + DB의 요청자가 ADMIN·ACTIVE |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| `uuid` | UUID | 대상 계정 UUID |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| - | - | - | 없음 |

### Request Body

DTO: `WithdrawAccountRequest`

```json
{
  "password": "ExamplePwd2026!"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `password` | String | O | 대상 계정의 현재 비밀번호, 공백 불가·6~64자 |

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `ApiResponse<Void>`

### 처리 규칙 및 예외

본인 탈퇴와 같은 서비스 메서드를 사용한다. 관리자 비밀번호를 받는 기능이 아니다. `A001`, `A008`, `G004` 등은 본인 탈퇴와 같다. Front 관리자 화면 라우트는 확인되지 않았다.

---

# 22. [공개키 목록] - 공개

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/auth/.well-known/jwks.json` |
| Method | `GET` |
| 설명 | 공개키 목록 |
| 인증·권한 | 공개. 제출하는 토큰·입력값은 별도 검증 |

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

DTO: 없음. Request Body를 사용하지 않는다.

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: JWKS JSON 객체

```json
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "account-key-001",
      "n": "example-public-modulus",
      "e": "AQAB"
    }
  ]
}
```

### 처리 규칙 및 예외

RSA 공개키 집합을 반환한다. Account는 `GET /.well-known/jwks.json`도 제공한다. Gateway의 Account 경로 및 공개 목록에는 `/api/auth/.well-known/jwks.json`이 등록되어 있다. 개인키는 응답하지 않는다.

---

# 23. [내부 이메일 검색] - 내부 서비스

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/internal/search` |
| Method | `GET` |
| 설명 | 내부 이메일 검색 |
| 인증·권한 | 내부 서비스용. Account SecurityConfig에서는 permitAll; 외부 공개용 인증 정책으로 사용하지 않음 |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| - | - | 없음 |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `email` | String | O | 검색할 이메일 |

### Request Body

DTO: 없음. Request Body를 사용하지 않는다.

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `List<InternalAccountInfoResponse>` (배열 자체)

```json
[
  {
    "accountUuid": "00000000-0000-4000-8000-000000000001",
    "name": "홍길동",
    "email": "user@example.com"
  }
]
```

### 처리 규칙 및 예외

`@`가 있으면 정규화한 이메일 정확 일치로 조회한다. 없으면 입력값 뒤에 `@`를 붙인 접두어로 조회한다. WITHDRAWN을 제외하며 일치 없음은 `404 A001`이다.

---

# 24. [내부 UUID 목록 조회] - 내부 서비스

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/internal` |
| Method | `GET` |
| 설명 | 내부 UUID 목록 조회 |
| 인증·권한 | 내부 서비스용. Account SecurityConfig에서는 permitAll; 외부 공개용 인증 정책으로 사용하지 않음 |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| - | - | 없음 |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `uuids` | List<String> | O | UUID 값 목록. 예: `?uuids=00000000-0000-4000-8000-000000000001` |

### Request Body

DTO: 없음. Request Body를 사용하지 않는다.

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `List<InternalAccountInfoResponse>`

### 처리 규칙 및 예외

공백 제거·UUID 변환·중복 제거 후 조회한다. 존재하지 않거나 탈퇴한 계정은 결과에서 빠진다. 입력 순서를 유지한다. 서비스에서 잘못된 UUID는 `400 G001`로 처리한다.

---

# 25. [내부 단건 탈퇴] - 내부 서비스

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/internal` |
| Method | `DELETE` |
| 설명 | 내부 단건 탈퇴 |
| 인증·권한 | 내부 서비스용. Account SecurityConfig에서는 permitAll; 외부 공개용 인증 정책으로 사용하지 않음 |

---

## Request

### Path Variable

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| - | - | 없음 |

### Query Parameter

| 이름 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| `account-uuid` | String | O | 탈퇴할 계정의 UUID 문자열 |

### Request Body

DTO: 없음. Request Body를 사용하지 않는다.

---

## Response

HTTP Status

```text
200 OK
```

Response DTO: `InternalAccountInfoResponse` (객체 자체)

### 처리 규칙 및 예외

비밀번호 확인이나 Inventory 조직 탈퇴를 재호출하지 않고 Account만 탈퇴 처리한다. 응답은 탈퇴 직전 정보를 변환한 객체다. 계정 없음 `A001`, 이미 탈퇴한 상태 `A006`. 이 내부 응답은 외부 AccountResponse와 다르다.

---

# 26. [내부 일괄 탈퇴] - 내부 서비스

## 기본 정보

| 항목 | 내용 |
| --- | --- |
| URL | `/api/accounts/internal/bulk-delete` |
| Method | `POST` |
| 설명 | 내부 일괄 탈퇴 |
| 인증·권한 | 내부 서비스용. Account SecurityConfig에서는 permitAll; 외부 공개용 인증 정책으로 사용하지 않음 |

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

DTO: `List<String>`

```json
[
  "00000000-0000-4000-8000-000000000001"
]
```

| 필드 | 타입 | 필수 여부 | 설명 |
| --- | --- | --- | --- |
| 본문 배열 | List<String> | - | UUID 문자열 배열 |

DTO 수준의 `@Valid`·최대 건수 제한은 선언되어 있지 않다.

---

## Response

HTTP Status

```text
204 No Content
```

Response DTO: 본문 없음

### 처리 규칙 및 예외

한 DB 트랜잭션에서 순서대로 탈퇴 처리한다. 계정 없음 `A001`, 이미 탈퇴 `A006`. 비밀번호 확인·Inventory 재호출은 없다. UUID 파싱 오류를 별도로 G001에 매핑하는 분기는 이 컨트롤러에 없다.

---

# 구현 근거

- [AuthController.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/controller/AuthController.java#L23)
- [AccountController.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/controller/AccountController.java#L34)
- [AccountAdminController.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/controller/AccountAdminController.java#L29)
- [AccountsInternalController.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/controller/AccountsInternalController.java#L19)
- [JwkSetController.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/controller/JwkSetController.java#L16)
- [SecurityConfig.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/config/SecurityConfig.java#L80)
- [AccountService.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/service/AccountService.java#L45)
- [AuthService.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/service/AuthService.java#L28)
- [GlobalExceptionHandler.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/global/error/GlobalExceptionHandler.java#L49)
- [InventoryApiClient.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/global/client/InventoryApiClient.java#L57)
- [ErrorCode.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/global/error/ErrorCode.java#L9)
- [AccountResponse.java](https://github.com/nhnacademy-aiot3-iUnoT/account/blob/b39666c757e8ca8bc12a8102e1bac97b5845a109/src/main/java/com/nhnacademy/account/dto/response/AccountResponse.java#L10)
- [application.yaml](https://github.com/nhnacademy-aiot3-iUnoT/msa-infra/blob/ef5f822f88e09677dbbd7f8d05c9b08cf7c41f3d/gateway/src/main/resources/application.yaml#L22)
