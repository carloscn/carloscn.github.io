# AGENTS.md — Carlos' Navigator (GitHub Pages)

Guidance for AI agents and humans maintaining this personal navigation site.

## New agent — read this first

Open this repo and treat this file as the source of truth before editing.

**Product:** Static personal URL navigation homepage for Carlos.  
**Live host:** GitHub Pages from public repo `carloscn.github.io` (custom domain may exist via `CNAME`).  
**Stack:** Plain HTML + CSS + shared `data.js` — **no build step, no framework**.  
**Brand title:** `Carlos' Navigator` (stored in `data.js` → `window.NAV_DATA.title`).  
**Default branch / deploy:** `master` → GitHub Pages. Commit/push **only when the user asks** (提交 / 上传 / 上线).

### Suggested resume prompt

```text
Open carloscn.github.io. Read AGENTS.md and follow it.
Continue from current master / working tree.
(Describe the task here.)
Commit and push only if I say 提交/上传/上线.
```

### Do not break these without an explicit request

- Keep the **three views** (icon default / text / compact-icon).
- **Default site root (`index.html`) is the full icon view (有图).**
- Keep **text view free of item logos**.
- Keep **privacy rules** (`obf1:`, never commit internal hosts/plaintext secrets).
- Prefer **simple, green-accent** UI — avoid flashy redesigns unless asked.

---

## Views (shared `data.js`)

| File | Role |
|------|------|
| `index.html` | **Default homepage (有图)** — full hao268-style layout; logos on the first group (**首页 / 常用** hot grid) only. Category rows stay text-only. |
| `text.html` | **Pure text view** — same layout/features as `index.html`, but **does not render item logos**. |
| `icons.html` | **Legacy redirect** → `index.html` (keep for old bookmarks). Do not put a second full copy here. |
| `icons-lite.html` | **Compact icon view** — classic tile grid + avatar; lighter chrome. |
| `data.js` | Single shared `window.NAV_DATA` for **all** views. |

Cross-links (keep working):

- Full icon ↔ text: `index.html` ↔ `text.html`
- Compact icon ↔ others: `icons-lite.html` ↔ `index.html` / `text.html`
- `icons.html` → `index.html` (redirect only)

### Sync rules (important)

When changing layout/UX on the full homepage:

1. **`index.html` and `text.html` are nearly the same page.** Prefer editing both, or copy carefully. Differences should stay limited to: labels (“有图” vs “纯文字”), toggle links, and whether `makeLink(..., withIcon)` draws logos on 常用.
2. Keep in sync across entry pages: `resolveUrl` + `OBF_KEY` (`cn-nav-k7`), theme boot script / `cn-nav-theme`, and `data.js?v=...` cache-bust (bump **all** HTML files that load `data.js`).
3. `icons-lite.html` only needs sync for: `OBF_KEY` / `resolveUrl`, theme key, `data.js?v=...`, and cross-links — not the full hao268 layout.
4. Do not reintroduce a full duplicate page at `icons.html`; keep it as a redirect to `index.html`.

---

## Current UX / design notes (as of 2026-08)

Use these as defaults when polishing UI:

| Area | Current behavior |
|------|------------------|
| Brand | Title **Carlos' Navigator**; brand `<h1>` uses **JetBrains Mono** (Google Fonts). |
| Theme | Light/dark toggle in chrome; `localStorage` key `cn-nav-theme`; early `<head>` script avoids flash; falls back to `prefers-color-scheme`. |
| Top bar | Left: site label + weather snippet; center: **today’s date** (zh-CN, green); right: theme + city + view switches. |
| World clocks | Madrid / Beijing / LA — **HH:MM only** (no seconds); day/night tint. |
| Category labels | Soft **green** background (`--green-soft` / `--cat`), not amber/orange. |
| Visual tone | Flat, compact, green accents; text-first. Avoid heavy cards, purple gradients, loud shadows, decorative images. |
| Icons | On `index.html` 常用 grid and throughout `icons-lite.html` tiles. Text view has none. Missing logos fail soft (letter / no image). |

CSS tokens on the full pages live in `:root` / `html[data-theme="dark"]` (e.g. `--green`, `--surface`, `--text`, `--line`). Prefer variables over new hardcoded colors.

