# 원본 출처 및 포트폴리오 정리 범위

| 항목 | 내용 |
| --- | --- |
| 원본 팀 저장소 | https://github.com/pyyzzz/wondealer-backend |
| 복사 브랜치 | `dev` |
| 복사 기준 커밋 | `9d97bed1a0f352c6781c9981dde89310fdac512f` |
| 원본 코드 | [고정 커밋 보기](https://github.com/pyyzzz/wondealer-backend/tree/9d97bed1a0f352c6781c9981dde89310fdac512f) |
| 정리일 | 2026.10.11 |
| 포트폴리오 소유자 | 홍유나 (`h-yuna-lab`) |

팀 결과물의 특정 시점 코드를 별도 포트폴리오 저장소로 복사했습니다. 팀 저장소의 커밋 기록은 위 원본 링크에서 확인할 수 있습니다. 이 저장소의 초기 커밋은 포트폴리오 정리 기록이며, 전체 코드를 개인이 작성했다는 의미가 아닙니다.

## 개인 담당 근거

[팀 Notion](https://app.notion.com/p/3f6cb363cc93800a8270cb210d838dca) 및 [설계명세서](https://app.notion.com/p/3f6cb363cc9380d0bdbfc29352b66cfe)의 역할표에서 홍유나는 백엔드 회원가입·로그인, 이메일 인증, 구글 OAuth, 약관 동의, 마이페이지, 계정 찾기·비밀번호 변경을 담당합니다.

JWT/Security 기반, WebSocket·크롤링·배포·관리자 API는 김현중, 상품·경매·WonPay·거래·채팅 REST API는 변후민의 담당입니다. README의 프로젝트 전체 기능과 개인 담당 범위를 구분했습니다.

## 추가·수정한 내용

- README: 프로젝트 개요, 개인 구현 설명·관련 코드, API 목록, 시스템 구조, 팀 역할 및 실행 방법
- `src/main/resources/application.example.properties`: 실제 인증정보가 없는 로컬 실행 설정 예시
- `.gitignore`: 로컬 환경 설정 파일 제외 범위 확장
- 이 출처 문서

Java 소스·Gradle 설정·Wrapper·테스트 소스는 원본 그대로 보관했습니다. 실제 `application.properties`, 운영 인증정보 및 사용자 데이터는 포함하지 않습니다.

## 자료와 코드의 차이

- 문서에 적힌 경매 낙관적 락 대신 코드의 `PESSIMISTIC_WRITE`를 기술했습니다.
- 문서에 적힌 S3·토스 구성과 달리 복사본의 프론트는 Firebase Storage·PortOne을 사용합니다.
- 기획상의 AI 트렌드 분석, 자동 정산 기간, 운영 배포 성공 등은 별도 확인 없이 구현 성과로 표시하지 않았습니다.
- 상품 수정의 HTTP 메서드 등 프론트 통합 브랜치와 백엔드 `dev` 사이의 API 차이는 README에 기재했습니다.

## 확인 범위

API·서비스·엔티티·설정 참조를 읽어 문서를 확인했습니다. 이 정리 환경에는 Java·MySQL 및 외부 서비스 설정이 없어 백엔드 빌드·통합 실행은 미검증입니다. 프론트는 프로덕션 빌드와 공개 인증 화면을 확인했습니다.
