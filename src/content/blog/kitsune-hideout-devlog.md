---
title: "Kitsune Hideout 블로그 UI 제작기"
description: "우주 캔버스 애니메이션, CSS shimmer 효과, 파비콘 생성, 검색 버그 수정까지 — 이 블로그를 만들면서 겪은 기술적 도전들을 정리합니다."
pubDate: 2026-10-08
tags: ["astro", "web", "blog", "personal"]
---

이 블로그를 만들면서 겪었던 기술적 도전들을 기록해두려고 합니다. 디자인 콘셉트 설정부터 세부 버그 수정까지, 꽤 많은 것을 배웠습니다.

## 디자인 콘셉트: 우주 속 여우 은신처

"Kitsune Hideout"는 일본 신화 속 여우 신령(狐, きつね)에서 이름을 따왔습니다. 사이드바에 우주 배경 애니메이션을 넣어 신비롭고 몽환적인 분위기를 만드는 것이 목표였습니다.

기술 스택은 간단합니다.

- **Astro v7** — 정적 사이트 생성, 콘텐츠 컬렉션
- **Tailwind CSS v4** — `@tailwindcss/vite` 플러그인 방식
- **Pagefind** — 빌드 후 인덱스 생성 방식의 검색
- **GitHub Pages + Actions** — 자동 배포

---

## 우주 배경 캔버스 애니메이션

사이드바와 페이지 배경 두 곳에 `<canvas>`를 배치하고 별과 성운을 그립니다. 처음에는 단색 배경에 흰 별만 있었는데, 점점 발전시켜서 8가지 색상의 별과 6개의 성운, 다중 파형 색상 사이클까지 넣게 됐습니다.

### 다중 파형 배경색 사이클

배경색이 단순하게 밝아졌다 어두워지는 게 아니라 violet/teal/indigo 세 가지 색조가 서로 다른 속도로 물결치도록 했습니다.

```javascript
// 3개의 독립적인 사인파로 복잡한 색상 변화를 만든다
var t  = ts * 0.000013;
var c1 = Math.max(0, Math.sin(t));              // violet 파
var c2 = Math.max(0, Math.sin(t * 0.68 + 2.0)); // teal 파
var c3 = Math.max(0, Math.sin(t * 0.45 + 4.1)); // indigo 파

var r = Math.round(4  + c1 * 22 + c3 * 8);
var g = Math.round(5  + c2 * 14 + c1 * 4);
var b = Math.round(22 + c2 * 24 + c3 * 16);

var skyGrad = ctx.createLinearGradient(0, 0, 0, H);
skyGrad.addColorStop(0,   `rgb(${r+8},${g+3},${b+10})`);
skyGrad.addColorStop(0.5, `rgb(${r},${g},${b})`);
skyGrad.addColorStop(1,   `rgb(${r-2},${g-2},${b-4})`);
```

세 파형이 공약수 없는 주파수비로 돌기 때문에, 단순한 sin 한 개보다 훨씬 긴 주기를 갖는 비반복 패턴처럼 보입니다.

### 성운 렌더링

성운은 `createRadialGradient`로 그립니다. 불투명도에 별도의 사인파를 적용해 천천히 숨쉬는 효과를 줬습니다.

```javascript
var nebulae = [
  { nx:0.45, ny:0.12, nr:1.00, r:115, g:65,  b:230, sp:0.0000025, ph:0.0 }, // violet
  { nx:0.72, ny:0.38, nr:0.88, r:220, g:55,  b:120, sp:0.0000020, ph:2.1 }, // rose
  { nx:0.28, ny:0.65, nr:0.82, r:45,  g:170, b:215, sp:0.0000016, ph:4.2 }, // cyan
  { nx:0.82, ny:0.06, nr:0.72, r:210, g:158, b:48,  sp:0.0000030, ph:1.0 }, // gold
  { nx:0.18, ny:0.88, nr:0.68, r:130, g:50,  b:205, sp:0.0000022, ph:3.0 }, // purple
  { nx:0.88, ny:0.55, nr:0.62, r:45,  g:185, b:148, sp:0.0000018, ph:5.5 }, // emerald
];

// 각 성운마다 위상(ph)이 달라 동시에 밝아지지 않는다
var pulse = 0.46 + 0.54 * Math.sin(ts * 0.00022 + nb.ph);
var ng = ctx.createRadialGradient(nx, ny, 0, nx, ny, nb.nr * W);
ng.addColorStop(0,    `rgba(${r},${g},${b},${0.082 * pulse})`);
ng.addColorStop(0.42, `rgba(${r},${g},${b},${0.034 * pulse})`);
ng.addColorStop(1,    `rgba(${r},${g},${b},0)`);
```

---

## CSS Shimmer 카드 테두리

