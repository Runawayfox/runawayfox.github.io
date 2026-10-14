---
name: project-blog
description: RunawayFox GitHub Pages 블로그 프로젝트 현황 (Astro + Tailwind CSS)
metadata: 
  node_type: memory
  type: project
  originSessionId: f44d4d1e-bf4c-42e6-b3ba-29d07400c5ff
---

이 프로젝트는 Astro v7 + Tailwind CSS v4로 만든 GitHub Pages 블로그.

**Why:** 개인 블로그를 GitHub Pages에서 무료 호스팅하기 위해 구축.
**How to apply:** 블로그 기능 추가/수정 시 Astro 7 콘텐츠 컬렉션 API와 Tailwind v4 문법을 기준으로 작업.

주요 구조:
- 저장소: https://github.com/Runawayfox/runawayfox.github.io
- 배포 URL: https://runawayfox.github.io
- 콘텐츠: `src/content/blog/*.md` (Astro 7 content layer API, `src/content.config.ts`)
- 배포: GitHub Actions (`.github/workflows/deploy.yml`), GitHub Pages source를 "GitHub Actions"로 설정 필요
- 검색: Pagefind (빌드 후 `npx pagefind --site dist`로 인덱스 생성)
- 다크모드: HTML `<html class="dark">` + localStorage 저장

Astro 7 주의사항:
- 콘텐츠 컬렉션 설정은 `src/content/config.ts` 아닌 `src/content.config.ts`
- `post.slug` 대신 `post.id.replace(/\.mdx?$/, '')` 사용
- `render(post)` import from `astro:content`
- Tailwind v4: `@custom-variant dark (&:where(.dark, .dark *));` for class-based dark mode
