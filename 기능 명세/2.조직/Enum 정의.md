# Enum 정의

## 조직 상태

| 값 | 설명 |
|----|------|
| ACTIVE | 정상 운영 |
| INACTIVE | 일시 비활성화 |
| CLOSED | 폐쇄 |

---

## 조직 <-> 회원 관계

| 값 | 설명 |
|----|------|
| ACTIVE | 현재 조직원 |
| LEFT | 자진 탈퇴 |
| REMOVED | 강제 제외 |

---

## 가입 요청 상태

| 값 | 설명 |
|----|------|
| PENDING | 승인 대기 |
| APPROVED | 승인 완료 |
| REJECTED | 거절 |
| CANCELED | 요청 취소 |

---

## 조직 내 역할

| 값 | 설명 |
|----|------|
| OWNER | 조직 생성자 |
| MEMBER | 일반 조직원 |

---

## 조직 내 권한

| 값 | 설명 |
|----|------|
| INVENTORY_MANAGE | 재고 관리 |
| ENVIRONMENT_MANAGE | 환경 관리 |
| OPERATION_MANAGE | 운영 관리 |

