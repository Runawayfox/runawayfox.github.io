---
title: "우주 속 여우의 은신처를 짓다 — Kitsune Hideout 제작기"
description: "Astro v7, Canvas 2D 애니메이션, CSS polychrome shimmer까지 — 이 블로그를 설계하고 구현한 과정을 처음부터 끝까지 정리합니다."
pubDate: 2026-10-13T15:00:00+09:00
tags: ["astro", "web", "blog", "canvas", "css", "personal"]
---

직접 만든 블로그에 글을 쓰는 건 조금 이상한 일이다. 마치 목수가 자기 작업실을 짓고 나서 그 작업실에 대한 글을 그 작업실 책상에 앉아 쓰는 것 같은 기묘한 자기참조. 하지만 이 기록을 남기지 않으면 분명 나중에 "내가 왜 이렇게 했더라?" 하고 머리를 긁게 될 것이다. 그러니 써두기로 했다.

---

## 왜 직접 만들었나

처음부터 직접 만들겠다고 작정한 건 아니었다. Velog, Tistory, Notion 공개 페이지 같은 선택지들을 다 고려했다. 결국 직접 만들기로 한 이유는 단순했다. *내 취향을 완전히 반영하고 싶었다.*

블로그가 글만 담는 공간이 되는 게 싫었다. 사이트 자체가 하나의 작품 — 방문하면 잠깐 멈추게 되는 무언가 — 이 되었으면 했다. 그리고 마침 Astro가 흥미로웠다.

---

## 기술 스택

복잡하지 않다.

- **Astro v7** — 정적 사이트 생성기. 아일랜드 아키텍처 덕분에 필요한 곳에만 JS가 들어간다.
- **Tailwind CSS v4** — `@tailwindcss/vite` 플러그인 방식. `tailwind.config.js` 없이 Vite 플러그인 하나로 끝난다.
- **Pagefind** — 빌드 후 정적 인덱스를 생성하는 검색 라이브러리. 서버 없이 전문 검색이 된다.
- **GitHub Pages + Actions** — `master` 푸시 → 자동 빌드 → 배포.

프레임워크를 최대한 얇게 썼다. 핵심 로직은 대부분 `<canvas>` 위의 바닐라 JS와 CSS다.

---

## 배포: GitHub Pages + Actions

`.github/workflows/deploy.yml`을 만들면 끝이다.

```yaml
on:
  push:
    branches: [master]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build   # astro build + pagefind 인덱스 생성
      - uses: actions/deploy-pages@v4
```

레포지터리 설정에서 Pages 소스를 **GitHub Actions**로 바꿔줘야 한다. Branch가 아니라 Actions로. 처음에 이걸 몰라서 한참 헤맸다.

---

## 우주 배경 — Canvas 2D 애니메이션

이 블로그의 핵심이다. 전체 페이지 배경(`#bg-canvas`)과 사이드바 내부(`#sidebar-canvas`) 두 곳에 `<canvas>`를 깔고, `requestAnimationFrame`으로 매 프레임 그린다.

### 배경색 사이클 — 다중 사인파

단색이 천천히 밝아졌다 어두워지는 건 너무 단순하다. 세 개의 독립 사인파를 RGB 채널에 각각 다른 비율로 섞으면, 훨씬 긴 주기의 비반복 색상 변화가 만들어진다.

```javascript
var t  = ts * 0.000042;  // 주기 ≈ 2.5분
var c1 = Math.max(0, Math.sin(t));                  // 크림슨 파
var c2 = Math.max(0, Math.sin(t * 0.68 + 2.0));    // 딥블루/틸 파
var c3 = Math.max(0, Math.sin(t * 0.45 + 4.1));    // 바이올렛 파

var r = Math.round(5  + c1 * 90 + c3 * 12);
var g = Math.round(5  + c2 * 18 + c1 * 6);
var b = Math.round(22 + c2 * 24 + c3 * 16);
```