카드에 마우스를 올리면 테두리가 빛나는 효과를 순수 CSS로 구현했습니다. 핵심은 `mask-composite: exclude`입니다.

```css
.post-card {
  position: relative;
}

/* ::after 가상 요소로 1px 그라데이션 링을 만든다 */
.post-card::after {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: inherit;
  padding: 1px;

  background: linear-gradient(135deg,
    #2266bb 0%, #88ccff 40%, #2266bb 60%, #cceeff 80%, #2266bb 100%
  );
  background-size: 200% 200%;

  /* 안쪽 영역을 마스크로 제거 → 1px 테두리만 남는다 */
  -webkit-mask:
    linear-gradient(#fff 0 0) content-box,
    linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;

  opacity: 0;
  transition: opacity 0.3s ease;
}

.post-card:hover::after {
  opacity: 1;
  animation: borderShimmer 2s linear infinite;
}

@keyframes borderShimmer {
  0%   { background-position: 200% 200%; }
  100% { background-position: -200% -200%; }
}
```

`background-clip`이나 `border-image`로도 유사한 효과를 낼 수 있지만, `mask-composite` 방법이 `border-radius`와 가장 자연스럽게 어울립니다.

---

## 파비콘: PNG 마스크 → 금색 래스터화

`fox-mask.png`를 그대로 파비콘으로 쓰되, 흰색 실루엣을 금색으로 바꿔야 했습니다. CSS에서 하던 방식(마스크 + 그라데이션)을 Node.js에서 재현했습니다.

```javascript
import sharp from 'sharp';

// 1. 마스크 PNG를 그레이스케일로 읽는다 (흰 여우 = 255, 배경 = 0)
const { data: mask } = await sharp('public/fox-mask.png')
  .resize(32, 32, { fit: 'contain', background: { r:0, g:0, b:0, alpha:1 } })
  .flatten({ background: { r:0, g:0, b:0 } })
  .grayscale()
  .raw()
  .toBuffer({ resolveWithObject: true });

// 2. 밝기값을 알파로 변환 → 여우 픽셀 = 불투명 금색, 배경 = 투명
const rgba = Buffer.alloc(32 * 32 * 4);
for (let i = 0; i < mask.length; i++) {
  rgba[i * 4 + 0] = 201; // R (#c9a227)
  rgba[i * 4 + 1] = 162; // G
  rgba[i * 4 + 2] = 39;  // B
  rgba[i * 4 + 3] = mask[i]; // 밝기 → 알파
}

// 3. 검은 배경 위에 금색 여우를 합성
await sharp({ create: { width: 32, height: 32, channels: 3,
                         background: { r:7, g:7, b:15 } } })
  .composite([{ input: await sharp(rgba, {
      raw: { width:32, height:32, channels:4 }
    }).png().toBuffer(), gravity: 'center' }])
  .png()
  .toFile('public/favicon-32x32.png');
```

단순히 `.tint()`를 쓰면 흰색의 높은 밝기가 유지되어 결과가 밝은 노란색에 가까워집니다. 밝기를 알파로 변환하는 방법을 쓰면 진한 금색(#c9a227)을 정확하게 재현할 수 있습니다.

---

## Pagefind 검색 버그: IIFE를 ES 모듈로 착각

가장 허무한 버그였습니다. `pagefind-ui.js`가 ES 모듈이라고 가정하고 `import()`로 불렀는데, 실제로는 IIFE 스크립트였습니다.

```javascript
// ❌ 잘못된 방법
import('/pagefind/pagefind-ui.js').then(m => {
  new m.PagefindUI({ element: '#search' });
  // m.PagefindUI === undefined → TypeError → catch로 떨어짐
});

// ✅ 올바른 방법: <script> 태그로 주입 → window.PagefindUI 사용
const script = document.createElement('script');
script.src = '/pagefind/pagefind-ui.js';
script.onload = () => {
  new window.PagefindUI({ element: '#search' });
};
document.head.appendChild(script);
```

`pagefind-ui.js`의 마지막 줄은 이렇게 생겼습니다:

```javascript
window.PagefindUI = mt; })();
```

export가 없습니다. ESM 동적 import는 export가 없는 스크립트에서 빈 네임스페이스 객체를 반환합니다. `m.PagefindUI`는 `undefined`였고, `new undefined()`는 TypeError를 던졌으며, `.catch()`가 그걸 받아 "검색 인덱스를 불러올 수 없습니다" 메시지를 표시했습니다. 파일은 멀쩡하게 서버에 있었는데 처음부터 잘못된 방법으로 불러오고 있었던 겁니다.

---

## 마치며

완성형 블로그보다는 계속 손을 대는 블로그가 될 것 같습니다. 기능을 하나씩 추가하고 버그를 잡으면서 기록을 남기는 게 목표입니다.
