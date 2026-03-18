# ✂️ clipstash

> CIME 라이브 스트림 클립 추출 · 전체 영상 다운로드 · 실시간 녹화 서비스

![SvelteKit](https://img.shields.io/badge/SvelteKit_v2-FF3E00?style=flat-square&logo=svelte&logoColor=white)
![Svelte](https://img.shields.io/badge/Svelte_5-FF3E00?style=flat-square&logo=svelte&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS_4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white)
![FFmpeg](https://img.shields.io/badge/FFmpeg_WASM-007808?style=flat-square&logo=ffmpeg&logoColor=white)

<br/>

## 프로젝트 개요

URL을 입력하면 CIME 및 일반 HLS 스트림에서 메타데이터를 추출하고, 브라우저 내에서 영상을 처리합니다.

- **클립 추출** — 시작/종료 시간을 지정해 구간 클립 다운로드
- **전체 영상 다운로드** — VOD 전체를 하나의 MP4 파일로 저장
- **실시간 녹화** — 라이브 스트림 녹화 시작 · 일시정지 · 재개 · 완료

> 모든 영상 처리(세그먼트 다운로드, FFmpeg 인코딩)는 **브라우저 내 FFmpeg WASM**으로 실행됩니다.

<br/>

## 아키텍처

```
┌──────────────────────────────────────────────────────────┐
│                    cime-clip-app                          │
│              SvelteKit v2 + Svelte 5 + TS                │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Browser (Client)                                  │  │
│  │  ┌──────────┐  ┌──────────────┐  ┌─────────────┐  │  │
│  │  │ ClipForm │  │  VideoForm   │  │ RecordForm  │  │  │
│  │  │ 클립 추출 │  │ 전체 다운로드 │  │ 실시간 녹화  │  │  │
│  │  └─────┬────┘  └──────┬───────┘  └──────┬──────┘  │  │
│  │        └──────────────┼─────────────────┘          │  │
│  │                       ▼                            │  │
│  │              ┌─────────────────┐                   │  │
│  │              │  FFmpeg WASM    │                   │  │
│  │              │ 세그먼트 → MP4   │                   │  │
│  │              └─────────────────┘                   │  │
│  └────────────────────────┬───────────────────────────┘  │
│                           │ fetch                        │
│                           ▼                              │
│  ┌────────────────────────────────────────────────────┐  │
│  │  SvelteKit Server Routes (Server-Side)             │  │
│  │  ┌────────────────────┐  ┌─────────────────────┐  │  │
│  │  │ GET /clips/info    │  │ GET /stream/proxy   │  │  │
│  │  │ yt-dlp / 스크래핑   │  │ CORS 프록시          │  │  │
│  │  │ 메타데이터 조회      │  │ HLS 세그먼트 중계     │  │  │
│  │  └────────────────────┘  └─────────────────────┘  │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
        별도 백엔드 서버 없음 · SvelteKit 단일 서버로 구동
```

<br/>

## 기술 스택 — `cime-clip-app`

| 분류 | 기술 |
|------|------|
| Framework | SvelteKit v2, Svelte 5 (runes) |
| Language | TypeScript |
| 스타일링 | Tailwind CSS 4 |
| 빌드 도구 | Vite |
| 영상 처리 | FFmpeg WASM (브라우저 내 인코딩) |
| 스트림 파싱 | HLS m3u8 플레이리스트 파서 |
| 코드 품질 | ESLint, Prettier |
| Server-Side | SvelteKit Server Routes (yt-dlp 메타데이터 조회, CORS 프록시) |

<br/>

## 레포지토리 구성

| 레포지토리 | 설명 | 주요 기술 |
|------------|------|-----------|
| [cime-clip-app](https://github.com/clipstash/cime-clip-app) | 웹 앱 (클라이언트 + 서버 라우트 통합) | SvelteKit · Svelte 5 · TypeScript · FFmpeg WASM |
| [cime-clip-server](https://github.com/clipstash/cime-clip-server) | 백엔드 API 서버 (미사용 · 초기 프로토타입) | FastAPI · Python · yt-dlp |

<br/>

## 동작 방식

1. URL 입력 → SvelteKit 서버 라우트가 yt-dlp / HTML 스크래핑으로 메타데이터(제목, 썸네일, 재생 시간, m3u8 URL) 반환
2. 브라우저에서 m3u8 플레이리스트를 파싱해 세그먼트 목록 획득
3. CORS 우회가 필요한 경우 SvelteKit `/stream/proxy` 서버 라우트를 경유해 세그먼트 다운로드
4. **FFmpeg WASM**이 브라우저 내에서 세그먼트를 합쳐 MP4로 저장
