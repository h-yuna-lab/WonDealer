# 원딜러 백엔드 · 기술 안내

회원·인증 및 거래 API의 구현 파일, 환경 설정과 실행 안내입니다. 프로젝트 소개·개인 담당 업무·팀 구성·회고는 [최상위 README](../README.md)에 있습니다.

## 의존성

Java 17, Spring Boot 3.5.14, Spring Security, Spring Data JPA, MySQL, JWT (jjwt), OAuth2 Client, Spring Mail, WebSocket/STOMP, Jsoup, Lombok, Springdoc OpenAPI, Gradle.

## 폴더 구조

```text
src/main/java/com/wondealer/
├─ config/       # 보안·WebSocket·초기 데이터
├─ controller/   # REST API 및 메시지 컨트롤러
├─ service/      # 회원·상품·경매·거래 비즈니스 로직
├─ repository/   # Spring Data JPA
├─ entity/       # 회원·약관·거래 등 도메인
├─ dto/          # 요청·응답·실시간 메시지
├─ security/     # JWT·OAuth 인증 연계
├─ scheduler/    # 경매 종료·게임 순위 갱신
├─ crawler/      # 게임 순위 수집
└─ exception/    # 공통 예외 처리
```

## 실행·설정·빌드

**Java 17와 MySQL 8.0**을 준비하고, MySQL에 `won_dealer` 데이터베이스를 생성합니다.

```sql
CREATE DATABASE won_dealer CHARACTER SET utf8mb4;
```

`src/main/resources/application.example.properties`를 같은 폴더의 `application.properties`로 복사합니다. `${...}`에 해당하는 환경변수를 설정하거나 로컬 설정 파일에 본인의 값을 입력합니다.

- DB 접속: `DB_USERNAME`, `DB_PASSWORD`
- JWT: `JWT_SECRET` · 32바이트 이상 난수의 Base64 문자열
- 메일: `MAIL_USERNAME`, `MAIL_PASSWORD`
- Google OAuth: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- 초기 관리자: `ADMIN_USERNAME`, `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_NAME`
- PortOne 충전 확인: `PORTONE_API_SECRET` · 충전 기능 사용 시 설정

Google OAuth 승인된 리디렉션 URI는 `http://localhost:8111/login/oauth2/code/google`입니다. 로그인 성공 후 프론트의 `http://localhost:3000/login-success`로 이동합니다.

```bash
# macOS / Linux
./gradlew bootRun

# Windows PowerShell
.\gradlew.bat bootRun
```

- API: `http://localhost:8111`
- Swagger UI: `http://localhost:8111/swagger-ui/index.html`
- 프론트엔드: `http://localhost:3000`
- 최초 실행 시 `DataInitializer`에서 관리자·약관·게임·서버·카테고리 데이터를 생성합니다.
- 실제 `application.properties`, `.env`, 서비스 키는 Git에 포함하지 않습니다.

```bash
./gradlew build
# Windows: .\gradlew.bat build
```

## 회원 기능 구현

### 1. 회원가입 및 약관 동의

- 가입 전 이메일 인증 완료 여부 확인
- 이메일·아이디·닉네임 중복 검사
- 필수 약관 동의 및 제출한 약관 ID 검증
- BCrypt로 비밀번호를 암호화하여 회원 저장
- 회원별 약관 동의 내역 저장 및 WonPay 지갑 생성 연계

관련 코드: [AuthController](src/main/java/com/wondealer/controller/AuthController.java), [AuthService](src/main/java/com/wondealer/service/AuthService.java), [SignUpReqDto](src/main/java/com/wondealer/dto/request/SignUpReqDto.java)

### 2. 로그인 및 인증 연계

- 아이디 또는 이메일을 이용한 로그인
- 차단된 회원의 로그인 제한
- 팀의 Spring Security·JWT 기반에서 로그인 인증 및 토큰 발급 연계
- Refresh Token 저장·갱신·검증 및 로그아웃 시 삭제

관련 코드: [AuthService](src/main/java/com/wondealer/service/AuthService.java), [CustomUserDetailsService](src/main/java/com/wondealer/service/CustomUserDetailsService.java)

### 3. 이메일 인증

- 가입 전 인증 메일 발송 및 이미 가입된 이메일 검증
- UUID 인증 토큰 생성 및 저장
- 메일의 인증 링크를 통한 토큰 확인
- 인증 토큰 만료 확인과 인증 완료 상태 반영
- 메일 인증·발송 실패에 대한 오류 처리

관련 코드: [EmailService](src/main/java/com/wondealer/service/EmailService.java), [EmailVerify](src/main/java/com/wondealer/entity/EmailVerify.java), [AuthService](src/main/java/com/wondealer/service/AuthService.java)

### 4. 구글 소셜 로그인