세 파형의 주파수 비율이 공약수를 갖지 않도록 골랐다. 덕분에 짧은 주기에 눈에 띄는 패턴 반복이 없고, 배경이 언제나 조금씩 다르게 보인다.

### 별 — 8가지 색상 분류

별을 흰색 하나로만 그리면 밋밋하다. 실제 별처럼 색온도에 따라 색상을 분류했다.

```javascript
var TYPES = [
  { r:215, g:225, b:255 },  // blue-white (청백색 주계열성)
  { r:255, g:245, b:200 },  // warm white (황백색)
  { r:245, g:200, b:65  },  // gold       (황색)
  { r:255, g:140, b:65  },  // orange     (주황 거성)
  { r:255, g:80,  b:70  },  // red        (적색 거성)
  { r:185, g:150, b:255 },  // violet     (고온 청색성)
  { r:75,  g:205, b:235 },  // cyan       (청록)
  { r:255, g:110, b:185 },  // magenta    (특이성)
];
```

일부 별(`s.glow = true`)은 `createRadialGradient`로 후광을 추가한다. 금색 별(`s.gold = true`)은 별도로 처리해 배경 애니메이션의 금색 shimmer 그라디언트를 그 위에 덮어 반짝임을 강조한다.

### 성운 — 숨쉬는 색구름

6개의 성운(violet, rose, cyan, gold, deep-purple, emerald)을 `createRadialGradient`로 그린다. 각 성운마다 `pulse` 파형의 위상(phase)을 다르게 해서 동시에 밝아지지 않도록 했다.

```javascript
var pulse = 0.46 + 0.54 * Math.sin(ts * 0.00022 + nb.ph);
ng.addColorStop(0, `rgba(${r},${g},${b},${0.082 * pulse})`);
```

`0.082`라는 숫자는 맥스 불투명도다. 여기서 눈에 보이는 선명한 색덩어리가 아니라, 있는 듯 없는 듯 배경에 녹아드는 색감을 원했기 때문에 이 정도로 낮췄다.

### 바다와 파도 — Stokes 비대칭 파형

캔버스 하단 20%쯤을 바다로 채운다. 단순한 `sin` 파형은 너무 규칙적이라서, Stokes 비대칭 공식을 써서 마루가 날카롭고 골이 완만한 실제 파도 모양을 구현했다.

```javascript
function waveY(layer, x, elp) {
  var s1 = Math.sin(x * k - t * 2.2);
  var h1 = A * (s1 - layer.steep * s1 * s1);  // Stokes 비대칭
  var h2 = A * 0.27 * Math.sin(x * k * 1.74 + t * 1.8 + layer.p2); // 교차파
  var h3 = A * 0.14 * Math.sin(x * k * 0.40 - t * 1.1 + layer.p3); // 장파 너울
  var h4 = A * 0.20 * Math.sin(x * k * 3.8  - t * 4.5 + layer.p2 * 0.7); // 잔물결
  var h5 = A * 0.11 * Math.sin(x * k * 6.2  + t * 6.8 + layer.p3 * 1.3); // 파쇄
  return layer.yBase * H + h1 + h2 + h3 + h4 + h5;
}
```

5개의 조파를 중첩하면 각 레이어가 제법 자연스럽게 움직인다. 바다 색은 `skyR/G/B` 변수에서 파생해서, 배경색이 바뀔 때 바다도 함께 물든다.

날씨 시스템도 있다. `SUNNY`, `CLOUDY`, `RAIN`, `STORM`, `SNOW` 다섯 가지 상태가 가중치에 따라 랜덤하게 전환되고, 상태에 따라 구름 수, 빗방울, 눈송이, 파도 진폭이 달라진다.

---

## 게시글 헤더 — 파도 경계선

게시글 상단의 Hero 영역(제목, 태그, 날짜)과 본문 사이를 파도 모양으로 구분한다. CSS 커브로 고정된 형태를 쓰지 않고, JS `requestAnimationFrame`으로 매 프레임 SVG path의 `d` 속성을 갱신해 실제로 물결친다.

