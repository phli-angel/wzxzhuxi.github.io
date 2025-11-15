# Repository Guidelines

## Project Structure & Module Organization
Source content lives in `_posts/` (dated Markdown posts), `index.html`, and standalone pages like `about.md` and `team.md`. Layout primitives are in `_layouts/`, reusable fragments in `_includes/`, and shared styles in `assets/css/style.scss`. `_config.yml` controls site metadata, navigation, and plugins; edit it before adding new collections or URL changes. `_site/` is build output and should never be edited manually—treat it as disposable. Keep images or downloadable assets under `assets/` so the Jekyll asset pipeline can fingerprint them consistently.

## Build, Test, and Development Commands
Use `bundle install` once to sync Ruby gems declared in `Gemfile`. Run `bundle exec jekyll serve --livereload` to build the site, watch for changes, and serve on `http://127.0.0.1:4000`. Use `bundle exec jekyll build` for a production-equivalent build; pass `JEKYLL_ENV=production` when validating deployment assets.

## Coding Style & Naming Conventions
Write Markdown using 80–100 character lines where feasible and prefer reference-style links for readability. Liquid tags should be indented two spaces inside loops/conditionals, mirroring existing layouts. Posts must follow the `YYYY-MM-DD-title.md` naming pattern and include front matter for `title`, `layout`, and any custom variables referenced in `_layouts`. SCSS follows BEM-style class names and nests selectors only two levels deep. Global configuration keys in `_config.yml` use snake_case to match current usage.

## Testing Guidelines
Every change should pass `bundle exec jekyll build --trace` with zero warnings. After building, spot-check updated pages inside `_site/` in a browser to verify Liquid output and asset paths. Run `bundle exec jekyll doctor` before opening a pull request to catch malformed permalinks or missing assets. When modifying styling, test both desktop and mobile breakpoints using the browser dev tools responsive mode.

## Commit & Pull Request Guidelines
Commits should be small, focused, and use imperative subject lines such as `Add hero section layout` so the history reads like instructions. Reference related issue IDs inside the body when applicable. Pull requests need: a clear summary of changes, screenshots or GIFs for visual updates, and a checklist confirming `jekyll build` and `jekyll doctor` were executed. Tag reviewers responsible for content and for code whenever both change. Merge only after at least one approval to keep the main branch deployable at all times.
