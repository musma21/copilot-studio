# Jekyll Developer Guide

This document explains how to develop and test this site locally using Jekyll.

---

## Component Versions

| Component | Version | Role |
|-----------|---------|------|
| **Ruby** | 4.0.3 | Runtime language Jekyll is built on |
| **Bundler** | 2.6.9 | Gem (package) dependency manager |
| **Jekyll** | 4.4.1 | Static site generator |
| **Kramdown** | 2.4.0 | Markdown-to-HTML parser used by Jekyll |
| **Liquid** | 4.0.4 | Templating engine used in layouts and pages |
| **Rouge** | 3.30.0 | Syntax highlighter for code blocks |

---

## Quick Start: Edit → Preview → Push

### Prerequisites (one-time setup)

**1. Install Ruby via Homebrew**
```bash
brew install ruby
```

**2. Add Homebrew Ruby to your PATH**

Add the following lines to `~/.bash_profile` (bash) or `~/.zshrc` (zsh):
```bash
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
export PATH="$(gem env gemdir)/bin:$PATH"
```

Apply immediately:
```bash
source ~/.bash_profile   # or source ~/.zshrc
```

Verify:
```bash
ruby --version   # should show 3.x or higher
```

**3. Install Bundler**
```bash
gem install bundler
```

**4. Install project gems**
```bash
cd /path/to/copilot-studio
bundle install
```

---

### Daily Workflow: Edit → Preview → Push

**Step 1 — Start the local server**
```bash
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
cd /path/to/copilot-studio
bundle exec jekyll serve --livereload --host 127.0.0.1
```

**Step 2 — Open the browser**

Navigate to:
```
http://127.0.0.1:4000/copilot-studio/
```

The `--livereload` flag automatically refreshes the browser whenever you save a file. No manual refresh needed.

**Step 3 — Edit a Markdown file**

Open any file under `_chapters/`, make your changes, and save.  
Jekyll detects the change, rebuilds `_site/`, and the browser reloads automatically.

Example:
```
_chapters/ws1-2-knowledge.md  →  save  →  _site/ rebuilt  →  browser reloads
```

**Step 4 — Verify the result**

Check the page in the browser. Confirm tables, images, and links render correctly.

**Step 5 — Stop the server**

Press `Ctrl+C` in the terminal.

**Step 6 — Commit and push**
```bash
git add -A
git commit -m "Your descriptive commit message"
git push
```

GitHub Pages will automatically rebuild and deploy the site from the updated `_chapters/` files.

**Step 7 — Verify GitHub Actions succeeded**

Go to the Actions tab and confirm the latest build passed:
```
https://github.com/musma21/copilot-studio/actions
```

Look for a green checkmark on the **"Deploy Jekyll site to Pages"** workflow run triggered by your push. If it fails, check the **build** job logs for errors before making further edits.

---

### Important: Liquid Template Conflicts

Jekyll uses the `{{ }}` and `{% %}` syntax (Liquid templating). If your Markdown content contains `{{variable}}` patterns (common in Copilot Studio prompt examples), Jekyll will try to interpret them and may produce warnings.

**Fix:** Wrap such blocks in `{% raw %}` / `{% endraw %}` tags:

````markdown
{% raw %}
{{보고서_제목}}
{{보고일}}
{% endraw %}
````

---

## Why These Components Are Needed

### Jekyll
Converts Markdown (`.md`) files and HTML templates into a complete static website. GitHub Pages runs Jekyll automatically on every push, so no server is required for hosting.

### Ruby
Jekyll is written in Ruby. The Ruby runtime must be installed to run Jekyll locally.

### Bundler
Manages gem (Ruby package) versions for the project. Without Bundler, gem version conflicts between projects can cause failures. `Gemfile` declares what you need; Bundler installs exactly those versions.

### Kramdown
Jekyll's default Markdown parser. It converts `.md` syntax into HTML. **Important:** Kramdown requires a blank line between a heading and a table. GitHub's renderer is more lenient, which is why a table may render correctly on GitHub.com but break on GitHub Pages.