```javascript
function getY(x, t) {
  return (
    BASELINE
    + 13 * Math.sin(0.0072 * x + 0.32 * t)       // 주 파도
    +  5 * Math.sin(0.0140 * x - 0.21 * t + 1.3) // 교차 파도
    +  3 * Math.sin(0.0031 * x + 0.18 * t + 2.7) // 장파 배경
  );
}
```

파도 위에는 SMIL `<animateTransform>`으로 움직이는 금색 shimmer 그라디언트가 얹힌다. 파도 형태 애니메이션(JS)과 shimmer 스윕(SMIL)이 서로 독립적으로 돌기 때문에, 둘을 조율할 필요가 없다.

---

## 타이포그래피 — 제목 위계의 금색 바

게시글 본문 `h2 / h3 / h4` 왼쪽에 금색 세로 바를 넣어 위계를 시각화했다. CSS `::before` 가상 요소 하나로 해결했는데, 포인트는 위계가 낮을수록 바가 얇고 반투명해지도록 opacity를 달리한 것이다.

```css
.prose :where(h2)::before {
  width: 4px; background: rgba(201, 162, 39, 0.90); /* 굵고 진하게 */
}
.prose :where(h3)::before {
  width: 3px; background: rgba(201, 162, 39, 0.55); /* 중간 */
}
.prose :where(h4)::before {
  width: 2px; background: rgba(201, 162, 39, 0.30); /* 얇고 연하게 */
}
```

처음에는 색상도 바꿔볼까 했는데, 같은 금색의 불투명도만 달리하는 쪽이 훨씬 정돈된 느낌을 주었다.

---

## 인용문 카드 — CSS 폴리크롬 shimmer

각 게시글 상단에 랜덤 게임 디자인 명언을 보여주는 인용문 카드가 있다. 처음에는 단순한 반투명 글래스 스타일이었는데, 이번에 `conic-gradient`와 CSS `mask-composite` 기법을 써서 은색 무지개 shimmer 테두리를 입혔다.

### 테두리: conic-gradient + mask

```css
@property --qc-border {
  syntax: '<angle>';
  initial-value: 0deg;
  inherits: false;
}
@keyframes qc-border-spin { to { --qc-border: 360deg; } }

.quote-card::before {
  content: '';
  position: absolute;
  inset: 0;
  padding: 1.5px;           /* 테두리 두께 */
  border-radius: 6px;
  background: conic-gradient(
    from var(--qc-border),
    rgba(145,155,200,0.40),
    rgba(255,255,255,1.00),  /* 흰 하이라이트 */
    rgba(190,240,255,0.88),  /* 아이스블루 */
    rgba(210,180,255,0.78),  /* 소프트바이올렛 */
    rgba(255,242,192,0.78),  /* 웜골드 */
    rgba(145,155,200,0.40)
  );
  /* 가운데를 뚫어 1.5px 테두리만 남긴다 */
  -webkit-mask: linear-gradient(#fff 0 0) content-box,
                linear-gradient(#fff 0 0);
  -webkit-mask-composite: destination-out;
  mask-composite: exclude;
  animation: qc-border-spin 6s linear infinite;
}
```

`mask-composite: exclude`가 핵심이다. 두 마스크 레이어의 XOR — padding box 전체에서 content box를 빼면 padding 영역, 즉 테두리 띠만 남는다. `border-image`로는 `border-radius`와 함께 쓸 수 없는데, 이 방법은 가능하다.

### 내부: polychrome sheen

테두리만으로는 부족하다. `::after`로 `linear-gradient`를 카드 전체에 덮어 박스 자체가 무지개빛으로 은은하게 흐르게 했다.

