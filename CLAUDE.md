# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A_Bit_S Blog is a Jekyll-based publishing platform for technical content, primarily in Chinese. The site features both individual blog posts and serialized book-length content (books with chapters). It's deployed to GitHub Pages at `https://wzxzhuxi.github.io/`.

## Core Commands

### Development
```bash
# Install dependencies (run once after clone)
bundle install

# Start dev server with live reload (default port: 4000)
bundle exec jekyll serve --livereload

# Production build
JEKYLL_ENV=production bundle exec jekyll build

# Build with full error trace
bundle exec jekyll build --trace

# Pre-deployment health check
bundle exec jekyll doctor
```

### Validation workflow
Before pushing or opening PR:
1. `bundle exec jekyll build --trace` - must complete with zero warnings
2. `bundle exec jekyll doctor` - catches broken permalinks, missing assets
3. Manual browser check of `_site/` output, verify responsive layout

## Architecture

### Collections system
The site uses three custom Jekyll collections beyond standard posts:

- **`_books/`**: Book metadata files (flat structure)
  - Structure: Flat directory with one file per book
  - Book file: `_books/book-slug.md`
  - Front matter fields: `title`, `slug`, `author`, `description`, `cover`, `order`
  - Each book's `slug` field acts as the primary key linking to chapters in `_chapters/`
  - Books appear at `/books/:slug/` via permalink pattern in `_config.yml`
  - **CRITICAL**: Book filename (without .md) and `slug` field must match exactly

- **`_chapters/`**: Chapter files organized by book
  - Structure: Subdirectories per book (`_chapters/book-slug/`)
  - Chapter files: `_chapters/book-slug/chapter-name.md` (can be simple, e.g., `01-basics.md`)
  - Front matter fields: `title`, `book`, `order`, `summary`, `permalink` (optional)
  - Each chapter specifies `book: book-slug` linking to parent book
  - Chapters appear at `/books/:slug/:chapter-slug/` or custom `permalink`

- **`_diaries/`**: Learning journal collection
  - Structure: Organized by author (`_diaries/author-key/`)
  - Journal entries: `_diaries/author-key/YYYY-MM-DD-title.md`
  - Template: `_templates/DIARY_TEMPLATE.md` (in templates directory, not processed by Jekyll)
  - Front matter fields: `title`, `author`, `date`, `mood`, `weather`, `location`, `excerpt`

- **`_templates/`**: Template files (not processed by Jekyll)
  - `BOOK_TEMPLATE.md` - Book main file template
  - `CHAPTER_TEMPLATE.md` - Chapter template
  - `DIARY_TEMPLATE.md` - Learning journal entry template

### Book and Chapter Relationship

Books and chapters are now in separate collections with explicit linking:
- The `book.html` layout queries chapters via `site.chapters | where: "book", book_slug`
- Book files (in `_books/`) are identified by having a `slug` field
- Chapter files (in `_chapters/`) are identified by having a `book` field
- **Consistency requirement**: Book slug → `_books/book-slug.md`, Chapter book field → `_chapters/book-slug/`

### Layout hierarchy
```
_layouts/default.html          # Base wrapper with <html>, <head>, site navigation
  ├─ _layouts/home.html        # Homepage post grid, uses pagination
  ├─ _layouts/post.html        # Single article view
  ├─ _layouts/page.html        # Static pages (team.md, etc.)
  ├─ _layouts/book.html        # Book landing page, dynamically lists chapters
  ├─ _layouts/chapter.html     # Individual chapter content
  └─ _layouts/diary.html       # Diary entry view
```

Key layout behaviors:
- `book.html` - Queries all chapters matching current book slug from `site.chapters`, sorts by `order` field
- `post.html` - Displays metadata: date, author, categories in header
- `home.html` - Lists all books from `site.books` (filtering not needed with separate collections)
- `books.md` - Iterates `site.books` to display all books
- `diaries.md` - Iterates `site.data.authors` to build list of authors with diaries, then displays latest entries per author

### Front matter conventions
All front matter keys use **snake_case** (e.g., `reading_time`, `cover_alt`), never camelCase.

