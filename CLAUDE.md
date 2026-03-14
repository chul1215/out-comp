# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

대전선병원·유성선병원의 2025년 외래환자 데이터를 분석해 2026년 마케팅 타겟을 도출하는 **단일 HTML 파일 대시보드**. 서버·빌드 도구 없이 브라우저에서 직접 실행된다.

- **실행**: `index.html`을 Chrome/Edge로 열면 즉시 동작 (더블클릭)
- **빌드/테스트/린트 명령 없음** — 순수 HTML+CSS+Vanilla JS 단일 파일
- **GitHub Pages**: `main` 브랜치 root(`/`) 서빙 중

## 파일 구조

```
index.html          # 전체 앱 (데이터·라이브러리·로직·UI 모두 포함, ~2,790줄)
briefing/           # 수동 작성된 마케팅 타겟 브리핑 MD 파일
plan/               # PRD·기획안·와이어프레임 문서
25 data/            # 원본 엑셀 데이터 (.gitignore — 커밋 금지)
chartjs.min.js      # 라이브러리 원본 (.gitignore — index.html에 인라인 내장됨)
xlsx.min.js         # 라이브러리 원본 (.gitignore)
data_2025.js        # 원본 추출 데이터 (.gitignore)
data_2025_min.js    # 최소화 데이터 (.gitignore)
```

## index.html 내부 구조 (라인 기준)

파일은 ~2,790줄이며, 순서대로 구성된다:

1. **인라인 라이브러리** (line 1~) — Chart.js 4.4.0, SheetJS 0.20.1 (CDN 불필요)
2. **`const D25`** (line ~1362) — 2025년 진료실적(`p`)·진단별 통계(`d`) 데이터 JS 객체. 병원 키: `daejeon` / `yuseong`
3. **상수·매핑** (line ~1367~1700)
   - `MONTHS`, `QTR`, `SEASON` — 기간 분류 정의
   - `SEASONAL_BONUS` — 계절별 마케팅 타겟 전략 가중치 (봄/여름/가을/겨울)
   - `QUARTERLY_BONUS` — 분기별 전략 가중치 (Q1~Q4)
   - `DEPT_MAP` — 진료과 코드 → 한글명 매핑
   - `DIAG_MAP` — ICD 진단코드 → 한글 진단명 매핑
4. **`const App`** (line ~1703) — 전역 상태 (선택된 병원, 기간 모드, 가중치 슬라이더 값 등)
5. **데이터 접근 함수** — `getSource()`, `getPerfMonthly()`, `getDiagMonthly()`
6. **집계 함수** — `aggPerf(months)`, `aggDiag(months)`
7. **점수 계산** — `calcComposite()`, `calcDiagComposite()`, `getSeasonalBonus()`
8. **렌더 함수** — `renderHome()`, `renderTrend()`, `renderQuarter()`, `renderTarget()`, `renderYoY()`, `renderDiag()`

## 핵심 데이터 구조

### D25 / data26 진료실적 배열 인덱스

```
r[0]  = 과코드
r[1]  = 일반
r[2]  = 보험
r[3]  = 보호(의료급여)
r[4]  = 자보
r[5]  = 산재
r[6]  = 기타
r[7]  = 신환
r[8]  = 초진
r[9]  = 재진
r[10] = 남자
r[11] = 여자
r[12] = 선택진료
r[13] = 일반진료
r[14] = 계(총 환자수)
```

### D25 진단별 통계 배열 인덱스

```
r[0] = 과코드
r[1] = 입력자명(의사명)
r[2] = 진단코드
r[3] = 진단명
r[4] = 건수
```

### App 전역 상태 주요 필드

```javascript
App.hospital          // 'daejeon' | 'yuseong'
App.year              // '2025' | '2026'
App.has26             // 26년 데이터 업로드 여부 (YoY탭 활성화 조건)
App.data26            // { daejeon: { p: {}, d: {} }, yuseong: { p: {}, d: {} } }
App.weights           // { growth: 40, volume: 30, newpt: 30 }
App.targetPeriodMode  // 'all' | 'quarterly' | 'seasonal'
App.targetPeriod      // null | 'Q1'~'Q4' | '봄'~'겨울'
```

### 제외 진료과 규칙 (`isExcluded`)

| 규칙 | 대상 |
|------|------|
| `DN`으로 시작 | 치과 계열 |
| `HPC` | 건강검진센터 |
| `FM` | 가정의학과 |
| 코드 끝에 `2` | 미수(수납 미완료) 코드 |
| `AER` | 응급실 |

## 복합점수 산출 로직

```
Composite = 0.40 × Growth_Score + 0.30 × Volume_Score + 0.30 × NewPatient_Score
(각 점수는 min-max 정규화 후 100점 만점)
```

계절/분기 선택 시 `SEASONAL_BONUS` 또는 `QUARTERLY_BONUS`에 정의된 가중치가 raw 점수에 더해져 최종 순위가 결정된다. 조정된 점수는 UI에 `조정값 (raw값)` 형식으로 표시된다.

## 진료과 코드 주의사항

| 코드 | 진료과 |
|------|--------|
| `NR` | 신경과 |
| `NS` | 신경외과 |
| `NP` | 정신건강의학과 |
| `IMC` | 심장내과 |
| `IML` | 내과(호흡기) |
| `SC` | 척추센터 |
| `JC` | 관절센터 |

## 26년 데이터 업로드 흐름

- 사용자가 진료실적/진단통계 엑셀 파일을 업로드하면 SheetJS로 파싱 → `App.data26`에 저장
- `App.has26 = true`가 되면 YoY 비교 탭이 활성화됨
- `getSource()`가 `App.year === '2025'`이면 `D25`, 아니면 `App.data26`를 반환
- 엑셀 파싱 시 macOS NFD 유니코드로 인한 파일명 문자 깨짐에 주의

## 계절/분기 보너스 조정 원칙

보너스 값은 **실제 live raw 점수 기반으로 캘리브레이션**해야 한다. 이론적 값을 넣으면 의도한 순위가 나오지 않을 수 있다. 조정 절차:

1. 화면에서 해당 계절/분기 선택 → 각 진료과의 괄호 안 raw 점수 확인
2. 목표 순위가 되도록 보너스 역산
3. `SEASONAL_BONUS` 또는 `QUARTERLY_BONUS` 업데이트

봄철 대전선병원 기준 실측 raw 점수 (참고): IMG 73.6 / JC 69.4 / ENT 62.5 / NR 55.5 / OS 47.4

## 브리핑 파일

`briefing/` 폴더의 MD 파일은 코드가 자동 생성한 것이 아니라 수동 작성된 자료다. 코드 변경 시 브리핑 파일의 진료과명·순위도 함께 수동으로 일치시켜야 한다.
