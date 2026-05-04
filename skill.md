# 페이지 패턴 (Page Patterns)

sen-landing 프로젝트에서 사용되는 UI/UX 패턴을 정리한 문서입니다.

---

## 1. 메인 페이지 구조 (index.html)

### 레이아웃
- **좌측**: 전국 SVG 지도 (`#map-panel`)
- **우측**: 정보 패널 (`#info-panel`) — 지역 정보, 대학 목록, 상세 페이지 표시

### 전국 SVG 지도
- `<svg id="korea-svg">` 에 전국 지도 경로(path)가 포함
- `<g id="uni-markers-overlay">` 에 대학교 마커가 동적으로 렌더링됨
- 기본 viewBox: `0 0 600 820`

---

## 2. 지역 데이터 구조 (REGIONS)

```javascript
const REGIONS = {
  regionKey: {
    name: '지역명',
    zoomViewBox: 'x y width height',   // SVG 줌 타겟 좌표
    detailPage: 'region.html',          // (선택) 상세 페이지 URL
    universities: [
      { name: '대학명', color: '#hex', desc: '설명', tags: ['태그'] }
    ],
    markers: [
      { name: '대학명', x: 숫자, y: 숫자, color: '#hex' }
    ]
  }
}
```

| 필드 | 설명 |
|------|------|
| `zoomViewBox` | 클릭 시 SVG가 애니메이션으로 줌되는 목표 viewBox |
| `detailPage` | 설정 시 마커 클릭 → iframe으로 상세 페이지 로드 |
| `universities` | 우측 패널 목록에 표시되는 대학 정보 |
| `markers` | SVG 지도 위에 렌더링되는 마커 좌표 |

---

## 3. SVG 줌 내비게이션

### 동작 흐름
1. 지역 클릭 → `animateViewBox()` 로 해당 지역까지 부드럽게 줌
2. 줌 완료 → 대학교 마커 렌더링 + 우측 패널에 대학 목록 표시
3. `(<- 전국지도)` 버튼 클릭 → 기본 viewBox로 줌 아웃 + 마커 제거

### 줌 애니메이션
```javascript
function animateViewBox(svg, from, to, duration)
```
- `easeInOutQuad` 이징 적용
- `requestAnimationFrame` 기반 부드러운 전환
- 기본 duration: 600ms

### 예외: 서울
- 서울은 SVG 영역이 ~37×45 단위로 너무 작고 대학이 38개로 많아 줌 방식 부적합
- 서울 클릭 시 → `seoul.html` 로 페이지 이동 (별도 HTML 방식 유지)

---

## 4. SVG 마커 렌더링

### 마커 구성요소
```
<g class="svg-uni-marker">
  ├── <circle>  — 위치 표시 점 (r=1.4)
  ├── <rect>    — 라벨 배경 (hug fit)
  └── <text>    — 대학명 텍스트 (font-size=4.2, x+5 간격)
</g>
```

### getBBox 허그 핏 (Hug Fit)
- 텍스트를 먼저 렌더링한 후 `getBBox()`로 실제 크기를 측정
- `<rect>` 의 x, y, width, height를 측정된 크기 + 패딩(좌우 2, 상하 1.5)으로 설정
- 한글/영문 혼합 텍스트에서도 정확한 크기 보장

### 인터랙션 (CSS)
| 상태 | 배경색 | 테두리 | 텍스트 |
|------|--------|--------|--------|
| 기본 | `#fff` | `#e2e8f0` | `#1e293b` |
| hover | `#eff6ff` | `#93c5fd` | `#2563eb` |
| selected | `#fffbeb` | `#f59e0b` | `#d97706` |

---

## 5. 대학 목록 인터랙션

### 우측 패널 대학 목록
- **마우스 호버** → 좌측 지도에서 해당 대학 마커 하이라이트 (스크롤 이동 없음)
- **마우스 떠남** → 마커 하이라이트 해제
- **클릭** → 상세 페이지(iframe) 열기

