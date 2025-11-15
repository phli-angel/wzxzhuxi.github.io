# Jekyll Site

## Overview
This repository hosts a Jekyll-powered static site. Content pages (`index.html`, `about.md`, `team.md`) and dated posts under `_posts/` render through layouts in `_layouts/` and snippets in `_includes/`. Styles are authored in `assets/css/style.scss`, while `_config.yml` defines metadata, navigation, and plugin settings. Generated output appears inside `_site/` and should be treated as disposable.

## Getting Started
1. Install Ruby 3.x, Bundler, and make sure `node` is available for asset tooling.
2. Run `bundle install` to install dependencies listed in `Gemfile`.
3. Start the dev server with:
   ```bash
   bundle exec jekyll serve --livereload
   ```
   The site will be available at `http://127.0.0.1:4000`.

## Development Workflow
- **Content editing**: add Markdown posts named `YYYY-MM-DD-title.md` inside `_posts/`. Each file must include YAML front matter with `layout`, `title`, and any custom variables used in layouts.
- **Styling**: modify `assets/css/style.scss`; the file supports SCSS nesting and is compiled automatically by Jekyll.
- **Configuration**: update `_config.yml` when changing navigation, URL structures, or enabling plugins. Restart `jekyll serve` after edits to this file.
- **Validation**: run `bundle exec jekyll build --trace` before opening a pull request. For sanity checks, run `bundle exec jekyll doctor` to catch broken permalinks or missing assets.

## Deployment
Deployment depends on the environment (GitHub Pages, custom CI, etc.). Regardless of the target, deploy the contents of `_site/` produced by `bundle exec jekyll build JEKYLL_ENV=production`. Never commit `_site/` to version control.

## Contributing
Follow the workflows documented in `AGENTS.md` for branching, testing, and pull-request expectations. Keep commits small, use imperative subject lines (e.g., `Add hero section layout`), and include screenshots for visual updates. Ensure `jekyll build` and `jekyll doctor` both succeed locally before requesting a review.
