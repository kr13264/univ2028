# 대학진학 PreView — 2028 대입전형 시행계획

2028학년도부터 5등급 체제로 전환되며 입시제도가 전면 개편됩니다.
전국 대학의 변경된 전형 정보를 지역별 지도 인터랙션으로 한눈에 정리하는 프로젝트입니다.

---

## 파일 구조

```
sen-landing/
├── index.html              ← 메인: 전국 SVG 지도 + 수능최저 필터 탭
├── style.css               ← 공통 스타일
├── seoul.html              ← 서울 지도 페이지
├── seoul-detail.html       ← 서울 대학 상세 (iframe용)
├── gyeonggi-detail.html    ← 경기·인천 대학 상세
├── gangwon-detail.html     ← 강원 상세
├── chungcheong-detail.html ← 충청·대전·세종 상세
├── gyeongsang-detail.html  ← 경상·대구·부산 상세
├── jeolla-detail.html      ← 전라·광주 상세
├── jeju-detail.html        ← 제주 상세
├── images/enrollment/      ← 모집요강 이미지
├── skill.md                ← 기술 패턴·로직 상세 문서
└── README.md               ← 이 파일
```

---

## 페이지 구조

| 지역 | 진입 방식 | 상세 페이지 |
|------|-----------|-------------|
| **서울** | index.html → seoul.html 이동 | seoul-detail.html |
| **경기·인천** | index.html SVG 줌 | gyeonggi-detail.html |
| 강원~제주 | index.html SVG 줌 | *-detail.html |

> 서울만 대학 수가 많아 별도 HTML로 분리. 나머지는 SVG 줌 + iframe.

---

## 2028 업데이트 현황

| 대학 | 지역 | 날짜 |
|------|------|------|
| 가천대, 아주대, 인천대, 경기대(수원), 한양대(ERICA), 명지대(자연), 한세대 | 경기·인천 | ~ 2026-05-04 |
| 숙명여대, 숭실대, 서울여대, 동덕여대 | 서울 | ~ 2026-05-04 |
| 충북대, 단국대(천안), 상명대(천안) | 충청 | ~ 2026-05-04 |
| 이화여대 | 서울 | 2026-05-05 |
| 단국대(죽전) | 경기 | 2026-05-05 |

---

## 대학 업데이트 체크리스트

새 대학 2028 데이터 추가 시 아래 순서대로 반영:

1. `*-detail.html` — UNI_DETAILS에 데이터 추가 (`badges`에 `'new2028'` 포함)
2. 지도 페이지 (seoul.html 또는 gyeonggi.html 등) — UNI_DETAILS에 요약 데이터 추가
3. index.html — `markers[]`, `universities[]`에 `new2028:true` 추가
4. index.html — `MIN_SCORE_MAP`에 수능최저 정보 추가 (의학 제외)
5. 브라우저 확인 — 지도 N마크, 상세 페이지, 수능최저 탭 필터

> 각 항목의 데이터 형식과 로직 상세는 **skill.md** 참고

---

## 로컬 개발

```bash
cd sen-landing
python3 -m http.server 8080
open http://localhost:8080
```