**Post template** (`_posts/YYYY-MM-DD-slug.md`):
```yaml
---
layout: post
title: "Article Title"
author: author_slug        # String identifier (e.g., "zhuxi", "allen")
tags: [tag1, tag2]
excerpt: >
  1-2 sentence summary for listing pages
cover: /assets/images/posts/example.png  # Optional
---
```

Note: The `author` field references keys in `_data/authors.yml` for centralized author management.

**Book template** (`_books/book-slug.md`):
```yaml
---
title: Book Title
slug: book-slug                # MUST match filename (without .md)
author: author_key             # Key from _data/authors.yml
description: Brief description for metadata/listings
cover: /assets/images/books/book-slug/cover.png  # Optional
order: 1                       # Controls sort order in book listings
---
Book synopsis or reading guide...
```

**Chapter template** (`_chapters/book-slug/chapter-name.md`):
```yaml
---
title: Chapter Title
book: book-slug                # MUST match parent book slug
order: 1
summary: Short blurb for navigation/ToC
permalink: /books/book-slug/chapter-slug/  # Optional custom path
---
Chapter content...
```

**Diary template** (`_diaries/author-key/YYYY-MM-DD-title.md`):
```yaml
---
title: Learning Journal Entry
author: author_key             # Key from _data/authors.yml
date: YYYY-MM-DD
mood: hopeful
weather: sunny
location: Coffee Shop
excerpt: >
  Short summary of today's learning
---
Detailed reflection on technical learning, insights, challenges...
```

### Naming patterns
- Posts: `_posts/YYYY/YYYY-MM-DD-slug.md` where slug is lowercase, digits, hyphens only
- Books: `_books/book-slug.md` - filename and slug field must match
- Chapters: `_chapters/book-slug/chapter-name.md` - subdirectory must match book slug; filename can be simple (e.g., `01-basics.md`)
- Diaries: `_diaries/author-key/YYYY-MM-DD-title.md` - organized by author

### Styling architecture
- Global styles: `assets/css/style.scss` (SCSS with nesting, compiled by Jekyll)
- Inline styles: Some layouts embed `<style>` blocks for component-specific rules (e.g., `book.html:45-182`)
- Class naming: BEM-style, max 2 levels of nesting
- Responsive breakpoints: Mobile handled via `@media (max-width: 768px)`

## Critical Files

- `_config.yml` - Site metadata, collection definitions, plugin configuration, navigation links, pagination settings
  - **Must restart `jekyll serve` after editing this file**
  - Defines three collections: `books`, `chapters`, `diaries` (all with `output: true`)
  - Sets default layout per collection via `defaults` array (books → `book.html`, chapters → `chapter.html`, diaries → `diary.html`)
  - Defines permalink patterns for each collection to generate proper URLs

- `Gemfile` - Uses `github-pages` gem for GitHub Pages compatibility
  - Includes Ruby 3.4 stdlib shims (`erb`, `logger`, `csv`, `base64`, `bigdecimal`) - these gems are no longer bundled in Ruby 3.4+
  - If you get "cannot load such file" errors for these stdlib components, they're already declared in the Gemfile

- `_site/` - **NEVER edit manually**. Auto-generated output, excluded from git, disposable

- `_includes/` - Currently empty. This directory is for reusable Liquid template fragments that can be included in layouts

## Content authoring

