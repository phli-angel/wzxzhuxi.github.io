# A_Bit_S Blog (English)

> 🌐 Language / 语言: [中文](README.md)

## Overview
A_Bit_S Blog is our publishing hub built on Jekyll. Content pages (`index.html`, `team.md`) and dated posts under `_posts/` render through layouts in `_layouts/` and snippets in `_includes/`. Styles live in `assets/css/style.scss`, while `_config.yml` defines metadata, navigation, and plugin settings. Generated output resides in `_site/` and should never be edited or committed.

## Getting Started
1. Install Ruby 3.x, Bundler, and ensure `node` is available for asset tooling.
2. Run `bundle install` to install dependencies listed in `Gemfile`.
3. Start the dev server:
   ```bash
   bundle exec jekyll serve --livereload
   ```
   Visit `http://127.0.0.1:4000` in your browser.

## Development & Validation
- **Content editing**: add Markdown posts named `YYYY-MM-DD-title.md` inside `_posts/`. Each file must include YAML front matter with `layout`, `title`, and any custom variables referenced in layouts.
- **Styling**: modify `assets/css/style.scss`; the SCSS is compiled automatically by Jekyll.
- **Configuration**: edit `_config.yml` when changing navigation, URL structures, or enabling plugins, then restart `jekyll serve`.
- **Quality checks**: run `bundle exec jekyll build --trace` plus `bundle exec jekyll doctor` before pushing or opening a pull request.

## Content Creation Guide
### Adding Posts (`_posts/`)
1. **Filename**: `YYYY-MM-DD-title.md`. The date controls ordering; keep `title` lowercase with digits and hyphens only (e.g., `2025-12-01-zero-downtime.md`).
2. **Front matter template**:
   ```yaml
   ---
   layout: post
   title: "Sample Article Title"
   author: allen          # Matches an entry under authors/
   tags: [devops, release]
   excerpt: >
     1–2 sentence summary shown on listing pages.
   cover: /assets/images/posts/example-cover.png
   ---
   ```
3. **Body writing tips**:
   - Keep lines within ~80–100 characters for easier diffs.
   - Store media/downloads under `assets/` and reference them with absolute paths (e.g., `![Diagram](/assets/images/posts/foo.png)`).
   - Prefer reference-style links:
     ```
     Text[^ref]

     [^ref]: https://example.com
     ```
4. **Pre-publish checks**: ensure `bundle exec jekyll build --trace` and `bundle exec jekyll doctor` both pass.

### Adding Books (`_books/` & `_chapters/`)
1. **Book metadata** (`_books/<slug>.md`):
   ```yaml
   ---
   title: Building Scalable Systems
   slug: building-scalable-systems
   description: A long-form series on distributed system evolution
   cover: /assets/images/books/bss.png   # Optional
   order: 1                              # Controls listing order
   ---
   ```
   Use the body for a synopsis or reading guide.
2. **Chapter files** (`_chapters/<book-slug>-NN.md`):
   ```yaml
   ---
   title: Chapter 1 — System Foundations
   book: building-scalable-systems   # Must match the slug above
   order: 1
   summary: Short blurb used in navigation
   permalink: /books/building-scalable-systems/system-foundations/
   ---
   ```
   Structure chapters with sections such as introduction, core concepts, checklists/code, and references. Use `##`/`###` headings to break up content.
3. **Navigation**: book pages usually list chapters automatically; just keep `book` and `order` accurate, and update `_config.yml` navigation links if you add a new book.

### Markdown & Front Matter Conventions
- Start article content at `##` to avoid clashing with layout-provided `h1`.
- Use fenced code blocks with language hints (e.g., start with &#96;&#96;&#96;bash).
- Lists may use `-` or ordered numbers; indent child items by two spaces to match the current style.
- When introducing specialized English terms, add brief explanations for bilingual readability.
- Stick to snake_case for front matter keys (e.g., `reading_time`, `cover_alt`).

## Deployment
We deploy to GitHub Pages at `https://wzxzhuxi.github.io/`. Use GitHub Actions (or the Pages branch) to run:
```bash
JEKYLL_ENV=production bundle exec jekyll build
```
Publish the generated `_site/` directory and never commit it to the repo.

## Contributing
Follow the branching, testing, and PR expectations described in `AGENTS.md`. Keep commits small with imperative subjects (e.g., `Add hero section layout`), attach screenshots/GIFs for visual updates, and verify `jekyll build` plus `jekyll doctor` locally before requesting review.
