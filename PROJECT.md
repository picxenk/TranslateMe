# PROJECT.md — TranslateMe

## 1. Project Overview

TranslateMe는 Ollama를 통해 로컬에서 실행되는 Google Translate와 유사한 최소한의 번역 웹앱이다.

사용자는 웹 브라우저에서 원문을 입력하고 언어를 선택하면, 로컬 Ollama API를 통해 TranslateGemma 모델로 번역한다.

### 핵심 목표

- 단일 HTML 파일로 구현
- 별도의 서버/빌드 과정 없음
- 모든 번역은 로컬 Ollama에서 처리
- Google Translate와 유사한 단순한 2-column UI
- 영어, 한국어 등 주요 언어 간 번역 지원
- Ollama 연결/CORS 오류 발생 시 해결 방법 안내
- MVP에서는 번역 이력, 로그인, 서버 저장 등을 구현하지 않음

---

## 2. Architecture

```text
┌──────────────────────────────────────────────┐
│                 Browser                      │
│                                              │
│  ┌────────────────┐   ┌──────────────────┐  │
│  │ Source Text    │ → │ Translation      │  │
│  │                │   │ Result           │  │
│  └────────────────┘   └──────────────────┘  │
│          │                    ▲              │
│          └────── fetch ──────┘              │
└────────────────────┬─────────────────────────┘
                     │ HTTP
                     ▼
          http://localhost:11434
                     │
                     ▼
              Ollama API
                     │
                     ▼
          translategemma:4b
```

### 기술 스택

- HTML
- CSS
- Vanilla JavaScript
- Fetch API
- Ollama HTTP API
- TranslateGemma

외부 JavaScript 프레임워크나 패키지를 사용하지 않는다.

---

## 3. Runtime Requirements

사용자 컴퓨터에 다음이 설치되어 있어야 한다.

1. Ollama
2. TranslateGemma 모델
3. 웹 브라우저

모델 설치 예:

```bash
ollama pull translategemma:4b
```

실행 확인:

```bash
ollama run translategemma:4b
```

Ollama의 기본 API endpoint:

```text
http://localhost:11434
```

TranslateGemma는 Ollama에서 직접 `/api/chat`을 통해 호출할 수 있다.

---

## 4. MVP UI

화면은 Google Translate와 유사한 구조를 사용한다.

```text
┌──────────────────────────────────────────────────────────┐
│  TranslateMe          Local translation powered by Ollama│
│                      ● Ollama connected                  │
│                      | translategemma:latest ready       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  [ English ▼ ]              [ Korean ▼ ]     [⇄]         │
│                                                          │
│  ┌──────────────────────┐  ┌──────────────────────────┐  │
│  │                      │  │                          │  │
│  │Enter text...         │  │Translation               │  │
│  │                      │  │                          │  │
│  │                      │  │                          │  │
│  └──────────────────────┘  └──────────────────────────┘  │
│                                                          │
│                    [ Translate ]                         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### UI 원칙

- 흰색 배경
- 얇은 border
- 둥근 모서리
- 최소한의 색상
- 충분한 여백
- 시스템 기본 sans-serif 사용
- 반응형 레이아웃
- 모바일에서는 두 영역을 세로로 배치

화려한 애니메이션이나 장식적인 UI는 MVP에서 제외한다.

---

## 5. UI Components

### Header

좌측에 앱 이름을 표시한다.

```text
TranslateMe
```

우측 상단에 작은 설명과 연결/모델 상태를 함께 표시한다.

```text
Local translation powered by Ollama
● Ollama connected | translategemma:latest ready
```

모바일에서는 헤더가 세로 스택으로 전환되고 좌측 정렬된다.

---

### Language Selector

Source / Target 언어를 선택한다.

MVP에서는 우선 다음 언어를 제공한다.

- Auto Detect
- English
- Korean
- Japanese
- Chinese
- French
- German
- Spanish

추후 TranslateGemma의 전체 지원 언어 목록으로 확장할 수 있다.

TranslateGemma 자체는 55개 언어를 지원한다.

---

### Swap Button

두 언어를 서로 교환한다.

예:

```text
English → Korean
```

↓

```text
Korean → English
```

Source text와 translation result도 함께 교환한다.

단, Source가 `Auto Detect`인 경우에는 swap 동작을 제한하거나 적절히 처리한다.

---

### Source Text Area

placeholder:

```text
Enter text to translate
```

특징:

- multiline textarea
- 입력 글자 수 표시
- 비어 있으면 Translate 버튼 비활성화

---

### Translation Area

번역 결과를 표시한다.

번역 중:

```text
Translating...
```

완료:

```text
번역 결과
```

오류:

```text
Translation failed.
```

오류 발생 시 구체적인 오류 내용을 별도의 작은 영역에 표시한다.

---

### Translate Button

버튼을 누르면 Ollama API를 호출한다.

`Cmd/Ctrl + Enter` 단축키로도 번역을 실행할 수 있다.

Source와 Target 언어가 같으면 요청을 보내지 않고 안내 메시지를 표시한다.

MVP에서는 자동 번역(auto translate)을 사용하지 않는다.

즉:

```text
입력 → Translate 버튼 → 번역
```

방식으로 단순하게 구현한다.

---

### Status Indicator

Header 우측 상단에 Ollama 연결 상태를 표시한다.

작은 설명("Local translation powered by Ollama")과 모델 상태가 같은 영역에 함께 표기된다.

예:

```text
● Ollama connected
```

또는

```text
○ Ollama unavailable
```

페이지 로딩 시 Ollama API 연결 여부를 확인한다.

---

## 6. Ollama API

Ollama의 `/api/chat` endpoint를 사용한다.

```text
POST http://localhost:11434/api/chat
```

Request 예:

```json
{
  "model": "translategemma:4b",
  "messages": [
    {
      "role": "user",
      "content": "..."
    }
  ],
  "stream": false
}
```

응답에서 다음 값을 사용한다.

```text
message.content
```

MVP에서는 streaming을 사용하지 않는다.

향후 필요하면 streaming response로 확장한다.

---

## 7. Translation Prompt

TranslateGemma가 권장하는 번역 프롬프트 구조를 사용한다.

```text
You are a professional {SOURCE_LANG} ({SOURCE_CODE}) to {TARGET_LANG} ({TARGET_CODE}) translator. Your goal is to accurately convey the meaning and nuances of the original {SOURCE_LANG} text while adhering to {TARGET_LANG} grammar, vocabulary, and cultural sensitivities.

