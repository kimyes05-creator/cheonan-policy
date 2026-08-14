# 천안시 AI 에이전트 공유 허브

직원들이 바이브코딩으로 직접 만든 업무용 AI 에이전트를 한곳에 모아 서로 공유하고,
좋아요·의견으로 호응을 남기는 내부 공유 공간입니다. 반응이 좋은 도구는 함께 고도화합니다.

## 📁 구성

| 파일 | 설명 | 배포 주소(GitHub Pages 기준) |
|------|------|------------------------------|
| `index.html` | **인구정책 찾기** 에이전트 (기존 앱, 그대로 유지) | `/` |
| `hub.html` | **AI 에이전트 공유 허브** (이번에 추가) | `/hub.html` |

- 공유 허브 주소: `https://kimyes05-creator.github.io/cheonan-policy/hub.html`
- 허브 하단 링크와 "인구정책 찾기" 카드의 **체험하기** 버튼이 기존 앱(`index.html`)으로 연결됩니다.

> 나중에 공유 허브를 메인 화면으로 쓰고 싶으면, `index.html`을 `apps/population.html` 등으로 옮기고
> `hub.html`을 `index.html`로 이름만 바꾸면 됩니다. (지금은 안전하게 별도 페이지로 두었습니다.)

## ✨ 주요 기능

- **에이전트 갤러리** — 카드로 한눈에 둘러보기 (아이콘·부서·분류·소개)
- **좋아요(호응)** — 인기 있는 도구가 상단에 노출
- **의견/댓글** — 개선 아이디어를 남겨 함께 고도화
- **검색 / 분류 필터 / 정렬**(인기순·최신순·댓글순)
- **내 에이전트 등록 신청** — 직원이 직접 자기 도구를 제출 → 담당자 검토 후 정식 등록
- **참여 통계** — 등록 수 / 누적 좋아요 / 참여 부서

## 🧪 데모 모드 vs 🟢 실시간 공유 모드

별도 설정 없이 열면 **데모 모드**로 동작합니다.
이때 좋아요·의견은 **보는 사람의 브라우저에만** 저장되어 직원 간에 공유되지 않습니다. (시연·검토용)

직원 모두가 **실시간으로 공유되는** 좋아요·의견을 쓰려면 아래 파이어베이스(Firebase) 설정을 한 번만 해주세요.

## 🔧 실시간 공유 켜기 (Firebase, 무료)

준비 시간 약 10분. 코딩 지식이 없어도 화면 안내대로 따라 하면 됩니다.

### 1) 프로젝트 만들기
1. https://console.firebase.google.com 접속 → **프로젝트 추가**
2. 프로젝트 이름 입력(예: `cheonan-ai-hub`) → 애널리틱스는 꺼도 됩니다 → 생성

### 2) Firestore 데이터베이스 만들기
1. 왼쪽 메뉴 **빌드 > Firestore Database** → **데이터베이스 만들기**
2. 위치는 `asia-northeast3 (서울)` 권장 → **프로덕션 모드**로 시작

### 3) 보안 규칙 설정
Firestore의 **규칙(Rules)** 탭에 아래를 붙여넣고 **게시**하세요.
(내부용 간단 규칙 — 좋아요·의견·등록신청만 읽고 쓰도록 허용)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /likes/{id}        { allow read, write: if true; }
    match /comments/{id}     { allow read, create: if true; }
    match /submissions/{id}  { allow read, create: if true; }
  }
}
```

> 더 엄격하게 운영하려면 나중에 Firebase 인증(직원 로그인)을 붙여 `if request.auth != null` 로 바꿀 수 있습니다.

### 4) 웹 앱 등록 후 설정값 복사
1. 프로젝트 개요 옆 **⚙️ 프로젝트 설정** → 아래 **내 앱**에서 **웹(</>)** 아이콘 클릭
2. 앱 닉네임 입력 → 등록하면 `firebaseConfig = { ... }` 값이 나옵니다. 이 값을 복사하세요.

### 5) hub.html에 붙여넣기
`hub.html`을 열어 상단의 `FIREBASE_CONFIG` 부분을 복사한 값으로 채웁니다.

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "cheonan-ai-hub.firebaseapp.com",
  projectId: "cheonan-ai-hub",
  storageBucket: "cheonan-ai-hub.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef"
};
```

저장 후 다시 열면 상단 배너가 **"✅ 실시간 공유 모드"** 로 바뀌고, 좋아요·의견이 모든 직원에게 공유됩니다.

> 참고: 위 설정값(apiKey 등)은 웹 프론트엔드용 공개 식별자로, 비밀번호가 아닙니다.
> 실제 데이터 접근은 위 3)의 보안 규칙으로 통제됩니다.

## ➕ 대표 에이전트 추가/수정하기

담당자가 정식 에이전트를 추가하려면 `hub.html`의 `SEED_AGENTS` 배열만 편집하면 됩니다.

```js
{
  id: "unique-id",              // 고유 id (영문/숫자)
  type: "official",             // official(정식) | example(예시)
  icon: "📊",
  title: "에이전트 이름",
  dept: "담당 부서",
  category: "데이터·통계",       // CATEGORIES 중 하나
  desc: "무엇을 어떻게 개선하는지 한두 문장 설명",
  link: "https://…",            // 체험 링크 (없으면 "")
  createdAt: "2026-08-14"
}
```

직원이 **등록 신청**한 도구는 "검토중" 카드로 표시됩니다. 담당자가 확인 후 위 `SEED_AGENTS`에
정식 항목으로 옮기면 정식 등록이 완료됩니다.

## 🚀 배포 (GitHub Pages)

이 저장소가 이미 GitHub Pages로 배포 중이라면, `hub.html`을 `main` 브랜치에 올리는 것만으로
`/hub.html` 주소로 바로 접속할 수 있습니다. (Settings → Pages에서 소스 브랜치 확인)
