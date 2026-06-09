# 겁쟁이딜러 KPI · 고객 통합 트래커

중고차 딜러십 + 유튜브 채널 운영을 위한 **KPI / 고객(CRM) 관리 도구**입니다.
매일 조회수·구독자 숫자와 고객 정보만 넣으면 주간·월간·KPI(전환율, 매출, 영업이익,
CAC, LTV, 케파 포화도 등)가 자동으로 정리됩니다.

브라우저만 있으면 돌아가는 **단일 HTML 파일**(`index.html`)이라서 별도 서버나
빌드 과정이 필요 없습니다.

---

## 주요 기능

- **대시보드** — 월별 콘텐츠/재무/마케팅 효율 KPI, 조회수·구독자 추이 차트
- **전체표** — 월별 전체 KPI 한눈에 (고정비 자동 적용)
- **일간 입력** — 그날 조회수와 누적 구독자만 입력
- **고객 장부(CRM)** — 등록/착수/입금/판매까지 고객 단위 관리, 연락처 자동 매칭
- **담당자** — 담당자별 진행 상황, 확인 필요 고객 자동 상단 노출
- **설정** — 고정비/변동비/케파/담당자 명단 관리
- **내보내기** — Excel(.xlsx) / CSV 내보내기

---

## 1. 로컬에서 여는 법

가장 간단한 방법은 `index.html`을 더블클릭해서 브라우저로 여는 것입니다.

> 이 상태에서는 데이터가 **이 기기(브라우저)에만** 저장됩니다.
> 화면 우상단에 `○ 이 기기에만 저장` 이라고 표시됩니다.

조금 더 안정적으로 보려면 간단한 로컬 서버로 띄울 수 있습니다.

```powershell
# Python이 있는 경우 (프로젝트 폴더에서)
python -m http.server 8000
# → 브라우저에서 http://localhost:8000 접속
```

---

## 2. Supabase 연결하는 법 (팀 공유 켜기)

데이터를 여러 명이 같이 보려면(직원/담당자 공유) Supabase 무료 DB에 연결합니다.
연결하면 우상단 표시가 `● 공유 연결됨` 으로 바뀌고, 20초마다 자동 동기화됩니다.

### 2-1. Supabase 프로젝트 만들기
1. https://supabase.com 가입 후 **New project** 생성 (무료 플랜으로 충분)
2. 프로젝트가 만들어지면 좌측 메뉴 **Project Settings → API** 로 이동
3. 다음 두 값을 복사해 둡니다.
   - **Project URL** (예: `https://abcd1234.supabase.co`)
   - **anon public** key

### 2-2. 테이블 만들기
좌측 **SQL Editor** 에서 아래 SQL을 실행합니다. (앱은 `app_state` 테이블 한 곳에
전체 상태를 JSON으로 통째로 저장합니다.)

```sql
create table if not exists app_state (
  id text primary key,
  data jsonb,
  updated_at timestamptz default now()
);

-- 데모/팀 내부용: anon 키로 읽고 쓰기 허용
alter table app_state enable row level security;

create policy "allow all on app_state"
  on app_state for all
  using (true) with check (true);
```

> ⚠️ 위 정책은 anon 키만 알면 누구나 읽고 쓸 수 있는 **내부 공유용** 설정입니다.
> 외부에 URL을 공개하지 마세요. 보안이 더 필요하면 Supabase Auth + RLS로 강화하세요.

### 2-3. 코드에 두 줄 넣기
`index.html` 을 열어 아래 부분(현재 비어 있음)에 값을 채웁니다.

```js
/* ===== 공유 DB 설정 — 이 두 줄만 채우면 팀 공유가 켜집니다 ===== */
const SB_URL = "https://abcd1234.supabase.co";   // ← 본인 Project URL
const SB_KEY = "여기에-anon-public-key";          // ← 본인 anon key
```

저장 후 새로고침하면 `● 공유 연결됨` 으로 바뀝니다.
(처음 접속 시 예시 데이터가 보이면 `예시 지우기` 를 눌러 비우세요.)

---

## 3. Vercel 배포하는 법 (공유 링크 만들기)

직원들이 주소만으로 접속할 수 있게 인터넷에 올립니다.

### 방법 A — GitHub 연동 (권장)
1. 이 저장소를 GitHub에 올립니다.
2. https://vercel.com 에 GitHub 계정으로 로그인
3. **Add New → Project → 이 저장소 선택 → Import**
4. 빌드 설정은 건드릴 필요 없습니다(정적 HTML).
   - Framework Preset: **Other**
   - Build/Output 설정 비워둠
5. **Deploy** → 잠시 후 `https://gupjaengi-kpi.vercel.app` 같은 주소가 생성됩니다.
6. 이후 GitHub에 푸시(push)할 때마다 자동으로 재배포됩니다.

### 방법 B — Vercel CLI
```powershell
npm i -g vercel
vercel        # 최초: 로그인 + 프로젝트 설정
vercel --prod # 운영 배포
```

> Supabase `SB_URL` / `SB_KEY` 는 코드(`index.html`)에 직접 적혀 있으므로 별도 환경변수
> 설정 없이 배포만 하면 그대로 공유 DB에 연결됩니다.

---

## 코드 구조 요약 (`index.html` 한 파일)

| 영역 | 위치(대략) | 내용 |
|------|-----------|------|
| 스타일 | 상단 `<style>` | 디자인 토큰, 레이아웃, 컴포넌트 CSS |
| 마크업 | `<div class="wrap">` | 헤더 + 6개 탭 화면(대시보드/전체표/일간/고객/담당자/설정) |
| **공유 DB 설정** | `SB_URL`, `SB_KEY` 두 줄 | **여기만 채우면 Supabase 팀 공유 ON** (비우면 로컬 저장) |
| 저장소 로직 | `load()` / `save()` / `refresh()` | Supabase ↔ 로컬 저장 자동 전환, 20초 동기화 |
| 데이터 모델 | 전역 변수 `S` | `{daily, customers, settings}` 전체 상태 객체 |
| 집계 | `aggregate()` / `monthSlice()` | 일/주/월 단위 KPI 계산 |
| 화면 렌더 | `renderDash/Full/Daily/CRM/Dealer/Set` | 탭별 화면 그리기 |
| 내보내기 | `exportXLSX()` / `exportCSV()` | Excel / CSV 파일 생성 |

### 데이터는 어디에 저장되나?
- **Supabase 연결 시**: `app_state` 테이블의 `id="main"` 한 행에 전체 상태(`S`)를
  JSON으로 통째로 upsert. (`save()` 함수 참고)
- **연결 안 했을 때**: 브라우저 `localStorage` 의 `kkepjangi:v2` 키에 저장.

즉 **데이터 저장의 핵심 스위치는 `SB_URL` / `SB_KEY` 두 줄**입니다.
