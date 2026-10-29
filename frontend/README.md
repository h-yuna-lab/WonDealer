# 원딜러 프론트엔드 · 기술 안내

React 화면의 실행·API 연결·외부 서비스 설정 안내입니다. 프로젝트 소개·담당 업무·전체 화면·회고는 [최상위 README](../README.md)에 있습니다.

## 의존성

- **화면:** React 18, JavaScript, React Router 6, styled-components, CSS
- **통신:** Axios, STOMP, SockJS
- **이미지 업로드:** Firebase Storage
- **WonPay 충전 연동:** PortOne 브라우저 SDK
- **아이콘·차트:** lucide-react, Recharts
- **협업:** Git, GitHub, Notion

기획 문서에는 S3·토스로 적혀 있지만, 실제 이미지 업로드와 충전 코드는 Firebase Storage와 PortOne을 사용합니다.

## 폴더 구조

```text
src/
├─ api/          # 인증·상품·경매·거래 API
├─ components/   # Navbar, Footer, 인증 라우트
├─ context/      # 로그인 상태, 테마
├─ hooks/        # WebSocket 연결
├─ pages/        # 메인·회원·상품·경매·채팅·마이페이지·관리자
├─ styles/       # 전역 스타일 및 테마
└─ utils/        # 공통 설정, 이미지 업로드 등
```

## 실행·설정·빌드

Node.js와 Yarn Classic 또는 npm을 준비합니다. 원본의 `yarn.lock`을 사용하려면 다음과 같이 실행합니다.

```bash
yarn install --frozen-lockfile
yarn start
```

npm을 사용하는 경우 `npm install` 후 `npm start`를 실행합니다.

- 프론트엔드: `http://localhost:3000`
- 백엔드: `http://localhost:8111`
- 이미지 업로드·WonPay 충전을 사용할 때 `.env.example`을 `.env`로 복사하고 본인 서비스의 설정값을 입력합니다.
- API 주소는 `src/utils/Common.js`, WebSocket 주소는 `src/hooks/useWebSocket.js`, 구글 로그인 주소는 `src/pages/auth/LoginPage.js`에 지정되어 있습니다. `.env`의 `REACT_APP_API_URL`을 바꿔도 위 파일들의 주소는 바뀌지 않습니다.
- 전체 기능을 이용하려면 백엔드·MySQL·메일·OAuth·외부 서비스 설정이 필요합니다.

```bash
yarn build
```

## 인증·실시간 연결

- 가입 화면에서 인증 메일을 요청하고, 인증 링크 확인 후 회원 정보·약관을 제출합니다. 백엔드가 인증 상태와 가입 조건을 검증합니다.
- Axios 인터셉터가 Access Token을 요청에 첨부하고 인증 오류 응답에서 토큰 재발급을 요청합니다.
- Google OAuth 로그인 성공 후 `/login-success` 화면으로 이동합니다.
- 경매·채팅은 STOMP over SockJS로 실시간 이벤트를 수신합니다.

서버 API와 설정은 [백엔드 기술 안내](../backend/README.md)를 확인하세요.

## 연동 전 확인사항

프론트 프로덕션 빌드가 성공했고, 메인·테마 전환·로그인·회원가입·아이디 찾기·비밀번호 재설정 화면을 로컬 브라우저에서 확인했습니다. 백엔드 연결과 실제 결제·메일 발송은 검증하지 않았습니다.

원본 브랜치의 일부 API 호출과 백엔드 `dev`의 제공 API에는 차이가 있습니다. 상품 수정의 `PATCH`/`PUT`, 찜·일부 경매 보조 API 등은 전체 연동 실행 전에 확인이 필요합니다. 애플리케이션 코드는 팀 결과물 그대로 보관했습니다.

## 문서

- [프로젝트 포트폴리오·화면·회고](../README.md)
- [백엔드 API·설정](../backend/README.md)
- [통합 출처](../SOURCE.md) · [프론트 출처 기록](docs/SOURCE.md)