```css
@property --qc-sheen { syntax: '<angle>'; initial-value: 135deg; inherits: false; }
@keyframes qc-sheen-sweep { to { --qc-sheen: 495deg; } }

.quote-card::after {
  background: linear-gradient(
    var(--qc-sheen),
    rgba(110, 80, 225, 0.11),   /* 퍼플 */
    rgba(65, 155, 245, 0.13),   /* 블루 */
    rgba(45, 210, 195, 0.09),   /* 틸 */
    rgba(215, 210, 50,  0.07),  /* 옐로 */
    rgba(238, 88,  182, 0.10),  /* 핑크 */
    rgba(110, 80,  225, 0.11)   /* 다시 퍼플 */
  );
  animation: qc-sheen-sweep 18s linear infinite;
}
```

테두리 회전(6s), 내부 쉰 스윕(18s), 큰따옴표 shimmer(4s) — 세 애니메이션이 서로 다른 속도로 돌기 때문에 같은 패턴이 반복되지 않는다.

### @property: CSS 커스텀 속성 애니메이션

`--qc-border`처럼 각도를 CSS 커스텀 속성으로 선언하면 `@keyframes`로 애니메이션할 수 있다. 단, `@property`가 없으면 브라우저가 두 각도 값 사이를 보간할 방법을 모른다. `@property`의 `syntax: '<angle>'` 선언이 "이 값은 각도다, 보간 가능하다"고 알려주는 역할을 한다.

---

## 플로팅 탐색 바

스크롤을 200px 이상 내리면 페이지 상단에서 슬라이드인되는 바. 좌측에 여우 로고(홈 링크), 중앙에 게시글 제목, 우측에 맨 위/아래 버튼이 있다.

포인트는 위치 계산이다. 사이드바(340px)를 제외한 본문 컬럼(최대 880px) 안에서 정확히 센터에 오도록, `getBoundingClientRect()`로 실제 DOM 위치를 읽어 `left`/`right`를 동적으로 설정한다.

```javascript
function positionBar() {
  if (window.innerWidth <= 1024) return;
  const rect = document.querySelector('.main-content').getBoundingClientRect();
  const gap = 24;
  floatBar.style.left  = Math.round(rect.left  + gap) + 'px';
  floatBar.style.right = Math.round(window.innerWidth - rect.right + gap) + 'px';
}
positionBar();
window.addEventListener('resize', positionBar, { passive: true });
```

CSS의 `100vw`로 계산하면 스크롤바 너비 때문에 미세하게 틀어진다. DOM을 직접 읽는 게 확실하다.

모바일에서는 고정 상단바 대신 플로팅 바가 그 역할을 대신한다. 게시글 페이지에 진입하면 JS로 `.app-brand`의 `position`을 `relative`로 바꿔 페이지와 함께 스크롤되도록 한다.

---

## Pagefind 검색 — IIFE를 ES 모듈로 착각한 버그

검색 페이지를 처음 만들 때 제일 황당했던 버그를 하나 남겨둔다.

```javascript
// ❌ 잘못된 방법 — pagefind-ui.js는 IIFE다
import('/pagefind/pagefind-ui.js').then(m => {
  new m.PagefindUI({ element: '#search' });
  // m.PagefindUI === undefined → TypeError
});

// ✅ 올바른 방법 — <script> 태그로 주입 → window.PagefindUI
const script = document.createElement('script');
script.src = '/pagefind/pagefind-ui.js';
script.onload = () => new window.PagefindUI({ element: '#search' });
document.head.appendChild(script);
```

`pagefind-ui.js`는 마지막 줄이 `window.PagefindUI = mt; })();`로 끝난다. export가 없다. ESM 동적 import는 export가 없는 스크립트에서 빈 객체를 반환하기 때문에, `m.PagefindUI`는 `undefined`였고 `new undefined()`가 TypeError를 던졌다. 파일 자체는 서버에 멀쩡히 있었다.

---

## 마치며

완성형 블로그가 아니다. 글 한 편이 올라올 때마다, 혹은 새 아이디어가 생길 때마다 디자인이 조금씩 바뀐다. 어떻게 보면 이 사이트 자체가 일종의 지속적인 프로젝트다. 이 글도 나중에 업데이트될 것이다.

코드를 보고 싶다면 [GitHub](https://github.com/Runawayfox/runawayfox.github.io)에 올려뒀다.
