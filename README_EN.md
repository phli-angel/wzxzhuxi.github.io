# A_Bit_S Blog

> 🌐 Language / 语言: [中文](README.md)

## Project Overview

A_Bit_S Blog is a technical content publishing platform for the team, built on Jekyll and deployed on GitHub Pages (`https://wzxzhuxi.github.io/`).

It supports two content formats: articles and books (multi-chapter long-form content), with a unified author data management system for multi-author collaboration.

## Quick Start

### Local Development

```bash
# 1. Install dependencies
bundle install

# 2. Start dev server with live reload
bundle exec jekyll serve --livereload

# 3. Preview in browser
# http://localhost:4000
```

### Build Validation

```bash
# Build site
bundle exec jekyll build --trace

# Health check
bundle exec jekyll doctor
```

**Note**: Restart the dev server after modifying `_config.yml`.

## Project Structure

```
.
├── _data/
│   └── authors.yml           # Author database (centralized member info)
├── _posts/
│   └── YYYY/                 # Articles organized by year
│       └── YYYY-MM-DD-slug.md
├── _books/                   # Book metadata
│   └── book-slug.md
├── _chapters/                # Book chapters
│   └── book-slug-01.md
├── _layouts/                 # Page layout templates
├── assets/
│   ├── css/style.scss        # Global styles
│   └── images/               # Image assets
│       ├── authors/          # Author avatars
│       ├── posts/YYYY-MM-DD-slug/  # Article images (per article)
│       └── books/book-slug/  # Book images
├── _config.yml               # Site configuration
├── QUICK_START.md            # Quick operations guide
├── CONTRIBUTING.md           # Collaboration guidelines
└── CLAUDE.md                 # Project architecture docs
```

## Content Management

### Adding New Members

Edit `_data/authors.yml`:

```yaml
newmember:
  name: NewMember              # English name (page display)
  display_name: 昵称           # Chinese nickname (team page display)
  role: Job Description
  avatar: /assets/images/authors/newmember.jpg
  bio: Personal bio
  areas:
    - Skill 1
    - Skill 2
  focus: Current technical focus
  github: github_username      # Optional
```

### Adding New Articles

1. Create file: `_posts/2025/2025-MM-DD-title.md`
2. Fill in front matter:

```yaml
---
layout: post
title: "Article Title"
author: phli                   # Use key from authors.yml
date: 2025-11-20
categories: [Industrial Control]  # Choose from predefined categories
tags: [C#, .NET Core, Database]
excerpt: 1-2 sentence article summary
cover: /assets/images/posts/2025-11-20-title/cover.png  # Optional
---
```

**Predefined Categories**:
- Backend Development
- Frontend Development
- Embedded
- Data Engineering
- Industrial Control
- Tutorial

### Adding New Books

1. Create book: `_books/book-slug.md`

```yaml
---
title: Book Title
slug: book-slug               # URL identifier
author: zhuxi                  # Use key from authors.yml
description: Book description
cover: /assets/images/books/book-slug/cover.png  # Optional
order: 1                       # Sort order
---
```

2. Create chapters: `_chapters/book-slug-01.md`

```yaml
---
title: Chapter 1: Chapter Title
book: book-slug                # Must match book slug
order: 1                       # Chapter order
summary: Chapter summary
---
```

## Image Management

### Directory Structure

```
assets/images/
├── authors/           # Author avatars (e.g., zhuxi.jpg)
├── posts/
│   └── YYYY-MM-DD-slug/  # Separate directory per article
│       ├── cover.png
│       ├── diagram-flow.png
│       └── screenshot-result.png
└── books/
    └── book-slug/     # Separate directory per book
```

### Naming Conventions

- Cover: `cover.png`
- Diagrams: `diagram-{description}.png`
- Screenshots: `screenshot-{description}.png`
- **Forbidden**: `1.png`, `image.png`, `图片1.png`

See `assets/images/README.md` for details.

## Markdown Guidelines

- Start content body at `##` (h2), don't use `#` (h1)
- Code blocks must specify language: ` ```python `, ` ```c++ `
- Use absolute paths for images: `/assets/images/posts/...`
- Front matter uses `snake_case`: `reading_time`, `cover_alt`

## Git Commit Standards

```bash
# Format
[type] Brief description

# Type tags
[feat]     # New content (article, book, feature)
[fix]      # Bug fixes (links, format, typos)
[docs]     # Documentation updates
[style]    # Style adjustments
[refactor] # Refactoring
[chore]    # Miscellaneous

# Examples
git commit -m "[feat] Add article: Rust Ownership Explained"
git commit -m "[fix] Fix broken image links"
```

## Deployment

The project deploys automatically via GitHub Pages:

1. Push to `main` branch
2. GitHub Actions builds automatically
3. Publishes to `https://wzxzhuxi.github.io/`

**Note**:
- `_site/` directory is build output, excluded by `.gitignore`
- Never manually edit or commit the `_site/` directory

## Core Documentation

| Document | Description |
|----------|-------------|
| `QUICK_START.md` | Quick operations guide (add members/articles/books) |
| `CONTRIBUTING.md` | Collaboration guidelines (PR workflow, checklists) |
| `CLAUDE.md` | Project architecture and design principles |
| `ARCHITECTURE_IMPROVEMENTS.md` | Architecture improvement log |
| `assets/images/README.md` | Image guidelines |
| `_posts/POST_TEMPLATE.md` | Article template |

## Common Commands

```bash
# Development
bundle exec jekyll serve --livereload

# Build
bundle exec jekyll build --trace

# Check
bundle exec jekyll doctor

# View content
ls _posts/2025/          # List articles
ls _books/               # List books
ls _chapters/            # List chapters

# Search
grep -r "keyword" _posts/
```

## Tech Stack

- **Static Site Generator**: Jekyll 3.10+
- **Theme**: Minima (custom styles)
- **Code Highlighting**: Prism.js (Tomorrow Night theme)
- **Hosting**: GitHub Pages
- **Dependency Management**: Bundler

## Contributing

See `CONTRIBUTING.md` for:
- Fork/PR workflow
- Article submission template
- Image guidelines
- Local testing requirements
- Code review standards

## Team

Visit the [/team/](/team/) page to learn about team members.

## License

Content copyright © A_Bit_S Team.