### Liquid
The templating language embedded in layouts (`.html` files under `_layouts/`). It handles variables like `{{ page.title }}`, loops, and conditionals that build the sidebar, navigation, and page structure dynamically.

### Rouge
Syntax highlighter. Automatically adds color to fenced code blocks (` ```python `, ` ```bash `, etc.).

---

## Jekyll vs. Similar Technologies

| | **Jekyll** | **Hugo** | **MkDocs** | **Docusaurus** |
|---|---|---|---|---|
| **Language** | Ruby | Go | Python | JavaScript (React) |
| **Speed** | Moderate | Very fast | Fast | Moderate |
| **GitHub Pages** | Native support (no config) | Requires Actions | Requires Actions | Requires Actions |
| **Learning curve** | Low–Medium | Medium | Low | Medium–High |
| **Theming** | Gem-based themes | Rich theme ecosystem | Material theme | React components |
| **Best for** | Blogs, docs, GitHub Pages | Large sites, speed-critical | Pure documentation | Developer portals |
| **Markdown parser** | Kramdown (strict) | Goldmark (lenient) | Python-Markdown | MDX |

### Jekyll Advantages
- **Zero-config GitHub Pages**: Push Markdown → GitHub builds and deploys automatically. No GitHub Actions needed.
- **Mature ecosystem**: Large library of themes and plugins (gems).
- **Simple mental model**: Markdown in `_chapters/` → HTML in `_site/` → served at your URL.

### Jekyll Disadvantages
- **Ruby dependency**: Requires a compatible Ruby version locally (version mismatch is a common pain point).
- **Kramdown strictness**: Requires blank lines before tables and certain block elements, unlike GitHub's GFM renderer.
- **Build speed**: Slower than Hugo for large sites (100+ pages).
- **Liquid conflicts**: Prompt templates using `{{ }}` syntax need `{% raw %}` wrapping.

---

## Local vs. GitHub Actions Environment Isolation

### Background

Jekyll's `Gemfile.lock` records the exact gem versions **and the OS platform** resolved during `bundle install`. When this file is committed from macOS (`arm64-darwin`), GitHub Actions (running on `ubuntu-latest` / `x86_64-linux`) cannot use it — bundler exits with an error and the deployment fails.

### Applied Fix

The following two measures have been applied to permanently prevent this conflict:

| Measure | File | Effect |
|---------|------|--------|
| `Gemfile.lock` added to `.gitignore` | `.gitignore` | Lock file is never committed; each environment generates its own |
| `Gemfile.lock` removed from git tracking | `git rm --cached` | Existing entry deleted from repository history |

### How Each Environment Now Works

| | **Local (macOS / arm64-darwin)** | **GitHub Actions (ubuntu-latest / x86_64-linux)** |
|---|---|---|
| **Lock file** | Generated locally on `bundle install`, not committed | Generated fresh on each Actions run |
| **Platform** | `arm64-darwin` | `x86_64-linux` |
| **Interference** | None — file stays local only | None — file is generated in the runner |

### Rule

> **Never commit `Gemfile.lock`.**  
> It is listed in `.gitignore` for this reason. If you accidentally generate it and `git status` shows it, it will be ignored automatically.

---

## Project File Structure

```
copilot-studio/
├── _chapters/          # Source Markdown files (edit these)
├── _layouts/           # HTML templates (default.html, chapter.html)
├── _site/              # Generated output (auto-created by Jekyll, do not edit manually)
├── assets/             # CSS, images, sample data
├── _config.yml         # Jekyll site configuration
├── Gemfile             # Ruby gem dependencies
├── .gitignore          # Excludes Gemfile.lock and macOS artifacts from git
└── index.html          # Home page (Liquid template)
```

> **Rule of thumb:** Only edit files in `_chapters/`, `_layouts/`, `assets/`, `_config.yml`, and `index.html`.  
> Never manually edit `_site/` — it is overwritten on every Jekyll build.
