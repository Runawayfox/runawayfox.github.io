---
title: "Astro로 GitHub Pages 블로그 만들기"
description: "Astro와 Tailwind CSS를 사용하여 GitHub Pages에 블로그를 배포하는 방법을 정리합니다."
pubDate: 2026-10-08
tags: ["astro", "blog", "web", "github-pages"]
---

이 블로그를 만드는 과정에서 사용한 기술 스택과 배포 방법을 정리합니다.

## 기술 스택

- **[Astro](https://astro.build)**: 정적 사이트 생성기. Rust 기반 컴파일러로 빌드가 빠르고, 아일랜드 아키텍처로 필요한 곳에만 JavaScript를 사용합니다.
- **[Tailwind CSS v4](https://tailwindcss.com)**: 유틸리티 클래스 기반 CSS 프레임워크.
- **[Pagefind](https://pagefind.app)**: 정적 사이트용 검색 라이브러리. 빌드 후 인덱스를 생성합니다.
- **GitHub Pages**: 정적 사이트 무료 호스팅.

## 프로젝트 구조

```
src/
  content/
    config.ts        # 콘텐츠 컬렉션 스키마
    blog/            # 마크다운 포스트
  layouts/
    Layout.astro     # 기본 레이아웃
    BlogPost.astro   # 블로그 포스트 레이아웃
  components/
    Header.astro
    Footer.astro
    PostCard.astro
    TagPill.astro
    ThemeToggle.astro
  pages/
    index.astro
    blog/[slug].astro
    tags/[tag].astro
    search.astro
```

## GitHub Pages 배포

GitHub Actions를 통해 `master` 브랜치에 푸시하면 자동으로 빌드 및 배포됩니다.

```yaml
on:
  push:
    branches: [master]
```

## 다크모드 구현

`localStorage`에 설정을 저장하고, 페이지 로드 시 즉시 적용하여 깜빡임(FOUC)을 방지합니다.

```javascript
// Layout.astro의 <head>에 인라인 스크립트로 삽입
const stored = localStorage.getItem('theme');
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
if (stored === 'dark' || (!stored && prefersDark)) {
  document.documentElement.classList.add('dark');
}
```