### SVG 마커 클릭 (좌측 지도)
- 마커 클릭 → 마커에 `.selected` 클래스 추가 + 상세 페이지 iframe 열기

---

## 6. 상세 페이지 (iframe 패턴)

### 페이지 분리 구조
| 지역 | 지도 페이지 | 상세 전용 페이지 (iframe용) |
|------|------------|---------------------------|
| 서울 | seoul.html | seoul-detail.html |
| 경기·인천 | gyeonggi.html | gyeonggi-detail.html |
| 강원 | — (SVG 줌) | gangwon-detail.html |
| 충청·대전·세종 | — (SVG 줌) | chungcheong-detail.html |
| 경상·대구·부산 | — (SVG 줌) | gyeongsang-detail.html |
| 전라·광주 | — (SVG 줌) | jeolla-detail.html |
| 제주 | — (SVG 줌) | jeju-detail.html |

> 지도 페이지와 상세 페이지를 분리하여 iframe 충돌 방지

### iframe 로드
```
<iframe src="gyeonggi-detail.html?uni=가천대&embed=1">
```

### iframe ↔ 부모 통신
- **목록으로 돌아가기**: `window.parent.postMessage({type:'backToList'}, '*')`
- **부모 수신**: `window.addEventListener('message', ...)` → 패널 복원 + 목록 재렌더링

### 상세 페이지 데이터 구조 (UNI_DETAILS)
```javascript
const UNI_DETAILS = {
  '대학명': {
    fullName: '정식명칭',
    location: '소재지',
    badges: ['region', 'national', 'med', 'new2028'],
    total: { 수시: 000, 정시: 000, 총계: 000 },
    changes: [{ title, from, to }],
    susi: [{ type, name, count, minReq, method }],
    minScore: [{ name, track, 국, 수, 영, 탐, req, note }],
    gradeConversion: { tables: [{ title, subtitle, grades, scores }], note: '' },
    jeongsi: [{ track, 국, 수, 영, 탐, 비고 }],
    jeongsiGradeTable: [{ 영역, 전형명, 반영방법, scores }],
    jeongsiNote: '',
    comprehensiveEval: { title, desc, items: [{ 평가항목, 평가내용, 비율 }] },
    top10Susi: [...],
    top10Jeongsi: [...],
    enrollmentImg: 'images/enrollment/xxx.png' 또는 ['img1.png','img2.png'],
  }
}
```

### badges 값
| 값 | 설명 | 표시 |
|-----|------|------|
| `'seoul'` | 서울권 | 서울권 뱃지 |
| `'gyeonggi'` | 경기·인천권 | 경기·인천권 뱃지 |
| `'national'` | 국립 | 국립 뱃지 |
| `'med'` | 의학계열 | 의학계열 뱃지 |
| `'new2028'` | 2028 주요 변경 | **빨간 N 마크** (목록·지도·상세 3곳) |

---

## 7. 별도 HTML 페이지 패턴 (seoul.html, gyeonggi.html)

### 공통 구조
- 좌측: 지역 상세 지도 이미지 (`#map-panel`)
- 우측: 대학 목록 (`#info-panel`)
- **호버** → 지도 마커 하이라이트 / **클릭** → iframe 상세 페이지

### 마커 클릭 동작 (지도 위 마커)
- 클릭 → iframe으로 `*-detail.html` 로드 (직접 `showUniversityDetail` 호출 X)

### postMessage 리스너
- 각 지도 페이지에 `window.addEventListener('message', ...)` 으로 iframe 복귀 처리

---

## 8. 줌 상태에서의 패널 규칙

| 상태 | 좌측 | 우측 |
|------|------|------|
| 기본 (줌 전) | 전국 SVG 지도 | 지역 소개 + 지도 이미지 |
| 줌 (지역 선택) | 줌된 SVG + 마커 | 대학 목록 (지도 이미지 없음) |
| 줌 + 대학 선택 (detailPage) | 줌된 SVG + 선택 마커 | iframe 상세 페이지 |
| 줌 + 대학 선택 (일반) | 줌된 SVG + 선택 마커 | 목록 내 하이라이트 |