Produce only the {TARGET_LANG} translation, without any additional explanations or commentary. Please translate the following {SOURCE_LANG} text into {TARGET_LANG}:


{TEXT}
```

중요:

- 번역 결과만 출력하도록 명시
- 추가 설명을 요청하지 않음
- 원문의 의미와 뉘앙스를 유지하도록 요청
- 언어명과 언어 코드를 함께 전달

이 형식은 TranslateGemma 공식 Prompt Guide의 권장 형식에 따른다.

---

## 8. Language Configuration

JavaScript 내부에 간단한 language map을 둔다.

예:

```javascript
const LANGUAGES = {
  en: "English",
  ko: "Korean",
  ja: "Japanese",
  zh: "Chinese",
  fr: "French",
  de: "German",
  es: "Spanish"
};
```

Select에는 표시 이름을 사용하고 API prompt에는 code를 사용한다.

예:

```text
English (en)
Korean (ko)
```

---

## 9. Auto Detect

MVP에서 `Auto Detect`를 지원할 경우 별도의 언어 감지 모델을 사용하지 않는다.

TranslateGemma prompt에서 source language를 다음처럼 처리한다.

```text
the source language detected from the input
```

다만 TranslateGemma의 공식 prompt가 명시적인 source language를 전제로 하므로, 명시적 Source Language 선택을 기본으로 한다.

기본 초기 UI:

```text
English ▼  →  Korean ▼
```

구현 상태:

- 드롭다운에 `Auto Detect`가 포함되어 있으며, 선택 시 prompt에서 source language를 `the source language detected from the input`로 처리한다.
- `Auto Detect` 선택 시 Swap 버튼이 비활성화된다.
- 기본 선택값은 `English → Korean`이다.

---

## 10. CORS Handling

브라우저에서 HTML이 Ollama API에 직접 요청하므로 CORS 문제가 발생할 수 있다.

Ollama는 기본적으로 `127.0.0.1`과 `0.0.0.0`에서 오는 cross-origin 요청을 허용하지만, HTML을 `file://`로 직접 열거나 다른 origin에서 실행하는 경우 `OLLAMA_ORIGINS` 설정이 필요할 수 있다.

앱에서 CORS/API 연결 오류가 발생하면 다음 도움말을 표시한다.

---

## 11. CORS Help — macOS

화면에 다음과 같은 안내를 제공한다.

### macOS

Terminal에서:

```bash
launchctl setenv OLLAMA_ORIGINS "*"
```

그 다음 Ollama를 완전히 종료하고 다시 실행한다.

