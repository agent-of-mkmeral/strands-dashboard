# AGENTS.md

This document provides context, patterns, and guidelines for AI coding assistants working in this repository.

## Project Overview

**Strands Dashboard** is a client-side web dashboard for monitoring and managing autonomous [Strands Agents](https://github.com/strands-agents) running via GitHub Actions. It is hosted on GitHub Pages and requires no backend — all data is fetched directly from the GitHub API using a personal access token stored in the browser's `localStorage`.

**Live URL:** https://agent-of-mkmeral.github.io/strands-dashboard/

**Owner:** [@mkmeral](https://github.com/mkmeral) via [agent-of-mkmeral](https://github.com/agent-of-mkmeral)

## Directory Structure

```
strands-dashboard/
│
├── docs/                    # GitHub Pages root (served via Settings → Pages → /docs)
│   ├── index.html           # Main dashboard (single-page app, all tabs)
│   └── activity.html        # Standalone agent activity page (public PRs/issues)
│
├── AGENTS.md                # This file
└── README.md                # (not yet created)
```

### Key Architectural Decisions

- **Single-file HTML apps** — Each page is a self-contained HTML file with inline CSS and JavaScript. No build step, no bundler, no npm. This keeps deployment trivial (just push to `main`).
- **Client-side only** — No server, no database. All state lives in `localStorage`. GitHub API calls happen directly from the browser.
- **GitHub Pages deployment** — Served from the `docs/` folder on the `main` branch. Changes go live ~1 minute after push.

## Pages and Tabs

### `docs/index.html` — Main Dashboard

A single-page application with tab-based navigation. Tabs:

| Tab | ID | Description |
|---|---|---|
| 📊 Dashboard | `dashboard` | Overview stats: open issues, running actions, project board (via GitHub Projects v2 GraphQL API) |
| 📋 Issues | `issues` | List all issues in the configured repo, click to view detail + linked action runs |
| ⚡ Actions | `actions` | GitHub Actions workflow runs, with log streaming and step-level detail |
| 📊 Activity | `activity` | Agent activity throttle status, external vs internal action counts |
| 🌍 Wild | `wild` | Open PRs and issues created by the agent in **external** public repos (excludes `agent-of-mkmeral` and `mkmeral` owned repos) |
| 📈 Traces | `traces` | Links to Amazon Bedrock observability console (URL loaded from repo variable `TRACES_URL`) |
| 🤖 Agent | `agent` | View/edit agent configuration: system prompt, model, tools, knowledge base ID. Loads from GitHub Actions variables/secrets |
| 📅 Schedule | `schedule` | View/edit the agent's scheduled jobs (cron-based). Reads/writes the `AGENT_SCHEDULE` repo variable |
| ⚙️ Settings | `settings` | Configure GitHub token, repo, workflow, project ID, traces URL. Import/export via encrypted share links |

### `docs/activity.html` — Standalone Activity Page

A separate page showing the agent's open PRs and issues across public repositories. Features:
- Summary cards at top (Open PRs, Open Issues, Repos, Total)
- Filter by PRs or Issues
- Grouped by repository
- Uses GitHub Search API: `is:public author:agent-of-mkmeral`
- Settings panel for token, username, repo exclusions

## Tech Stack

- **HTML5 / CSS3 / Vanilla JavaScript** — No frameworks
- **Libraries (CDN):**
  - [Inter font](https://fonts.google.com/specimen/Inter) — Typography
  - [highlight.js](https://highlightjs.org/) — Code syntax highlighting
  - [marked.js](https://marked.js.org/) — Markdown rendering
  - [Mermaid](https://mermaid.js.org/) — Diagram rendering
- **APIs:**
  - [GitHub REST API v3](https://docs.github.com/en/rest) — Issues, Actions, Variables, Secrets
  - [GitHub GraphQL API v4](https://docs.github.com/en/graphql) — Projects v2, rich queries
  - [GitHub Search API](https://docs.github.com/en/rest/search) — Finding agent activity across repos

## Configuration

All configuration is stored in `localStorage` and managed via the Settings tab:

| Key | localStorage Key | Description |
|---|---|---|
| GitHub Token | `gh_token` | Personal access token with `repo` scope |
| Repository | `gh_repo` | Target repo (e.g., `agent-of-mkmeral/strands-coder`) |
| Workflow | `gh_workflow` | Workflow filename (e.g., `agent.yml`) |
| Project ID | `gh_project_id` | GitHub Projects v2 node ID (e.g., `PVT_kwHOAIyVLs4BMSWO`) |
| Traces URL | `traces_url` | URL for Bedrock observability console (auto-loaded from repo variable `TRACES_URL`) |

### Auto-Loading from Repo Variables

When the user clicks **Load** on the Agent tab, the dashboard fetches GitHub Actions variables from the configured repo and auto-populates:
- `SYSTEM_PROMPT` (or falls back to `SYSTEM_PROMPT.md` file)
- `STRANDS_MODEL_ID`
- `STRANDS_TOOLS`
- `STRANDS_KNOWLEDGE_BASE_ID`
- `TRACES_URL`
- `AGENT_SCHEDULE`

### Share/Import

Settings can be exported as an encrypted URL fragment for sharing between devices. Uses Web Crypto API (AES-GCM) for encryption.

## Design System

- **Theme:** Dark mode only (`--bg: #09090b`)
- **Accent color:** Indigo (`--accent: #6366f1`)
- **Font:** Inter (400, 500, 600, 700)
- **Border radius:** `--radius: 14px` (cards), `--radius-sm: 10px` (buttons)
- **Cards:** Semi-transparent white (`rgba(255, 255, 255, 0.03)`) with subtle borders
- **Animations:** `fadeIn` on page transitions, hover effects on cards
- **Mobile:** Responsive, PWA-capable meta tags

### CSS Custom Properties

All theme values are defined as CSS custom properties on `:root` in each HTML file. When making visual changes, use these variables — don't hardcode colors.

Key variables: `--accent`, `--bg`, `--bg-card`, `--border`, `--text`, `--text-secondary`, `--text-muted`, `--radius`, `--success`, `--warning`, `--danger`

## Development Guidelines

### Making Changes

1. Clone the repo
2. Edit files in `docs/`
3. Test locally by opening `docs/index.html` in a browser (configure a GitHub token in Settings first)
4. **Validate JavaScript syntax** before pushing:
   ```bash
   # Extract JS and validate (no syntax errors)
   sed -n '/<script>/,/<\/script>/p' docs/index.html | sed '1d;$d' > /tmp/check.js
   node --check /tmp/check.js
   ```
5. Commit and push to `main` — GitHub Pages deploys automatically

### Common Pitfalls

- **String quoting in `innerHTML`**: When building HTML strings in JS that contain `onclick` handlers with string arguments, use backtick template literals (`` ` ``) for the outer string to avoid quote conflicts:
  ```javascript
  // ✅ Good — backtick outer, double quotes for HTML, single quotes for JS
  container.innerHTML = `<button onclick="showPage('settings')">Go</button>`;
  
  // ❌ Bad — single quote collision
  container.innerHTML = '<button onclick="showPage(\'settings\')">Go</button>';
  ```
- **GitHub Pages cache**: Changes may take 1-2 minutes to propagate after push. Hard-refresh (`Ctrl+Shift+R`) if testing.
- **CORS**: GitHub API calls work fine from GitHub Pages. No CORS issues for `api.github.com`.
- **Token scopes**: The GitHub token needs `repo` scope for private repo variables/secrets, and `read:org` if working with organization projects.

### Adding a New Tab

1. Add a `<button>` to the nav bar (around line 396-404 in `index.html`)
2. Add a `<div class="page" id="{name}Page">` in the pages section
3. Add the page ID to the valid pages array (around line 825)
4. Add a `load{Name}()` function and wire it into `showPage()` (around line 839-854)
5. Validate JS syntax with `node --check`

### Code Style

- **Vanilla JS** — No TypeScript, no JSX, no modules. Everything is in `<script>` tags.
- **Functions** — Use `async function name()` for API calls, regular `function name()` for rendering.
- **Naming** — `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants.
- **Error handling** — Wrap API calls in try/catch, show user-friendly errors via `showToast()`.
- **DOM updates** — Use `innerHTML` for rendering lists/cards, `getElementById` for individual elements.

## Security Considerations

- **No secrets in this repo** — This is a public repository. All sensitive data (tokens, URLs) lives in `localStorage` or in private repo variables.
- **Tokens** — GitHub tokens are stored only in the browser's `localStorage`. They are never transmitted to any server other than `api.github.com`.
- **Encrypted sharing** — The share/import feature encrypts settings with a user-provided password using AES-GCM before putting them in the URL fragment (which is never sent to the server).
- **TRACES_URL** — The Bedrock observability URL is stored as a variable on the **private** repo (`strands-coder-private`), not here. The dashboard auto-loads it via API.

## Relationship to Other Repos

| Repo | Relationship |
|---|---|
| `agent-of-mkmeral/strands-coder-private` | Private repo where the agent runs. Dashboard monitors this repo's issues, actions, variables. `TRACES_URL` is stored here. |
| `agent-of-mkmeral/strands-coder` | Public agent repo (legacy). Dashboard defaults point here. |
| `strands-agents/sdk-python` | The Strands Agents SDK that powers the agent. |
| `strands-agents/tools` | Tool implementations used by the agent. |

## Things to Do

- Validate JS syntax (`node --check`) before every commit
- Use CSS custom properties for all colors and spacing
- Keep each page self-contained in its `load{Name}()` / `render{Name}()` function pair
- Test on mobile (the dashboard is PWA-capable)
- Use `showToast()` for user-facing success/error messages

## Things NOT to Do

- Don't add a build step or bundler — keep it zero-dependency
- Don't store secrets or sensitive URLs in this public repo
- Don't use frameworks (React, Vue, etc.) — vanilla JS only
- Don't break the single-file-per-page pattern
- Don't push without validating JS syntax first
- Don't hardcode colors — use CSS custom properties
