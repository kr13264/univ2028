# 쎈(SEN)진학 PreView — 2028 대입전형 시행계획

전국 대학교 입시 정보를 지역별 지도 인터랙션으로 제공하는 웹 프로젝트.

---

## 파일 구조

```
sen-landing/
├── index.html              ← 메인: 전국 SVG 지도 + 지역 줌
├── style.css               ← 공통 스타일
│
├── seoul.html              ← 서울 지도 페이지 (별도 HTML)
├── seoul-detail.html       ← 서울 대학 상세 (iframe용)
│
├── gyeonggi.html           ← (레거시, 미사용) 경기·인천 별도 지도 페이지
├── gyeonggi-detail.html    ← 경기·인천 대학 상세 (iframe용)
│
├── gangwon-detail.html     ← 강원 상세 (index.html SVG 줌에서 iframe)
├── chungcheong-detail.html ← 충청·대전·세종 상세
├── gyeongsang-detail.html  ← 경상·대구·부산 상세
├── jeolla-detail.html      ← 전라·광주 상세
├── jeju-detail.html        ← 제주 상세
│
├── images/
│   ├── seoul-map.png       ← 서울 지도 이미지 (seoul.html용)
│   ├── metro-map.png       ← (구) 수도권 지도 — 사용하지 않음
│   └── korea-*.png         ← 전국지도 타일
│
├── skill.md                ← 기술 패턴 문서
└── README.md               ← 이 파일
```

---

## 지역별 페이지 구조

| 지역 | 진입 방식 | 지도 페이지 | 상세 페이지 | 지도 소스 |
|------|-----------|-------------|-------------|-----------|
| **서울** | index.html → seoul.html 이동 | seoul.html | seoul-detail.html | seoul-map.png (별도) |
| **경기·인천** | index.html SVG 줌 | — | gyeonggi-detail.html | SVG 줌 |
| 강원 | index.html SVG 줌 | — | gangwon-detail.html | SVG 줌 |
| 충청·대전·세종 | index.html SVG 줌 | — | chungcheong-detail.html | SVG 줌 |
| 경상·대구·부산 | index.html SVG 줌 | — | gyeongsang-detail.html | SVG 줌 |
| 전라·광주 | index.html SVG 줌 | — | jeolla-detail.html | SVG 줌 |
| 제주 | index.html SVG 줌 | — | jeju-detail.html | SVG 줌 |

> 서울만 대학 수가 많아 별도 HTML(seoul.html)로 분리.
> 경기·인천 포함 나머지 지역은 index.html SVG 줌 → *-detail.html iframe 방식.
> gyeonggi.html은 레거시 (현재 미사용).

---

## 지도 시스템

### 메인 지도 (index.html)
- `<svg id="korea-svg" viewBox="0 0 1000 1000">` 에 전국 SVG path 포함
- 지역 클릭 → `zoomToRegion()` → `animateViewBox()` 로 부드럽게 줌
- 줌 완료 후 `REGIONS[key].markers[]` 좌표에 대학 마커 렌더링
- 경기·인천 zoomViewBox: `'270 100 300 310'`

### 마커 좌표 추가/수정 시
- index.html `REGIONS.gyeonggi.markers[]` 에 `{name, x, y, color}` 추가
- 마커의 `name`이 `gyeonggi-detail.html` UNI_DETAILS의 키와 일치해야 함
- 마커 클릭 → `gyeonggi-detail.html?uni=대학명&embed=1` iframe 자동 로드

---

## 대학 데이터 추가 가이드

### 1. 상세 데이터 (*-detail.html)
각 지역의 detail 파일 안의 `UNI_DETAILS` 객체에 대학 데이터 추가:

```javascript
'대학명': {
  fullName: '정식명칭',
  location: '소재지',
  badges: ['gyeonggi', 'new2028'],  // ← 2028 변경 시 'new2028' 추가
  total: { 수시: 000, 정시: 000, 총계: 000 },
  changes: [{ title, from, to }],
  susi: [{ type, name, count, minReq, method }],
  minScore: [{ name, track, 국, 수, 영, 탐, req, note }],
  gradeConversion: { tables: [...], note: '' },
  jeongsi: [{ track, 국, 수, 영, 탐, 비고 }],
  jeongsiGradeTable: [...],
  jeongsiNote: '',
}
```

### 2. 지도 마커 (index.html)
- `REGIONS.gyeonggi.markers[]` 에 `{name, x, y, color}` 추가
- `name`이 `gyeonggi-detail.html` UNI_DETAILS의 키와 일치해야 함

---

## N 마크 (new2028 뱃지) 시스템

2028학년도 시행계획에서 주요 변경이 있는 대학에 **빨간 N 마크**를 표시.

### 적용 조건
- `*-detail.html` UNI_DETAILS의 `badges` 배열에 `'new2028'` 포함 시 자동 표시

### N 마크가 표시되는 위치

| 위치 | 렌더링 | 파일 |
|------|--------|------|
| **상세 페이지 헤더** | `<span class="detail-badge badge-new2028">N 2028</span>` | *-detail.html |

### 체크리스트: 대학에 N 마크 추가 시
1. `*-detail.html` UNI_DETAILS → badges에 `'new2028'` 추가
2. 브라우저에서 상세 페이지 열어 N 2028 뱃지 확인