> 줌 상태에서는 우측 패널에 지역 지도 이미지를 표시하지 않음 (좌측 SVG와 중복 방지)

---

## 9. 브레드크럼 내비게이션

```html
<div id="zoom-breadcrumb">
  <button class="bc-btn" onclick="zoomOut()">← 전국지도</button>
</div>
```
- 줌 상태에서만 표시
- 클릭 시 기본 viewBox로 애니메이션 줌 아웃
- 스타일: `padding:9px 22px; border-radius:24px; font-size:18px; font-weight:700`

---

## 10. N 마크 (new2028 뱃지) — 필수 규칙

> **⚠️ 반드시 지켜야 할 규칙**: 대학 데이터에 `badges: ['new2028']`가 포함되면, 아래 **3곳 모두** N마크가 표시되어야 한다. 하나라도 빠지면 안 된다.

### 3곳 필수 표시

| # | 위치 | 파일 | 크기 |
|---|------|------|------|
| ① | **우측 대학 목록 카드** (대학명 옆) | seoul.html, gyeonggi.html, index.html 등 | 16×16px |
| ② | **좌측 지도 마커 라벨** (대학명 옆) | seoul.html, gyeonggi.html, index.html 등 | 14×14px |
| ③ | **상세 페이지 대학명 헤더** | seoul-detail.html, gyeonggi-detail.html 등 | 20×20px |

### 스타일 (공통: 빨간 원형 N)
```html
<!-- ① 목록 카드 (16px) -->
<span style="display:inline-flex;align-items:center;justify-content:center;
  width:16px;height:16px;border-radius:50%;background:#e11d48;color:#fff;
  font-size:9px;font-weight:800;vertical-align:middle;line-height:1">N</span>

<!-- ② 지도 마커 (14px) -->
<span style="display:inline-flex;align-items:center;justify-content:center;
  width:14px;height:14px;border-radius:50%;background:#e11d48;color:#fff;
  font-size:8px;font-weight:800;line-height:1">N</span>

<!-- ③ 상세 헤더 (20px) -->
<span style="display:inline-flex;align-items:center;justify-content:center;
  width:20px;height:20px;border-radius:50%;background:#e11d48;color:#fff;
  font-size:11px;font-weight:800;vertical-align:middle;line-height:1">N</span>
```

### 판단 조건
```javascript
// 목록·마커: UNI_DETAILS에서 조회
const det = UNI_DETAILS[key];
const isNew = det && (det.badges||[]).includes('new2028');

// 상세 헤더: badges 배열에서 직접 판단
(d.badges||[]).includes('new2028')
```

### 데이터 동기화 필수
- **지도 페이지** (seoul.html, gyeonggi.html 등)의 `UNI_DETAILS`
- **상세 페이지** (seoul-detail.html, gyeonggi-detail.html 등)의 `UNI_DETAILS`
- **양쪽 모두** `badges: [..., 'new2028']` 추가해야 3곳 전부 표시됨
- 한쪽만 추가하면 지도/목록은 되는데 상세는 안 되거나, 그 반대가 발생

---

## 11. 글씨 크기 — 필수 규칙

> **⚠️ 반드시 지켜야 할 규칙**: 상세 페이지의 모든 텍스트는 아래 최소 크기를 준수해야 한다. 10px, 11px 사용 금지.

### 최소 글씨 크기 기준

| 용도 | 최소 크기 | 비고 |
|------|----------|------|
| **본문 텍스트** (설명, 내용, 셀 데이터) | **12px** | 테이블 셀, 부제, 주의사항, 비고 등 |
| **제목/라벨** (섹션명, 헤더, 전형명, 평가요소) | **13px** | 테이블 헤더, 항목명, 박스 텍스트 등 |
| **강조 숫자** (점수, 인원, 등급) | **13px** | 환산점수, 모집인원 등 |

