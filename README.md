# SanGa Pro - 카카오 로그인 페이지

## 📋 개요

Chrome 확장 프로그램용 카카오 OAuth 로그인 페이지입니다.
`chrome-extension://` 프로토콜은 카카오 Redirect URI로 사용할 수 없으므로, HTTPS 웹 페이지를 사용합니다.

---

## 🚀 배포 방법

### 1. GitHub 리포지토리 생성

1. https://github.com/new 접속
2. Repository name: `sanga-auth`
3. **Public** 선택 (GitHub Pages 무료 사용)
4. "Create repository" 클릭

### 2. 파일 업로드

```bash
cd auth-pages

git init
git add .
git commit -m "Add Kakao OAuth callback page"
git branch -M main
git remote add origin https://github.com/ytp20231992/sanga-auth.git
git push -u origin main
```

### 3. GitHub Pages 활성화

1. Repository → Settings → Pages
2. Source: **Deploy from a branch**
3. Branch: **main** / **(root)**
4. Save

5분 후 배포 완료:
```
https://ytp20231992.github.io/sanga-auth/
```

---

## 🔧 설정

### 1. index.html 수정

`index.html` 파일을 열어 다음 값을 수정하세요:

```javascript
const SUPABASE_URL = 'https://YOUR_PROJECT.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR_ANON_KEY';
const KAKAO_REST_API_KEY = 'YOUR_KAKAO_REST_API_KEY';
```

### 2. 카카오 개발자 콘솔 설정

https://developers.kakao.com/console/app

**Redirect URI 등록:**
```
https://ytp20231992.github.io/sanga-auth/
```

**Web 플랫폼 등록:**
```
https://ytp20231992.github.io
```

### 3. Supabase 환경변수 설정

```bash
supabase secrets set KAKAO_REDIRECT_URI="https://ytp20231992.github.io/sanga-auth/"

# auth-kakao-callback 함수 재배포
supabase functions deploy auth-kakao-callback --no-verify-jwt
```

---

## 🔄 로그인 플로우

```
1. 사용자가 확장 프로그램에서 "카카오 로그인" 클릭
   ↓
2. 새 창(popup)으로 카카오 로그인 페이지 열림
   ↓
3. 카카오 로그인 완료 후 Redirect:
   https://ytp20231992.github.io/sanga-auth/?code=xxx
   ↓
4. index.html에서 인증 코드 처리:
   - 카카오 토큰 발급
   - Supabase auth-kakao-callback 호출
   - JWT 토큰 및 사용자 정보 수신
   ↓
5. 확장 프로그램으로 데이터 전달 (postMessage)
   ↓
6. 팝업 창 자동 닫힘
   ↓
7. 확장 프로그램에서 로그인 완료 처리
```

---

## 📝 확장 프로그램 연동

`auth/login-script.js` 파일에서 카카오 로그인 버튼 클릭 시:

```javascript
// 카카오 로그인 팝업 열기
function loginWithKakao() {
  const KAKAO_AUTH_URL = 'https://ytp20231992.github.io/sanga-auth/';
  const KAKAO_CLIENT_ID = 'YOUR_REST_API_KEY';

  const authUrl = `https://kauth.kakao.com/oauth/authorize?client_id=${KAKAO_CLIENT_ID}&redirect_uri=${encodeURIComponent(KAKAO_AUTH_URL)}&response_type=code`;

  // 팝업 창 열기
  const popup = window.open(authUrl, 'KakaoLogin', 'width=500,height=600');

  // postMessage 수신 대기
  window.addEventListener('message', async (event) => {
    if (event.data.type === 'KAKAO_LOGIN_SUCCESS') {
      // 로그인 성공
      await chrome.storage.local.set({
        authToken: event.data.token,
        user: event.data.user,
        subscription: event.data.subscription
      });

      // UI 업데이트
      window.location.reload();
    }
  });
}
```

---

## 🧪 테스트

### 로컬 테스트:

```bash
cd auth-pages
python -m http.server 8080
```

브라우저에서 http://localhost:8080?code=test 접속

### 프로덕션 테스트:

1. 확장 프로그램에서 "카카오 로그인" 클릭
2. 카카오 로그인 진행
3. 팝업 창이 자동으로 닫히는지 확인
4. 확장 프로그램에서 로그인 상태 확인

---

## 🔐 보안

- HTTPS 통신으로 안전성 보장
- postMessage로 확장 프로그램과 통신
- 토큰은 확장 프로그램의 로컬 스토리지에만 저장
- 웹 페이지에는 민감한 정보 저장하지 않음

---

## 📊 파일 구조

```
auth-pages/
├── index.html          # 카카오 OAuth 콜백 페이지
└── README.md          # 이 파일
```

---

**배포 후 확장 프로그램 로그인 연동 필요!**
"# busukgi-auth" 
