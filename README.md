# 🎪 페스티벌 스탬프 투어 시스템 (Festival Stamp Tour)

학교·동아리·기관 축제 및 행사를 위한 **부스 스캔 기반 실시간 QR 스탬프 투어 & 통합 관제 플랫폼**입니다.  
Custom Stickerbook UI(`styles.css`), Vanilla JS, 그리고 Supabase(PostgreSQL, Auth, Realtime, RPC, RLS)를 기반으로 구축되었으며, **Supabase Free Tier 한도 내에서 대규모 동시 접속을 안정적으로 처리**하도록 최적화되어 있습니다.

---

## 🌟 주요 특징 및 아키텍처

### 1. 부스 스캔 중심 적립 엔진 (학생 카메라 권한 불필요)
* **기존 방식의 문제 해결:** 수백 명의 학생이 동시에 카메라를 켜거나 인앱 브라우저(카카오톡, 인스타그램 등) 권한 차단으로 스캔에 실패하는 문제를 해결하기 위해 **부스 운영진 스캔 방식**을 채택했습니다.
* **초경량 Auth & Rate Limit 방어:** 학생 로그인 시 불필요한 Supabase Auth 가입 대신 `register_student` RPC로 즉시 등록하며, 학생 화면의 Realtime 상시 구독을 제거하여 Free Tier 동시 접속 한도를 극대화했습니다.

### 2. 3계층 역할 분리 시스템
* **L1 (학생 도장판 — `index.html` / `student.js`)**
  - **내 스탬프 QR 제시:** 카메라를 켤 필요 없이 버튼 클릭 한 번으로 학생 고유 QR(`STAMP:학번`) 제시
  - **1.5초 스마트 감지 폴링:** 부스 운영진이 스캔하는 즉시 학생 화면에서 성공 사운드·진동·컨페티 축하 연출과 함께 도장판 자동 반영
  - **3단 탭 메뉴:** 도장판(부스별 스티커 카드 및 위치 뱃지) / 학생 랭킹(TOP 20 + 시상대) / 동아리 랭킹(인기 부스 + 시상대)
  - **PWA & 인앱 브라우저 탈출 (`pwa-ui.js`, `webview-checker.js`):** 홈 화면 앱 설치 및 카카오톡/인스타 인앱 브라우저 감지 시 외부 브라우저 이동 안내
* **L2 (부스 운영진 콘솔 — `booth.html` / `booth.js`)**
  - **2단 지급 모드:** ① **카메라 스캔 모드**(연속 1초 QR 인식) / ② **학번 직접 입력 모드**(카메라 인식이 어렵거나 빛 반사가 심할 때 즉시 수동 지급)
  - 누적 적립 학생 수 카운트 및 방금 적립된 학생 실시간 피드(최신 5건)
  - **단일 기기 세션 제어 (1계정 1기기):** 다른 기기에서 로그인 시 기존 세션 자동 종료
* **L3 (총괄 관리자 관제탑 — `admin.html` / `admin.js`)**
  - 데스크톱/모바일 반응형 통합 관제 화면
  - 실시간 지표 메트릭(총 도장 적립, 등록 부스, 참여 학생, 인증 실패 기록) 및 부스별 적립 현황
  - 최근 인증 블랙박스 로그 스트리밍 및 전체 학생 랭킹 실시간 검색
* **통합 보안 로그인 포털 (`portal.html` / `portal.js`)**
  - L2(부스 담당자) / L3(총괄 관리자) 권한 선택 및 보안 세션 인증

### 3. `config.js` 단일 파일 100% 동적 커스터마이징
별도의 빌드 도구나 소스코드 수정 없이, 저장소를 포크한 뒤 **[`config.js`](./config.js) 단 하나의 파일만 수정**하면 다음 항목이 모든 화면과 검증 로직에 100% 자동 반영됩니다:
- Supabase 프로젝트 연결 정보 (`supabase.url`, `supabase.anonKey`)
- 학번 형식/자릿수 정규식 및 안내 문구 (`auth.studentIdPattern`, `auth.domain`)
- 브라우저 탭 제목, 행사 명칭, 히어로 배너, 3단 탭 이름, 완주 축하 모달, 크레딧 문구
- 부스별 Lucide 아이콘 오버라이드 (`boothIcons`)

---

## 📁 폴더 및 파일 구조

```text
stamp-tour/
├── assets/
│   ├── icons/              # Favicon, Apple Touch Icon, PWA 192x192 / 512x512 아이콘
│   └── vendor/             # 오프라인/로컬 호환 Lucide 아이콘 라이브러리 및 라이선스
├── migrations/             # 점진적 DB 마이그레이션 SQL (랭킹/시상대 등)
├── tests/                  # 통합 테스트 및 SQLite 기반 랭킹 쿼리 단위 테스트
├── config.js               # ★ Supabase 키, 학번 규칙, 행사 명칭/문구, 아이콘 단일 설정 파일
├── supabase-init.js        # config.js 기반 Supabase 클라이언트 초기화
├── app-service.js          # Supabase RPC/Auth/Realtime 연동 서비스 레이어
├── ui-shared.js            # 공통 Stickerbook UI 컴포넌트, 모달, 시상대, 탭 타이틀 동기화
├── styles.css              # Custom Stickerbook 디자인 시스템 및 반응형 레이아웃
├── pwa-ui.js               # PWA 설치 유도 및 서비스 워커 연동
├── webview-checker.js      # 인앱 브라우저 감지 및 외부 브라우저 탈출 안내
├── index.html / student.js # [L1] 학생용 도장판, QR 제시, 학생/동아리 랭킹
├── booth.html / booth.js   # [L2] 부스 운영진 콘솔 (카메라 QR 스캐너 & 학번 직접 입력)
├── admin.html / admin.js   # [L3] 총괄 관리자 실시간 관제탑 & 전체 랭킹 검색
├── portal.html / portal.js # [공통] L2/L3 통합 로그인 포털
├── schema.sql              # Supabase 전체 DB 스키마 (테이블, RPC, RLS, Realtime, 시드)
├── manifest.json           # PWA 웹앱 매니페스트
└── service-worker.js       # PWA 서비스 워커
```

