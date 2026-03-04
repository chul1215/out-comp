# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

대전선병원·유성선병원의 2025년 외래환자 데이터를 분석해 2026년 마케팅 타겟을 도출하는 **단일 HTML 파일 대시보드**. 서버·빌드 도구 없이 브라우저에서 직접 실행된다.

- **실행**: `index.html`을 Chrome/Edge로 열면 즉시 동작 (더블클릭)
- **빌드/테스트/린트 명령 없음** — 순수 HTML+CSS+Vanilla JS 단일 파일

## 파일 구조

```
index.html          # 전체 앱 (데이터·라이브러리·로직·UI 모두 포함)
briefing/           # 수동 작성된 마케팅 타겟 브리핑 MD 파일
plan/               # PRD·기획안·와이어프레임 문서
```

## index.html 내부 구조

파일은 약 2,500줄 이상이며, 순서대로 다음으로 구성된다:

1. **인라인 라이브러리** — Chart.js 4.4.0, SheetJS 0.20.1 (CDN 불필요)
2. **`const D25`** (line ~1362) — 2025년 진료실적(`p`)·진단별 통계(`d`) 데이터 JS 객체로 내장. 병원 키: `daejeon` / `yuseong`
3. **상수·매핑** (line ~1367~1660)
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

### D25 진료실적 배열 인덱스

```
r[0]  = 과코드
r[7]  = 신환
r[8]  = 초진
r[9]  = 재진
r[14] = 계(총 환자수)
```

### 제외 진료과 규칙 (`isExcluded`)

| 규칙 | 대상 |
|------|------|
| `DN`으로 시작 | 치과 계열 |
| `HPC` | 건강검진센터 |
| `FM` | 가정의학과 |
| 코드 끝에 `2` | 미수(수납 미완료) 코드 |

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

## 계절/분기 보너스 조정 원칙

보너스 값은 **실제 live raw 점수 기반으로 캘리브레이션**해야 한다. 이론적 값을 넣으면 의도한 순위가 나오지 않을 수 있다. 조정 절차:

1. 화면에서 해당 계절/분기 선택 → 각 진료과의 괄호 안 raw 점수 확인
2. 목표 순위가 되도록 보너스 역산
3. `SEASONAL_BONUS` 또는 `QUARTERLY_BONUS` 업데이트

봄철 대전선병원 기준 실측 raw 점수 (참고): IMG 73.6 / JC 69.4 / ENT 62.5 / NR 55.5 / OS 47.4

## 브리핑 파일

`briefing/` 폴더의 MD 파일은 코드가 자동 생성한 것이 아니라 수동 작성된 자료다. 코드 변경 시 브리핑 파일의 진료과명·순위도 함께 수동으로 일치시켜야 한다.