---

## Key files

| Path | Role | Committed? |
|------|------|------------|
| `index.html` | **Default** full layout + logos on 常用 | Yes |
| `text.html` | Full layout, text-only (no item logos) | Yes |
| `icons.html` | Redirect to `index.html` | Yes |
| `icons-lite.html` | Compact tile icon view | Yes |
| `data.js` | Public navigation data | Yes |
| `assets/logos/*` | Item logos | Yes |
| `assets/pic/headpic.jpg` | Avatar / favicon | Yes |
| `AGENTS.md` | This guide | Yes |
| `data.company.local.js` | Optional local plaintext company links (`window.NAV_COMPANY`) | **No** (`.gitignore`) |
| `*.infinity`, `link*.json` | Local backups | **No** (`.gitignore`) |

Do **not** commit ignored local backup files.

---

## Hard rules (privacy / public repo)

1. **This is a public repository.** Anything committed is searchable on GitHub.
2. **Never write sensitive / internal hostnames or paths as plaintext** in committed files (`data.js`, HTML entry points, this `AGENTS.md`, commit messages, PR text, comments).
3. **Never paste real internal/company URLs into agent prompts that will be logged into docs you plan to commit.** Prefer live chat only, or gitignored local files.
4. **Do not use short-link services** for internal entries. Use in-page `obf1:` scramble.
5. **Do not add company/internal URL inventories to this file.** Describe *how*, not *what*.
6. Public, non-sensitive URLs may stay as normal `https://...` in `data.js`.

---

## URL scramble (`obf1:`)

Internal or sensitive URLs in `data.js` must use `obf1:` so casual GitHub text search does not hit the raw host/path.

### Format

- Stored value: `obf1:` + Base64( XOR(plaintext_utf8, key) )
- Key (must match **all** HTML entry points): `cn-nav-k7`
- Runtime: `resolveUrl()` in each page reverses this for `href`.

### Encode (Python)

```python
import base64

KEY = b"cn-nav-k7"

def obf1(url: str) -> str:
    raw = url.encode("utf-8")
    xored = bytes(raw[i] ^ KEY[i % len(KEY)] for i in range(len(raw)))
    return "obf1:" + base64.b64encode(xored).decode("ascii")

# Example only (public dummy host — not a real site entry):
# print(obf1("https://example.com/path"))
# -> obf1:CxpZHhJMAkRSGw9AHg0TAwhYDkFdDxUe
```

### Decode check (Python)

```python
import base64

KEY = b"cn-nav-k7"

def deobf1(s: str) -> str:
    assert s.startswith("obf1:")
    xored = base64.b64decode(s[5:])
    return bytes(xored[i] ^ KEY[i % len(KEY)] for i in range(len(xored))).decode("utf-8")
```

### After editing `data.js`

- Confirm **no** sensitive host fragments appear as plaintext in the file.
- Bump the cache-bust query on **all** HTML entry points, e.g.  
  `data.js?v=20260808a` → `data.js?v=20260808b`  
  in `index.html`, `text.html`, and `icons-lite.html` (not needed on redirect-only `icons.html`)
  (also update the comment at the top of `data.js` if present).

This scramble is **not strong crypto** (key is in the page). It only reduces accidental discovery via repository search.

---

## Data shape

```js
window.NAV_DATA = {
  title: "Carlos' Navigator",
  groups: [
    {
      name: "分组名",
      items: [
        { name: "显示名", url: "https://...", logo: "assets/logos/....ext" },
        // or: url: "obf1:...."
      ]
    }
  ]
};
```

- First group is **首页** (常用 hot grid).
- `logo` paths are relative to site root; used by `index.html` (常用 only) and `icons-lite.html`. Text view ignores logos but keeps the field.
- Prefer existing logos under `assets/logos/`; if missing, use `""` or a fitting existing icon — do not invent fake assets.
- Typical logos: `gitlab.webp` for code forges, `wiki.webp` for wiki docs, `github.png` for GitHub, etc.

### Display name conventions