---

## 🗄️ 데이터베이스 스키마 명세 (`schema.sql`)

### 1. 주요 테이블
| 테이블명 | 설명 | 주요 컬럼 |
| :--- | :--- | :--- |
| `clubs` | 동아리/부스 기본 정보 | `club_id` (PK), `name`, `location`, `created_at` |
| `users` | 학생 및 운영진 프로필·권한 | `id` (PK), `student_id` (Unique/Email), `name`, `role` (`L1`/`L2`/`L3`), `is_approved`, `club_id`, `active_session_id` |
| `clubs_status` | 부스 상태 및 비상 코드 정보 | `club_id` (PK/FK), `current_otp`, `otp_expires_at`, `last_login_at` |
| `stamps` | 학생별 부스 스탬프 적립 내역 | `id` (PK), `student_id`, `club_id` (FK), `created_at`, Unique(`student_id`, `club_id`) |
| `stamp_logs` | 실시간 인증 시도 감사 로그 | `id` (PK), `student_id`, `club_id`, `status`, `created_at` |

### 2. 핵심 RPC 함수 (PostgreSQL `SECURITY DEFINER`)
* `register_student(p_student_id, p_name)`: Auth Rate Limit 없이 초경량 학생 세션 등록/갱신
* `booth_grant_stamp(p_student_id)`: 부스 운영진(L2/L3) 권한 검증 후 학생 QR(`STAMP:학번`) 또는 학번에 대해 원자적 스탬프 지급 및 중복 차단
* `get_student_stamps(p_student_id)`: 학생 본인의 도장 적립 내역 조회
* `get_stamp_leaderboard(p_limit)` / `get_my_rank(p_student_id)`: 개인정보가 마스킹(`201**`, `홍*동`)된 실시간 학생 랭킹 및 본인 순위 조회
* `get_club_leaderboard()` / `get_stamp_podium()`: 동아리별 방문 학생 수 공동 순위 및 학생 1~3위 시상대 집계

---

## 🚀 포크 후 배포 및 새 Supabase 이전 가이드

이 저장소를 다른 GitHub 조직으로 포크(Fork)하여 새로운 행사에 투입할 때는 아래 순서만 진행하면 됩니다.

### 1. 새 Supabase 프로젝트 세팅
1. [Supabase](https://supabase.com)에서 새 프로젝트를 생성합니다.
2. 대시보드의 **SQL Editor**에 [`schema.sql`](./schema.sql) 전체 내용을 복사해 붙여넣고 **RUN**을 실행합니다.
   - 필요에 따라 파일 하단의 `INSERT INTO public.clubs` 시드 구문을 실제 행사 부스 목록으로 수정한 뒤 실행하세요.
3. **운영진(L2) 및 관리자(L3) 계정 생성:**
   - `Authentication -> Users -> Add user`에서 계정을 생성합니다 (예: 총괄 관리자 `admin@festival.com`, `teacher1@festival.com` 등 여러 관리자 계정 생성 가능, 부스 `booth-01@festival.com`).
   - `Table Editor -> public.users` 테이블에서 관리자 계정의 `role`을 `'L3'`로, 각 부스 계정의 `role`을 `'L2'` 및 `club_id`를 담당 부스 ID(예: `'booth-01'`)로 지정합니다 (`is_approved = true`).
   - 총괄 관리자 로그인 화면에서는 기본 `admin` 외에도 등록된 관리자 ID(`teacher1` 등)를 입력해 여러 관리자가 서로 튕기지 않고 동시 접속할 수 있습니다.

### 2. `config.js` 설정 수정
[`config.js`](./config.js) 상단의 설정값을 본인 환경에 맞게 수정합니다:
```javascript
window.APP_CONFIG = {
  supabase: {
    url: "https://your-new-project.supabase.co/",
    anonKey: "your-new-publishable-anon-key"
  },
  auth: {
    domain: "@festival.com",
    studentIdPattern: "^\\d{4,8}$", // 학교 학번 자릿수/규칙에 맞게 조정
    studentIdPlaceholder: "학번을 입력하세요 (예: 20101)",
    studentIdHelp: "4~8자리 숫자 학번"
  },
  festival: {
    title: "우리 학교 동아리 발표회",
    ...
  }
};
```
*(선택 사항: 앱 아이콘과 홈 화면 설치 이름을 바꾸려면 `manifest.json`과 `assets/icons/` 이미지를 교체하세요.)*

### 3. 배포 및 테스트
별도의 빌드 과정 없이 **GitHub Pages, Cloudflare Pages, Vercel, Netlify** 등 정적 호스팅에 연결하면 즉시 서비스됩니다.

```bash
# 로컬 정적 서버 실행 예시
npx -y serve ./

# SQLite 기반 랭킹 쿼리 및 스키마 일관성 테스트 실행
python tests/ranking_queries.py
```

---

## 🔒 라이선스
MIT License. 자유롭게 포크, 수정 및 배포할 수 있습니다.
