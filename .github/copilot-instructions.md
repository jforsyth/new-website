# al-folio Academic Website - AI Agent Instructions

This is an **al-folio** Jekyll-based academic portfolio website. Many users are academics without coding experience.

## Architecture Overview

**Static site generator:** Jekyll 4.x with Liquid templating  
**Build system:** Ruby/Bundler + Node/npm (Prettier)  
**Deployment:** GitHub Pages via GitHub Actions (auto-deploys from `main` branch to `gh-pages`)  
**Development:** Docker (recommended) or local Ruby setup

### Key Components

- **Content Collections:** `_posts/` (blog), `_projects/`, `_news/`, `_books/`, `_teachings/`
- **Data Files:** `_data/*.yml` (cv.yml, socials.yml, repositories.yml, coauthors.yml, venues.yml)
- **Publications:** `_bibliography/papers.bib` (BibTeX format) processed by jekyll-scholar plugin
- **Layouts/Templates:** `_layouts/*.liquid` and `_includes/*.liquid` (Liquid templating)
- **Styling:** `_sass/` (SCSS) with theming in `_sass/_themes.scss`, `_sass/_variables.scss`
- **Assets:** `assets/img/`, `assets/pdf/`, `assets/json/resume.json` (JSON Resume format)

### Critical Configuration

**`_config.yml`** is the central configuration hub:
- Site metadata (title, author, description, url, baseurl)
- Feature toggles (dark mode, search, comments, analytics)
- Jekyll-scholar settings (author highlighting, bibliography style)
- Plugin configuration (30+ Jekyll plugins including jekyll-scholar, jekyll-archives-v2, jekyll-paginate-v2)
- Collection definitions (books, news, projects, teachings)

**Important:** Changes to `_config.yml` require a full rebuild. All other content changes are live-reloaded during development.

## Developer Workflows

### Running Locally

**Docker (recommended):**
```bash
docker compose pull
docker compose up
# Site available at http://localhost:8080 with live reload
```

**Legacy (requires Ruby 2.x-3.x, Bundler, Python, pip):**
```bash
bundle install
pip install jupyter  # if using Jupyter notebooks
bundle exec jekyll serve
# Site available at http://localhost:4000
```

**Entry point:** `bin/entry_point.sh` manages Gemfile.lock, starts Jekyll server, and auto-restarts on `_config.yml` changes.

### Building for Production

```bash
# Docker
docker compose up --build

# Legacy
bundle exec jekyll build  # Output: _site/
```

### Code Formatting

```bash
npx prettier . --write  # Format Liquid, Markdown, YAML, JS, SCSS
```

### Deployment

Automated via `.github/workflows/deploy.yml`:
1. Push to `main` branch
2. GitHub Action runs `bundle exec jekyll build`
3. Deploys to `gh-pages` branch
4. GitHub Pages serves from `gh-pages`

**Manual deployment:** `bin/deploy` script (advanced users)

## Project-Specific Conventions

### Content File Patterns

**Blog posts:** `_posts/YYYY-MM-DD-title.md`
```yaml
---
layout: post
title: Post Title
date: YYYY-MM-DD HH:MM:SS
description: Brief description
tags: [tag1, tag2]
categories: [category1]
---
```

**Projects:** `_projects/*.md`
```yaml
---
layout: page
title: Project Name
description: Project description
img: assets/img/project.jpg
importance: 1  # Display order (lower = higher priority)
category: work  # or "fun"
related_publications: true  # Links to relevant BibTeX entries
---
```

**News:** `_news/*.md` (inline announcements, no title needed)

### Publications (BibTeX)

Publications in `_bibliography/papers.bib` use **custom frontmatter keywords** for enhanced display:
- `pdf`, `supp` (supplementary material), `slides`, `poster`, `code`, `website`
- `abstract`, `preview` (image), `selected` (highlight on homepage)
- `abbr` (venue abbreviation), `award`, `award_name`
- `altmetric`, `dimensions` (badge IDs), `google_scholar_id`, `inspirehep_id`

**Author highlighting:** Configure in `_config.yml` under `scholar:last_name` and `scholar:first_name` to bold your name in author lists.

**Venue/coauthor enrichment:** Add venue URLs in `_data/venues.yml` and coauthor links in `_data/coauthors.yml`.

### Custom Plugins

**Ruby plugins in `_plugins/`:**
- `external-posts.rb` – Fetches blog posts from external RSS feeds (Medium, etc.) configured in `_config.yml` under `external_sources`
- `google-scholar-citations.rb` – Updates citation counts from Google Scholar
- `inspirehep-citations.rb` – Updates citation counts from INSPIRE-HEP
- `hide-custom-bibtex.rb` – Filters internal BibTeX keywords from display (see `filtered_bibtex_keywords` in `_config.yml`)
- `remove-accents.rb`, `details.rb`, `file-exists.rb` – Utility plugins

