# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static, self-contained HTML prototypes of the "trilha de treinamento" (training
track) screens for Lector's support team. There is no build system, package
manager, or test suite — plain HTML/CSS/JS meant to be opened directly or served
as static files, and deployed to Vercel as-is.

## Running locally

There's no `package.json`. Serve the folder as static files:

```
npx serve -p 3131 .
```

Then open `http://localhost:3131/trilha.html` — `trilha.html` is the entry point
of the flow. There is no lint, build, or test command; verify changes by opening
the relevant page in a browser.

## Deploy

Vercel project `modelagem-visualizacao-treinamento-trilha` (team `lector`),
connected to the `main` branch of the GitHub repo — pushing to `main` triggers a
production deploy. `vercel.json` only sets `cleanUrls: true`, so production URLs
drop the `.html` (`/trilha`, `/avaliacao`, …). **Note:** `/` serves `index.html`
(the treinamento screen), not the trilha.

Because `cleanUrls` rewrites `/index.html?x` → `/index` and drops the query
string, pages must not rely on query params to detect their embedding context —
see the frame-detection notes below.

## Architecture

Each top-level `.html` file is an **independent, self-contained page**: CSS lives
in an inline `<style>` block in `<head>` and behavior in a single inline
`<script>` block near the end of `<body>`. There are no shared `.js`/`.css`
modules — the only shared external assets are Google Fonts (Source Sans 3) and
the icon font in `assets/lector-icons/`. Because of this, **a fix that applies to
more than one screen must be manually duplicated in each file** — check the
sibling pages whenever you touch a shared pattern.

### Pages

- `trilha.html` — "Trilha Completa Automação": left sidebar with etapas and
  conteúdos, tabs *Descrição / Desempenho / Relatórios*, and an inline PDF
  viewer that auto-opens `uploads/manual-minha-area.pdf` on load. Entry point.
- `index.html` — "Treinamento Skill Suporte Lector": one content screen, with
  eight tabs (*Descrição, Resumo, Desempenho, Material Complementar, Sobre o
  Autor, Tutores, Fóruns, Sugestões*), inline PDF/video/avaliação readers, a
  performance chart, a forum feed, a tutor chat, and the certificate flow.
- `avaliacao.html` — the assessment: five questions, timer, and result screen.
- `certificado.html` — certificate of completion.
- `Trilha Desempenho.html` — the Desempenho tab in isolation (design variant).
- `Trilha Wireframes.html` — low-fidelity wireframes of the trilha.

### How the screens nest

The three main screens embed each other in iframes rather than navigating:

- `trilha.html` → `openTraining(src, title)` loads `index.html` (from
  `data-open-training`) or `avaliacao.html` (from `data-open-avaliacao`) into
  the `#trainingFrame` overlay, and exposes `window.closeTraining`.
- `index.html` → `openInlineAvaliacao(src, title)` loads `avaliacao.html` into
  `#inlineAvaliacaoFrame` as inline content, and exposes `window.syncQuizNav`.
- `index.html` decides whether it is embedded by checking that its parent
  exposes `closeTraining` (**not** by query string — `cleanUrls` strips it). When
  embedded it shows the "voltar para a trilha" button in the topbar.
- `avaliacao.html` decides its mode from `window.frameElement.id`:
  `#inlineAvaliacaoFrame` means it is inline content inside the treinamento, so
  it hides its own arrows (`body.nav-externa`) and lets the parent drive it
  through the `window.quizNav` API (`isActive/canPrev/canNext/prev/next/
  nextLabel/prevLabel`); `#trainingFrame` means it is the trilha overlay and it
  uses its own arrows.

### `index.html` internals

- **Tab switching** (`activate(name)`) toggles `.tab`/`.panel` and persists to
  `localStorage` under `lector-treinamento-tab`; switching to `desempenho`
  triggers `renderChart()`. On load the saved tab is validated against the tabs
  that actually exist before being applied — `trilha.html` uses a separate key
  (`lector-trilha-tab`) precisely so the two tab sets can't collide.
- **Sidebar collapse** via a `matchMedia('(max-width: 900px)')` listener
  (`syncSidebarMode`), plus manual open/close buttons.
- **Inline media viewer**: `openInlinePdf` / `openInlineVideo` /
  `openInlineAvaliacao` / `closeInlineMedia`, driven by `data-open-pdf`,
  `data-open-video` and `data-open-avaliacao` attributes on content cards. A
  separate full-screen reader (`openPdfReader`, `openVideoReader`,
  `closePdfReader`, `resetReader`) serves the certificate flow.
- **Content navigation**: `openContentByIndex(i)`, `goPrevContent`,
  `goNextContent`, with `updateNavButtons()` reflecting either content position
  or, when the avaliação is open inline, the quiz's own `quizNav` state.
- **Certificate flow**: `openCertificateFlow()` shows a loading state, then opens
  `uploads/Certificado.pdf` in the full-screen reader.
- **Performance chart**: `renderChart()` builds a bar chart from hardcoded
  values, switching to horizontal scroll past a threshold.

### `avaliacao.html` internals

Questions live in a hardcoded `QUESTIONS` array; each has a `type` that decides
how `renderQuestion()` draws it and how `isAnswered`/`isCorrect` grade it:
`unica`, `multipla`, `vf`, `associativa` (a `<select>` per row) and `lacuna`
(free text matched against an `accepts` list). `advance()` moves forward, or
calls `finish()` on the last question, which swaps `#quiz` for `#resultStage`
and marks the attempt approved at ≥60%.

Body classes (`sidebar-collapsed`, `inline-pdf-open`, `inline-video-open`,
`reader-open`, `training-open`, `nav-externa`, `sheet-open`, `page-loaded`, …)
are the primary state mechanism — most visibility rules key off them rather than
component-scoped state.

## Assets

- `assets/lector-icons/` — IcoMoon icon font, used as `<i class="ui-icon icon-*"></i>`.
- `assets/logo-lector.svg` — logo.
- `uploads/` — sample content: PDFs (`manual-minha-area.pdf`, `Certificado.pdf`),
  a sample video, and screenshots/covers used as card thumbnails.