Ollama가 macOS 앱으로 실행되는 경우 환경 변수는 `launchctl`을 통해 설정하도록 공식 문서에서 안내한다.

### 권장 실행 방법

HTML을 직접 열어 문제가 발생한다면 해당 폴더에서 간단한 로컬 서버를 실행할 수도 있다.

```bash
python3 -m http.server 8000
```

그리고 브라우저에서:

```text
http://localhost:8000
```

으로 접속한다.

---

## 12. CORS Help — Windows

### Windows

Windows 환경 변수 설정에서 다음 변수를 추가한다.

```text
Variable:
OLLAMA_ORIGINS

Value:
*
```

설정 후 Ollama를 종료하고 다시 실행한다.

또는 PowerShell에서 임시로:

```powershell
$env:OLLAMA_ORIGINS="*"
```

Ollama를 실행한다.

Ollama 공식 문서에서는 Windows에서 사용자 환경 변수에 `OLLAMA_*` 변수를 설정하고 Ollama를 다시 시작하는 방법을 안내한다.

### 로컬 서버 사용

CORS 문제가 계속되는 경우:

```powershell
python -m http.server 8000
```

그리고:

```text
http://localhost:8000
```

에서 HTML을 실행한다.

---

## 13. CORS Error UI

API 호출이 실패하고 CORS 가능성이 있다고 판단되면 번역 영역에 다음과 같은 도움말을 표시한다.

```text
Cannot connect to Ollama.

If Ollama is running but the browser reports a CORS error,
allow this page to access Ollama.

macOS:
launchctl setenv OLLAMA_ORIGINS "*"

Windows:
Set the OLLAMA_ORIGINS environment variable to *

Then restart Ollama.
```

가능하면 오류 원인을 다음 세 가지로 구분한다.

```text
1. Ollama is not running
2. TranslateGemma model is not installed
3. Browser CORS access is blocked
```

---

## 14. Error Handling

다음 오류를 처리한다.

### Ollama 서버가 실행되지 않은 경우

```text
Ollama is not running.
Please start Ollama and try again.
```

### 모델이 없는 경우

Ollama API에서 모델 관련 오류가 발생하면:

```text
TranslateGemma is not available.

Run:

ollama pull translategemma:4b
```

### CORS 오류

```text
Your browser cannot access Ollama because of CORS settings.

See the Windows/macOS setup instructions below.
```

### 기타 API 오류

```text
Translation failed.

[error message]
```

---

## 15. Connection Check

페이지가 로드되면 다음 endpoint를 호출하여 Ollama 연결 상태를 확인한다.

```text
GET http://localhost:11434/api/tags
```

성공하면:

```text
Ollama connected
```

실패하면:

```text
Ollama unavailable
```

가능하다면 `/api/tags` 응답에서 `translategemma` 모델이 존재하는지도 확인한다.

---

## 16. Model Configuration

JavaScript 상단에 설정값을 둔다.

```javascript
const OLLAMA_URL = "http://localhost:11434";
const MODEL = "translategemma:4b";
```

MVP에서는 UI에서 모델을 선택하지 않는다.

사용자가 소스 코드의 다음 값만 수정하면 다른 TranslateGemma 모델을 사용할 수 있도록 한다.

```javascript
const MODEL = "translategemma:12b";
```

TranslateGemma의 Ollama 모델은 4B, 12B, 27B 버전으로 제공된다.

`MODEL`에 지정한 모델이 정확히 설치되어 있지 않으면, 연결 확인 시 `/api/tags` 목록에서 `translategemma` 계열 모델(예: `translategemma:latest`)을 찾아 자동으로 사용한다. 계열 모델도 없으면 설치 명령을 안내한다.

---

## 17. Translation Flow

```text
Page Load
   ↓
Check Ollama
   ↓
Check TranslateGemma
   ↓
User selects languages
   ↓
User enters text
   ↓
Click Translate
   ↓
Validate input
   ↓
Build TranslateGemma prompt
   ↓
POST /api/chat
   ↓
Read message.content
   ↓
Display translation
```

---

## 18. Swap Flow

```text
English → Korean

[English text]
      ↓
[Korean translation]
```

Swap 클릭:

```text
Korean → English

[Korean translation]
      ↓
[English translation]
```

MVP에서는 swap 후 기존 결과를 그대로 새로운 source text로 이동시키고, 사용자가 다시 Translate를 누르도록 한다.

자동 재번역하지 않는다.

---

## 19. File Structure

프로젝트는 최대한 단순하게 유지한다.