### 적용 대상
- `*-detail.html` 상세 페이지의 모든 렌더링 코드
- 학생부교과 반영방법 (`gradeConversion`) 섹션
- 학생부종합 평가방법 (`comprehensiveEval`) 섹션
- 수시 전형방법, 최저학력기준, 정시 반영비율 등 모든 테이블

### 금지 사항
- `font-size:10px` → 최소 `12px`로 변경
- `font-size:11px` → 최소 `12px`로 변경 (라벨이면 `13px`)
- `color:#94a3b8` (너무 연한 회색) → `#64748b` 이상으로 진하게

---

## 12. enrollmentImg (전형별 모집인원 이미지)

> 대학별 전형별 모집인원 표 이미지를 클릭-줌 방식으로 표시하는 섹션

### 데이터 형식
```javascript
// 단일 이미지
enrollmentImg: 'images/enrollment/dongduck.png'

// 복수 이미지 (배열)
enrollmentImg: ['images/enrollment/myongji-susi-2.png', 'images/enrollment/myongji-susi-1.png']
```

### 이미지 파일 위치
`images/enrollment/` 폴더

### 현재 등록된 이미지
| 대학 | 파일명 | 지역 |
|------|--------|------|
| 경기대(수원) | kyonggi-susi-1.png, kyonggi-susi-2.png | 경기 |
| 한양대(ERICA) | hanyang-erica-susi.png, hanyang-erica-jeongsi.png | 경기 |
| 명지대(자연) | myongji-susi-2.png(인문), myongji-susi-1.png(자연) | 경기 |
| 동덕여대 | dongduck.png | 서울 |
| 서울여대 | seoulWoman.png | 서울 |
| 상명대(천안) | sangmung.png | 충청 |

### 렌더링 코드 (클릭 줌 인/아웃)
```javascript
${d.enrollmentImg ? `
<div class="detail-section">
  <div class="detail-section-title" style="--accent:#6366f1">전형별 모집인원</div>
  <div style="overflow-x:auto;margin:0 -20px;padding:0 20px">
    ${(Array.isArray(d.enrollmentImg) ? d.enrollmentImg : [d.enrollmentImg]).map(img => `
      <img src="${img}" ... onclick="this.style.maxWidth=this.style.maxWidth==='100%'?'none':'100%'">
    `).join('')}
  </div>
  <div style="font-size:12px;color:#64748b;margin-top:4px">※ 이미지를 클릭하면 확대/축소됩니다</div>
</div>` : ''}
```

### 데이터 동기화 필수
- **지도 페이지** (seoul.html, gyeonggi.html)와 **상세 페이지** (*-detail.html) **양쪽 모두** `enrollmentImg` 필드 추가 필요
- SVG 줌 방식 지역(충청, 강원 등)은 detail 파일만 수정하면 됨

### 주의사항
- seoul-detail.html에서는 `secNum`/`nextSec()` 자동번호와 연동 — `enrollImgHtml` 선언은 반드시 `secNum` 선언 이후에 위치
- 이미지 순서: 배열 순서대로 표시됨 (예: 인문→자연 순)

---

## 13. index.html의 new2028 마커 플래그

> SVG 줌 방식 지역(충청, 강원, 경상, 전라, 제주)에서 N마크를 표시하려면 index.html의 markers와 universities 배열 **양쪽 모두**에 `new2028:true` 추가 필요

```javascript
// markers 배열
{name:'상명대(천안)', x:410, y:348, color:'#374151', new2028:true},

// universities 배열
{name:'상명대(천안)', location:'충남 천안', national:false, tags:['수시','정시'], new2028:true},
```

- markers의 `new2028` → 지도 마커 라벨 옆 N 뱃지
- universities의 `new2028` → 우측 대학 목록 카드 옆 N 뱃지
- detail 파일의 `badges: ['new2028']` → 상세 페이지 헤더 N 뱃지