### Markdown conventions
- Start article body at `##` (h2) - layouts provide `<h1>` from title
- Code blocks: Use triple backticks with language hint (e.g., ` ```c# `)
- Lists: Use `-` for bullets, 2-space indent for nesting
- Line length: Aim for 80-100 chars for better diffs
- Links: Prefer reference style for readability:
  ```markdown
  Text with [link][ref]

  [ref]: https://example.com
  ```

### Assets management
- Store all media/downloads under `assets/` hierarchy
- Reference with absolute paths: `/assets/images/posts/foo.png`
- Images get auto-bordered via CSS (`border: 3px solid #000`)

### Book vs post vs diary decision
- **Post**: Single-topic article, standalone technical content
  - Create in `_posts/YYYY/YYYY-MM-DD-slug.md`
  - Use `_layouts/post.html`
- **Book**: Multi-chapter series requiring ordered navigation and unified branding
  - Create book metadata: `_books/book-slug.md` with `slug: book-slug`
  - Create chapter subdirectory: `_chapters/book-slug/`
  - Add chapter files: `_chapters/book-slug/chapter-name.md` with `book: book-slug` matching parent slug
  - Book page auto-generates chapter listing via Liquid query: `site.chapters | where: "book", book_slug`
  - Chapters are sorted by `order` field
- **Diary**: Learning journal entries, personal technical notes
  - Create in `_diaries/author-key/YYYY-MM-DD-title.md` with `author: author_key`
  - Organized by author for scalability
  - Not private - public technical learning logs
  - `diaries.md` auto-generates author listing from `site.data.authors`, showing only authors with diary entries

## Deployment

GitHub Pages deployment workflow:
1. Push to main branch
2. GitHub Actions (or Pages branch) runs `JEKYLL_ENV=production bundle exec jekyll build`
3. Publishes `_site/` directory to Pages hosting
4. Never commit `_site/` to repo (it's in `.gitignore`)

## Existing content reference

Current site structure:
- **Books**:
  - `cpp-functional-programming` (C++ functional programming series by Zhuxi) - 10 chapters covering setup through advanced patterns
  - Book metadata: `_books/cpp-functional-programming.md` with `slug: cpp-functional-programming`
  - Chapters: `_chapters/cpp-functional-programming/` containing 10 chapter files with `book: cpp-functional-programming`
- **Posts**: Technical articles stored in `_posts/YYYY/`
- **Diaries**: Learning journals organized by author in `_diaries/author-key/`
- **Pages**: `team.md` team roster, `books.md` book listing, `diaries.md` journal listing

The codebase demonstrates the new collections pattern: `_books/cpp-functional-programming.md` defines book metadata, and chapters in `_chapters/cpp-functional-programming/` link back via `book: cpp-functional-programming` field. The `book.html` layout queries chapters from `site.chapters` and filters by matching `book` slug.

## Common pitfalls

- **Forgetting to restart jekyll serve**: Required after `_config.yml` changes
- **Wrong front matter keys**: Use `book:` not `book_slug:` in chapters, use `slug:` in books, use `author:` in posts/diaries
- **Mismatched slugs/directories**:
  - Book filename must match `slug` field: `_books/book-slug.md` with `slug: book-slug`
  - Chapter `book` field must exactly match parent book slug: `_chapters/book-slug/` subdirectory with `book: book-slug`
  - Chapter directory prefix determines the collection grouping - must be book slug
- **Wrong file locations**: Chapters must be in `_chapters/book-slug/` subdirectory, not in `_books/`
- **Missing `order` fields**: Causes unpredictable chapter and post sorting
- **Starting content at `#`**: Clashes with layout-provided h1, start at `##`
- **Committing `_site/`**: Build output, should never be in version control
- **Author field inconsistency**: Must reference existing keys in `_data/authors.yml` for posts and diaries

## Testing chapter/book relationships

When adding new chapters:
1. Verify book slug matches:
   - Book file: `grep "slug:" _books/book-slug.md`
   - Chapters: `grep "book:" _chapters/book-slug/*.md` - must all match the book slug
2. Check chapter directory structure: Verify `_chapters/book-slug/` subdirectory exists
3. Check chapter ordering: `grep "order:" _chapters/book-slug/*.md | sort`
4. Build and verify book page lists all chapters at `/books/book-slug/`
5. Confirm chapter permalinks resolve correctly
6. Check that only books appear in book listings on homepage and `/books/` page (chapters should not appear there)

## Plugins in use

GitHub Pages-compatible plugins (defined in `_config.yml`):
- `jekyll-feed` - RSS/Atom feed generation
- `jekyll-seo-tag` - Meta tags for SEO
- `jekyll-sitemap` - XML sitemap
- `jekyll-paginate` - Pagination (configured for 6 posts per page)