### Liquid Includes

Reusable components in `_includes/`:
- `news.liquid`, `selected_papers.liquid`, `latest_posts.liquid` – Homepage sections
- `figure.liquid`, `video.liquid`, `audio.liquid` – Media embedding with cache busting
- `citation.liquid` – Renders BibTeX citations
- `calendar.liquid` – Google Calendar integration
- `cv/*.liquid`, `repository/*.liquid` – Specialized includes for CV and repository pages

### CSS Architecture

**Theme system:** Light/dark modes via CSS custom properties in `_sass/_themes.scss`  
**Primary theme color:** `--global-theme-color` variable (default: various blues)  
**Per-page styles:** `_sass/_cv.scss`, `_sass/_distill.scss` (for Distill-style posts)

### Data-Driven Content

**CV:** Prefers `assets/json/resume.json` (JSON Resume schema). Falls back to `_data/cv.yml` (YAML) if JSON missing.  
**Social links:** `_data/socials.yml` (username/URL pairs for GitHub, Twitter/X, LinkedIn, Google Scholar, etc.)  
**Repositories:** `_data/repositories.yml` (GitHub usernames/repos for stats display)

## Integration Points

### Jekyll-Scholar

BibTeX processing configured in `_config.yml` under `scholar:`:
- `source`, `bibliography` – Where to find `.bib` files
- `style`, `locale` – Citation style (APA by default)
- `bibtex_filters` – Preprocessing filters (latex, smallcaps, superscript)
- `query`, `group_by`, `group_order` – Control publication display

### External Services

**Analytics:** Google Analytics, Cronitor, Pirsch, Openpanel (configure IDs in `_config.yml`)  
**Comments:** Giscus (GitHub Discussions) or Disqus (deprecated)  
**Search:** Built-in search via `lunr.js` (enable with `search_enabled: true`)  
**Badges:** Altmetric, Dimensions, Google Scholar, INSPIRE-HEP (enable in `enable_publication_badges`)

### GitHub Actions

**`.github/workflows/deploy.yml`** – Main deployment workflow (builds and deploys to `gh-pages`)  
**`.github/workflows/axe.yml`** – Accessibility testing  
**`.github/workflows/broken-links-site.yml`** – Link checker  
**`.github/workflows/deploy-docker-tag.yml`, `deploy-image.yml`** – Docker image builds

## Important Differences from Typical Jekyll Sites

1. **Two-phase build:** Content → Jekyll → static site. Changes to `_config.yml` require restart; other changes are live-reloaded.
2. **BibTeX-first publications:** Use `papers.bib` with custom keywords, not Markdown files.
3. **External post ingestion:** Can pull blog posts from Medium/RSS feeds via `external-posts.rb`.
4. **Docker-first development:** Official setup uses Docker, not local Ruby (though both work).
5. **Dual CV formats:** Supports both JSON Resume (structured) and YAML (flexible) formats.
6. **GitHub Pages Actions deployment:** Not classic GitHub Pages build; uses custom Action for full control over plugins and build process.

## Common Tasks

**Add a blog post:** Create `_posts/YYYY-MM-DD-slug.md` with frontmatter, write in Markdown.  
**Add a publication:** Insert BibTeX entry in `_bibliography/papers.bib`, add `pdf:`, `code:`, etc. fields.  
**Change theme color:** Edit `--global-theme-color` in `_sass/_themes.scss`.  
**Update CV:** Edit `assets/json/resume.json` (JSON Resume) or `_data/cv.yml` (YAML).  
**Add a project:** Create `_projects/project-name.md` with frontmatter and content.  
**Configure social links:** Edit `_data/socials.yml` (add/remove platforms and usernames).  
**Enable/disable features:** Toggle settings in `_config.yml` (dark mode, search, comments, analytics).

## References

- **Documentation:** README.md (overview), INSTALL.md (setup), CUSTOMIZE.md (detailed guide), FAQ.md (troubleshooting)
- **Agent docs:** `.github/agents/customize.agent.md` (customization patterns), `.github/agents/docs.agent.md` (documentation standards)
- **Upstream:** [al-folio repository](https://github.com/alshedivat/al-folio) (community issues/discussions)
- **JSON Resume:** [jsonresume.org](https://jsonresume.org) (CV format schema)
- **Jekyll:** [jekyllrb.com](https://jekyllrb.com) (framework documentation)

## Writing for This Codebase

**Target audience:** Many users are academics with limited coding experience.  
**Tone:** Clear, patient, jargon-free. Explain technical terms when first used.  
**File changes:** Always update both code and relevant documentation (CUSTOMIZE.md, FAQ.md).  
**Testing:** Use Docker setup to verify changes locally before committing.  
**Formatting:** Run Prettier before committing (`npx prettier . --write`).