| Kind | Rule | Examples |
|------|------|----------|
| Global brands | Official spelling / casing | Reddit, Instagram, ChatGPT, LinkedIn, YouTube |
| Chinese sites | Common short Chinese names | 淘宝、携程、豆瓣、QQ 邮箱 |
| Multi-word English | Spaces + Title Case (or official form) | Flight Connections, Submarine Cable Map |
| Internal labels | Readable Title Case; no `snake_case` / raw host dumps | Thor Wiki, XCU Platform |
| Avoid | Domain-like dumps, random ALL CAPS, mixed `ChatGpt` | use ChatGPT not ChatGpt |

---

## Company group ordering

Group name **公司**. Keep this order:

1. **Code-related** first (repos / forge roots / code projects).
2. **Wiki / docs** next.
3. **Everything else** last (chat, office, expense tools, personal public repos, …).

Do not invent a second public company section. Optional local plaintext mirror: gitignored `data.company.local.js` (`window.NAV_COMPANY`) — keep in sync when convenient; never commit it.

---

## Public group guidance (non-company)

Prefer existing groups:

| Group | Typical contents |
|-------|------------------|
| 首页 | Daily high-frequency tiles (first group; denser grid) |
| 社交媒体 | Portals, social, video, music, consumer sites |
| OABA | Banks, brokers, tax/gov, telecom, personal admin |
| 旅行 | Maps, airlines, booking, flight tools |
| 小工具 | Utilities / one-off tools |
| 办公工具 | Productivity / AI / cloud office |
| 技术 | Engineering docs and tech bookmarks |
| 项目管理 | Work platforms / infra consoles |
| 外语学习 | Language learning |
| 书籍知识 | Books / long-form reading |
| 收藏 | Misc personal saves |

Do not invent parallel groups for the same purpose without user request.

---

## Workflow for “add these links”

1. Read current `data.js` target group(s).
2. Classify each item into an existing group (company: code / wiki / other).
3. Encode sensitive URLs with `obf1:`; leave clearly public URLs plain.
4. Apply display-name conventions; insert into correct position within group order.
5. Choose an existing logo path (or `""` if none).
6. Bump `data.js?v=...` in **`index.html`**, **`text.html`**, and **`icons-lite.html`**.
7. Grep the working tree for accidental plaintext sensitive hosts before commit.
8. Commit and push **only when the user asks**.

### Suggested user prompt (add company links)

```text
Clone/open carloscn.github.io. Read AGENTS.md and follow it.
Add to the 公司 group (obf1: for internal URLs; order Code → Wiki → other):
- name: ...
  url: ...
  type: code | wiki | other
Then commit and push if I say 提交/上传/上线.
```

(Users supply actual URLs only in the chat task — not by editing this file.)

---

## Git / deploy

- Default branch: `master` (GitHub Pages deploys from here).
- Prefer clear English commit messages describing *behavior*, without listing internal hosts.
- If remote advanced, pull/rebase then push; do not force-push unless explicitly requested.
- Do not remove `CNAME` / custom domain unless the user asks or deploy is blocked and they approve.
- Feature branches OK for larger UI work; merge to `master` when the user asks to go live.

---

## Out of scope / avoid

- Do not reintroduce dead short-link dependencies.
- Do not commit credentials, cookies, or backup dumps.
- Do not document real internal link lists in markdown, issues, or commits.
- Do not run destructive git commands (`reset --hard`, force-push) without explicit user request.
- Do not remove text / icon / compact-icon views without user request.
- Do not “helpfully” redesign into purple gradients, heavy glassmorphism, or image-heavy dashboards unless asked.

---

## Quick checklist before push

- [ ] Sensitive URLs are `obf1:` only in committed files
- [ ] Company group order: Code → Wiki → other
- [ ] Display names follow conventions
- [ ] Cache-bust query updated on `index.html`, `text.html`, and `icons-lite.html`
- [ ] If UX changed on full homepage: `index.html` and `text.html` still aligned (except intentional icon/label differences)
- [ ] Default root still serves 有图 (`index.html`); `icons.html` remains redirect only
- [ ] Theme / `OBF_KEY` still consistent across entry pages
- [ ] No gitignored local backups staged
- [ ] Commit message has no internal hostnames
- [ ] User asked to commit/push/上线
