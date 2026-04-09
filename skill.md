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
    badges: ['region', 'national', 'med'],
    changes: [{ title, from, to }],
    susi: [{ type, name, count, minReq, method }],
    minScore: [{ name, track, 국, 수, 영, 탐, req, note }],
    jeongsi: [{ track, 국, 수, 영, 탐, 비고 }],
    top10Susi: [...],
    top10Jeongsi: [...],
    enrollment: {...},
    // ... 추가 데이터
  }
}
```

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
