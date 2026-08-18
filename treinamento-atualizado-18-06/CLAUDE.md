# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Tela estática de treinamento com sidebar, conteúdos, leitor de PDF e player de vídeo — a set of static, self-contained HTML prototypes for a mobile "trilha de treinamento" (training track) viewer for Lector's support team. There is no build system, package manager, or test suite; this is plain HTML/CSS/JS meant to be opened directly or served as static files.

## Running locally

There's no `package.json`. Serve the folder as static files, e.g.:

```
npx serve -p 3131 .
```

(This matches the preconfigured launch config in `.claude/launch.json`, named `lect-preview`.) Then open `index.html` (or any other top-level `.html` file) in the browser.

There is no lint, build, or test command — verify changes by opening the relevant HTML file in a browser.

## Architecture

Each top-level `.html` file is an **independent, self-contained page**: CSS lives in an inline `<style>` block in `<head>` and behavior lives in a single inline `<script>` block near the end of `<body>`. There are no shared `.js`/`.css` modules between pages — the only shared external assets are Google Fonts (Source Sans 3) and the custom icon font in `assets/lector-icons/`. Because of this, **UI/logic fixes that apply to more than one screen must be manually duplicated across each relevant file** — check the other pages when you change a shared pattern.

Pages:
- `index.html` — main training screen ("Treinamento Skill Suporte Lector"): left sidebar (trilha context/progress), tabbed content panels, inline PDF/video readers, and a performance chart.
- `Trilha Conteudo (Dir A).html` — alternate content-layout direction of the trilha screen (design comparison variant).
- `Trilha Desempenho.html` — the "Desempenho" (performance) tab/screen in isolation.
- `Trilha Wireframes.html` — low-fidelity wireframes of the trilha screen.
- `certificado.html` — certificate-of-completion screen.

Within `index.html`'s inline script, the key building blocks are:
- **Tab switching** (`activate(name)`): toggles `.tab`/`.panel` visibility and persists the active tab to `localStorage` (`lector-trilhaA-tab`). Switching to the `desempenho` tab triggers `renderChart()`.
- **Sidebar collapse**: driven by a `matchMedia('(max-width: 900px)')` listener (`syncSidebarMode`) that auto-collapses below 900px, plus manual open/close buttons.
- **Inline media viewer**: `openInlinePdf(src, title)` / `openInlineVideo(src, title)` / `closeInlineMedia()` swap an inline `<iframe>`/`<video>` panel in place, driven by `data-open-pdf` / `data-open-video` attributes on content cards. A separate **full-screen reader** (`openPdfReader`, `openVideoReader`, `closePdfReader`, `resetReader`) is used for the certificate flow.
- **Certificate flow**: `openCertificateFlow()` shows a loading state (`certificate-loading` class) for 1.4s, then opens `uploads/Certificado.pdf` in the full-screen PDF reader.
- **Performance chart**: `renderChart()` builds a bar chart from the hardcoded `CONTENT_VALUES` array (percentage per content item), switching to horizontal scroll mode when there are more than `SCROLL_AFTER` (10) items.
- **Content card state**: clicking an unlocked `.conteudo` card toggles a `checked` class and, for "Não iniciado" (not-started) cards, dynamically flips status to "Visualizado" and injects a progress/aproveitamento (`.pa`) donut-stat widget.
- On load, the script restores the saved tab from `localStorage` and auto-opens `uploads/manual-minha-area.pdf` in the inline PDF viewer.

Body classes (e.g. `sidebar-collapsed`, `inline-pdf-open`, `inline-video-open`, `reader-open`, `pdf-reader`, `video-reader`, `certificate-loading`, `certificate-reader`, `page-loaded`) are the primary state mechanism driving CSS visibility — most layout/visibility rules are keyed off these classes rather than component-scoped state.

### Forced mobile viewport (`index.html` and `Trilha Conteudo (Dir A).html` only)

These two pages always render in mobile layout, even in a wide desktop browser window — matching how these two screens are meant to be shown (they were fully designed down to a 420px phone breakpoint; the other three pages were not). The mechanism, duplicated identically in both files:
- A `<script>` as the very first element in `<head>` synchronously `document.write()`s a `<style id="mv-hide">html{visibility:hidden!important;}</style>` when the page is top-level, wider than `MOBILE_W` (420px), and not already the embedded instance (`?mv=1` query param) — this prevents any flash of the desktop layout before the wrap kicks in.
- A second `<script>` at the very end of `<body>` (after the page's own logic has run) does the actual wrap: if the same conditions hold, it clears `document.body`, injects a fixed-width (420px, full-height, no bezel) `<iframe>` pointing back to the same URL with `?mv=1` appended, and removes the `#mv-hide` style. Because the iframe is a nested browsing context, its width — not the real window width — is what the page's `@media` queries evaluate against, so the mobile breakpoints always apply inside it.
- The `?mv=1` param is what makes the embedded iframe instance skip re-wrapping itself (both scripts check `new URLSearchParams(location.search).get('mv') !== '1'`).

Do not add this snippet to `Trilha Wireframes.html`, `Trilha Desempenho.html`, or `certificado.html` — they were tried and reverted because those pages lack a real phone-width layout (no `@media` rules narrower than 1080px, or none at all), so forcing them into a 420px frame just clips content with horizontal scroll instead of showing a genuine mobile view. If a page later gains full mobile breakpoints down to ~420px, this snippet can be added to it too.

## Known issues

- In `index.html`, `activate(saved)` (called on load, and again whenever the "Desempenho" tab is opened) calls `renderChart(currentKey)`, but `currentKey` is not declared anywhere in the file — evaluating it throws `ReferenceError: currentKey is not defined`. Since this happens inside the final inline `<script>` block, the uncaught exception stops the rest of that script from running, which means the `openInlinePdf('uploads/manual-minha-area.pdf', ...)` call on the last line never executes — the auto-open-welcome-PDF behavior described above is currently broken. This is pre-existing (git history shows a `let currentKey = ...` declaration was removed in a later commit while the call site was left behind) and unrelated to any prototype-variant work; flag it if picking up related bugs in `index.html`.

## Assets

- `assets/lector-icons/` — custom icon font (IcoMoon-generated: css/, font/, demo.html, config.json). Icons are used via `<i class="ui-icon icon-*"></i>`.
- `uploads/` — sample content referenced by the prototypes: PDFs (`manual-minha-area.pdf`, `Certificado.pdf`), a sample video (`linguagens-programacao.mp4`), and pasted screenshots/covers used as card thumbnails.
- `.history/` — editor-generated local-history snapshots of `index.html`; not part of the app, safe to ignore.