- Google OAuth2 사용자 정보 조회
- 제공자·외부 사용자 ID를 내부 소셜 계정과 연결
- 기존 이메일 회원 연결 또는 신규 회원 생성
- 소셜 로그인 성공 후 JWT 발급·Refresh Token 저장·프론트 리디렉션 연계

관련 코드: [CustomOAuth2UserService](src/main/java/com/wondealer/service/CustomOAuth2UserService.java), [OAuth2SuccessHandler](src/main/java/com/wondealer/security/OAuth2SuccessHandler.java), [OauthAccount](src/main/java/com/wondealer/entity/OauthAccount.java)

### 5. 마이페이지 및 회원 정보 관리

- 현재 로그인 회원의 정보 조회
- 닉네임·전화번호·프로필 이미지 URL 수정
- 현재 비밀번호 검증 후 새 비밀번호 암호화 저장
- 은행명·계좌번호·예금주 등록 및 수정
- 본인의 판매 상품·구매/판매 거래·입찰 내역을 페이지 단위로 조회

관련 코드: [MemberController](src/main/java/com/wondealer/controller/MemberController.java), [MemberService](src/main/java/com/wondealer/service/MemberService.java), [MemberResDto](src/main/java/com/wondealer/dto/response/MemberResDto.java)

### 6. 계정 찾기 및 비밀번호 재설정

- 이름과 이메일에 일치하는 회원의 아이디 조회
- 반환 아이디를 일부 마스킹하여 표시
- 임시 비밀번호 생성·암호화·저장 후 이메일 안내

관련 코드: [AuthService](src/main/java/com/wondealer/service/AuthService.java), [EmailService](src/main/java/com/wondealer/service/EmailService.java)

## 회원가입 처리 순서

1. 인증 메일 요청 시 UUID 토큰을 저장하고 메일을 발송합니다.
2. 인증 링크에서 토큰 유효성·만료를 확인하고 인증 완료 상태를 반영합니다.
3. 회원 등록 요청에서 이메일 인증, 중복 정보와 필수 약관을 검사합니다.
4. 비밀번호를 암호화하고 회원·약관 동의 내역을 저장합니다.
5. WonPay 지갑 생성과 연결합니다.

## 회원·인증 API

| 기능 | 메서드 | 경로 |
| --- | --- | --- |
| 회원가입 | POST | `/auth/signup` |
| 로그인 | POST | `/auth/login` |
| 토큰 재발급 | POST | `/auth/reissue` |
| 로그아웃 | POST | `/auth/logout` |
| 아이디 찾기 | POST | `/auth/find-username` |
| 임시 비밀번호 발급 | POST | `/auth/reset-password` |
| 인증 메일 발송 | POST | `/auth/email/send` |
| 이메일 인증 확인 | POST | `/auth/email/verify` |
| Google OAuth 진입 | GET | `/oauth2/authorization/google` |
| 내 정보 조회·수정 | GET / PATCH | `/api/members/me` |
| 비밀번호 변경 | PATCH | `/api/members/me/password` |
| 계좌 등록·수정 | PATCH | `/api/members/me/bank` |
| 내 판매 상품 | GET | `/api/members/me/items` |
| 내 구매·판매 내역 | GET | `/api/members/me/trades` |
| 내 입찰 내역 | GET | `/api/members/me/bids` |

Google OAuth 진입·콜백 경로는 Spring Security가 처리합니다. API 목록은 이 복사본의 컨트롤러를 기준으로 작성했습니다.

## 경매·거래 처리

입찰 시 `AuctionRepository`의 **비관적 쓰기 락**을 사용합니다. 입찰 금액·자기 상품 입찰·잔액 등을 검증하고 이전 최고 입찰자 환불 및 현재가 변경을 처리한 뒤, STOMP로 경매 상태를 전송합니다. 기획 문서의 낙관적 락 설명보다 복사본의 실제 구현을 기준으로 정리했습니다.

거래 상태는 `PENDING_PAYMENT → PAID → COMPLETED`로 관리하며, 결제 완료 상태에서는 간이 에스크로로 보관하고 수령 확인 시 판매자 정산을 처리합니다. 

## 연동 전 확인사항

이번 정리 환경에는 Java·MySQL 및 외부 서비스 인증 설정이 없어 백엔드 빌드·통합 실행을 검증하지 않았습니다. 프론트 빌드와 인증 관련 화면은 확인했으며 [Frontend의 화면 캡처](../frontend/README.md)에서 볼 수 있습니다.

프론트 통합 브랜치와 백엔드 `dev`에는 상품 수정의 `PATCH`/`PUT` 등 일부 API 차이가 남아 있으므로 전체 기능 실행 전 연동 확인이 필요합니다.

## 문서

- [프로젝트 포트폴리오·팀 구성·회고](../README.md)
- [프론트 실행·설정](../frontend/README.md)
- [통합 출처](../SOURCE.md) · [백엔드 복사 기준](docs/SOURCE.md)