```text
project/
└── index.html
```

CSS와 JavaScript도 별도의 파일로 분리하지 않는다.

```html
<style>
  ...
</style>

<script>
  ...
</script>
```

모든 코드를 `index.html` 하나에 포함한다.

---

## 20. No Dependencies

사용하지 않는다.

- React
- Vue
- Svelte
- Tailwind
- Bootstrap
- npm
- Vite
- Webpack
- 외부 CDN
- 외부 번역 API

브라우저의 기본 HTML/CSS/JavaScript만 사용한다.

---

## 21. Privacy

앱 자체는 번역 데이터를 서버에 저장하지 않는다.

번역 요청은 다음 로컬 endpoint로만 전달된다.

```text
http://localhost:11434
```

즉 MVP의 기본 동작에서는 번역 텍스트가 외부 번역 서비스로 전송되지 않는다.

최소한의 디자인을 위해 하단 footer 문구("Translation runs locally through Ollama.")는 제거했다. 로컬 실행 사실은 header 우측의 부제("Local translation powered by Ollama")로 표현한다.

---

## 22. Explicitly Out of Scope

MVP에서는 다음을 구현하지 않는다.

- 번역 기록
- 사용자 계정
- 클라우드 저장
- 파일 번역
- PDF 번역
- 이미지 번역
- 음성 입력
- Text-to-Speech
- 자동 번역
- 번역 품질 평가
- 번역 대안 여러 개 표시
- 모델 선택 UI
- 다크 모드
- 브라우저 확장
- Electron 앱
- PWA
- 서버 backend
- streaming UI

---

## 23. Acceptance Criteria

MVP가 완성되었다고 판단하는 조건:

### 기본 실행

- [ ] `index.html`을 브라우저에서 열 수 있다.
- [ ] 별도의 build 과정이 필요하지 않다.
- [ ] Ollama가 실행 중이면 연결 상태가 표시된다.

### 번역

- [ ] English → Korean 번역이 작동한다.
- [ ] Korean → English 번역이 작동한다.
- [ ] Japanese → Korean 등 다른 선택된 언어 조합도 작동한다.
- [ ] 빈 입력에서는 번역 요청을 보내지 않는다.
- [ ] 번역 중에는 중복 요청을 방지한다.
- [ ] 결과가 translation 영역에 표시된다.

### Ollama

- [ ] `translategemma:4b`를 사용할 수 있다.
- [ ] Ollama가 실행되지 않은 경우 오류 메시지가 표시된다.
- [ ] 모델이 없는 경우 설치 명령을 안내한다.

### CORS

- [ ] CORS/API 접근 오류를 사용자에게 알린다.
- [ ] macOS용 `OLLAMA_ORIGINS` 설정 방법을 표시한다.
- [ ] Windows용 `OLLAMA_ORIGINS` 설정 방법을 표시한다.
- [ ] 필요하면 `python -m http.server` 방식도 안내한다.

### UI

- [ ] Google Translate와 유사한 좌/우 번역 구조를 갖는다.
- [ ] 언어 교환 버튼이 작동한다.
- [ ] desktop/mobile에서 사용할 수 있다.
- [ ] 불필요한 UI 요소가 없다.

---

## 24. Implementation Priority

개발 순서는 다음과 같이 한다.

### Step 1 — Static UI

언어 선택 + textarea + 결과 영역 + Translate 버튼 구현.

### Step 2 — Ollama Connection

`/api/tags` 호출 및 상태 표시.

### Step 3 — Translation

`/api/chat`을 호출하고 `message.content`를 출력.

### Step 4 — Language Prompt

TranslateGemma 공식 prompt 형식을 적용.

### Step 5 — Swap

Source / Target 언어 및 텍스트 교환.

### Step 6 — Error Handling

Ollama 미실행 / 모델 없음 / CORS 오류 처리.

### Step 7 — CORS Help

Windows/macOS별 설정 안내 UI 추가.

### Step 8 — Responsive Polish

최소한의 모바일 대응 및 UI 정리.

---

## 25. MVP Design Principle

이 프로젝트의 핵심은 **"웹앱처럼 보이는 단일 HTML 클라이언트"**다.

따라서 다음 원칙을 유지한다.

> No backend.  
> No build system.  
> No database.  
> No cloud API.  
> Just Browser → Ollama → TranslateGemma.

가능한 한 적은 코드로 구현하고, 기능을 추가하기보다 **로컬 TranslateGemma를 가장 간단하게 사용할 수 있는 인터페이스**를 만드는 것을 MVP의 목표로 한다.