---

## 대학별 데이터 입력 체크리스트 (명지대 기준)

대학 하나를 추가할 때 아래 항목을 순서대로 채움.
PDF 시행계획서 기준 페이지를 참고하여 작성.

### 기본 정보
- [ ] `fullName` — 정식 대학명 (예: 명지대학교 (자연캠퍼스))
- [ ] `location` — 소재지 (예: 경기 용인)
- [ ] `badges` — 지역 + 해당 시 `new2028` (예: `['gyeonggi','new2028']`)
- [ ] `total` — 모집인원 총계 (예: `{ 수시: 2027, 정시: 881, 총계: 2908 }`)

### 1. 전년도 대비 변경사항 (`changes`)
```javascript
changes: [
  { title: '변경 제목', from: '2027년(전년)', to: '2028년(올해)' },
]
```
주요 체크 항목:
- [ ] 전형방법 변경 (교과비율, 출결 반영 등)
- [ ] 단계별 배수 변경
- [ ] 면접 반영비율 변경
- [ ] 모집인원 증감 (대폭 변동 위주)
- [ ] 수능최저학력기준 신설/변경
- [ ] 정시 반영방법 변경
- [ ] 가산점 신설/변경

### 2. 수시모집 (`susi`)
```javascript
susi: [
  { type:'교과|종합|논술|실기', name:'전형명', count:인원, minReq:'●|△|—', method:'전형방법' },
]
```
- [ ] 학생부교과 전형들 (학교장추천, 교과면접, 기회균형 등)
- [ ] 학생부종합 전형들 (서류형, 면접형, 특기자 등)
- [ ] 논술전형 (있는 경우)
- [ ] 실기/실적 전형들
- `minReq`: ● 수능최저 있음 / △ 일부만 / — 없음

### 3. 수시 수능최저학력기준 (`minScore`)
```javascript
minScore: [
  { name:'전형명', track:'계열', 국:'○', 수:'○', 영:'○', 탐:'상위1', req:'2합6', note:'' },
]
```
- [ ] 전형별 × 계열별 조합 (인문/자연/의약학 등)
- [ ] 탐구 반영 방식 (상위1과목, 2과목 평균 등)
- [ ] 특수 조건 (note: 과탐 필수 등)

### 4. 수시 학생부 반영방법 (`gradeConversion`)
```javascript
gradeConversion: {
  tables: [
    { title: '구분명', subtitle: '반영교과·학기', grades: [1~9], scores: [점수] },
  ],
  note: '산출식 등 보충 설명'
}
```
- [ ] 졸업예정자(재학생) 등급별 환산점수 테이블
- [ ] 이전 졸업자 등급별 환산점수 테이블 (다를 경우)
- [ ] 반영교과 (인문/자연/예체능 구분)
- [ ] 반영학기 (3학년 1학기까지 등)
- [ ] 성취평가(A~E) 환산 (있는 경우 `extraLabel`, `extraGrades`, `extraScores`)

### 5. 학생부종합 평가방법 (`comprehensiveEval`) — 해당 시
```javascript
comprehensiveEval: {
  target: '적용 전형 목록',
  period: '반영 기간',
  method: '평가 방식 설명',
  criteria: [
    { 평가요소:'학업역량', 평가항목:'학업성취도', 내용:'...' },
  ],
  notes: ['유의사항']
}
```
- [ ] 적용 대상 전형
- [ ] 평가요소별 항목 (학업역량, 진로역량, 공동체역량 등)
- [ ] 학교폭력 반영 등 유의사항

### 6. 정시모집 (`jeongsi`)
```javascript
jeongsi: [
  { track:'계열', 국:비율, 수:비율, 영:비율, 탐:비율, 비고:'가산점 등' },
]
```
- [ ] 계열별 수능 반영비율 (인문/자연/예체능 등)
- [ ] 경영대학 등 별도 비율 있으면 분리
- [ ] 가산점 정보 (과탐/사탐 가산 등)

### 7. 정시 영어/한국사 등급 환산 (`jeongsiGradeTable`)
```javascript
jeongsiGradeTable: [
  { 영역:'영어', 전형명:'일반전형', 반영방법:'등급별 점수', scores:[1등급~9등급] },
  { 영역:'한국사', 전형명:'일반전형', 반영방법:'가산점', scores:[1등급~9등급] },
]
```
- [ ] 영어 등급별 환산점수 (1~9등급)
- [ ] 한국사 가산점 (1~9등급)

### 8. 정시 보충 (`jeongsiNote`)
- [ ] 전형방법 요약 (수능 비율 + 기타)
- [ ] 모집인원, 군 정보
- [ ] 수능 활용지표 (백분위/표준점수)

### 9. 반영 확인
- [ ] `*-detail.html` UNI_DETAILS에 전체 데이터 입력
- [ ] index.html `REGIONS` markers에 마커 좌표 추가 (name 일치 확인)
- [ ] `new2028` 뱃지 추가 (해당 시)
- [ ] 브라우저에서 지도 마커 클릭 → 상세 페이지 정상 로드 확인

---

## 로컬 개발

```bash
# 로컬 서버 실행 (포트 8080)
cd sen-landing
python3 -m http.server 8080

# 접속
open http://localhost:8080
```
