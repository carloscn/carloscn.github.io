# AGENTS.md — Carlos' Nav (GitHub Pages)

Guidance for AI agents and humans maintaining this personal navigation site.

## Project

- **What:** Static personal URL navigation homepage.
- **Host:** GitHub Pages from this public repository (`carloscn.github.io`).
- **Stack:** Plain HTML + CSS + `data.js` (no build step, no framework).
- **Default home:** text-first hao268-style layout (`index.html`). Icon grid is a secondary view.

## Two views (shared data)

| File | Role |
|------|------|
| `index.html` | **Main homepage** — pure-text layout (search tabs, weather, world clocks, side tools, category rows). Does **not** render item logos. |
| `icons.html` | **Icon view** — classic tile grid with logos/avatar. Toggle from text home via “有图版本”. |
| `data.js` | Single shared `window.NAV_DATA` for **both** views. |

Cross-links:

- Text home → icon view: `icons.html`
- Icon view → text home: `index.html`

When editing layout/UX, keep the two pages in sync for: `resolveUrl` + `OBF_KEY`, and the `data.js?v=...` cache-bust query (bump **both** HTML files).

## Key files

| Path | Role | Committed? |
|------|------|------------|
| `index.html` | Default homepage: text layout, weather, multi-tab search, clocks, `resolveUrl` | Yes |
| `icons.html` | Icon/tile layout, `resolveUrl` | Yes |
| `data.js` | Public navigation data (`window.NAV_DATA`) | Yes |
| `assets/logos/*` | Icons (used by `icons.html`) | Yes |
| `assets/pic/headpic.jpg` | Avatar / favicon | Yes |
| `data.company.local.js` | Optional **local-only** plaintext company links | **No** (`.gitignore`) |
| `*.infinity`, `link*.json` | Local backups | **No** (`.gitignore`) |

Do **not** commit ignored local backup files.

## Hard rules (privacy / public repo)

1. **This is a public repository.** Anything committed is searchable on GitHub.
2. **Never write sensitive / internal hostnames or paths as plaintext** in committed files (`data.js`, `index.html`, `icons.html`, this `AGENTS.md`, commit messages, PR text, comments).
3. **Never paste real internal/company URLs into agent prompts that will be logged into docs you plan to commit.** When adding links, prefer giving them only in the live chat task, or keep plaintext only in gitignored local files.
4. **Do not use short-link services** for internal entries (they expire and still leave traces). Use in-page scramble instead.
5. **Do not add company/internal URL inventories to this file.** This document describes *how*, not *what* those links are.
6. Public, non-sensitive URLs (e.g. personal GitHub repos, common SaaS) may stay as normal `https://...` in `data.js`.

## URL scramble (`obf1:`)

Internal or sensitive URLs in `data.js` must use the `obf1:` scheme so casual GitHub text search does not hit the raw host/path.

### Format

- Stored value: `obf1:` + Base64( XOR(plaintext_utf8, key) )
- Key (must match **both** `index.html` and `icons.html`): `cn-nav-k7`
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
- Bump the cache-bust query on the script tag in **both** HTML entry points, e.g.  
  `data.js?v=20260807f` → `data.js?v=20260807g`  
  in `index.html` **and** `icons.html`  
  (also update the comment at the top of `data.js` if present).

This scramble is **not strong crypto** (key is in the page). It only reduces accidental discovery via repository search.

## Data shape

```js
window.NAV_DATA = {
  title: "Carlos' Nav",
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

- `logo` paths are relative to the site root; used by `icons.html`. Text home ignores logos but still keeps the field for the icon view.
- Prefer existing logos under `assets/logos/`; if missing, leave empty (`""`) or use a fitting existing icon rather than inventing fake assets.
- Typical logos: `gitlab.webp` for code forges, `wiki.webp` for wiki docs, `github.png` for GitHub, etc.

### Display name conventions

Keep `name` tidy and consistent:

| Kind | Rule | Examples |
|------|------|----------|
| Global brands | Official spelling / casing | Reddit, Instagram, ChatGPT, LinkedIn, YouTube |
| Chinese sites | Common short Chinese names | 淘宝、携程、豆瓣、QQ 邮箱 |
| Multi-word English | Spaces + Title Case (or official form) | Flight Connections, Submarine Cable Map |
| Internal labels | Readable Title Case; no `snake_case` / raw host dumps | Thor Wiki, XCU Platform |
| Avoid | Domain-like dumps, random ALL CAPS, mixed `ChatGpt` | use ChatGPT not ChatGpt |

## Company group ordering

There is a group named **公司**. When editing it, keep this order:

1. **Code-related** entries first (repos / forge roots / code projects).
2. **Wiki / docs** entries next.
3. **Everything else** last (chat, office, expense tools, personal public repos, …).

Do not invent a second public company section. Optional local plaintext mirror: gitignored `data.company.local.js` (`window.NAV_COMPANY`) — keep in sync when convenient; never commit it.

## Public group guidance (non-company)

When classifying new public links, prefer existing groups:

| Group | Typical contents |
|-------|------------------|
| 首页 | Daily high-frequency tiles (first group; denser grid on text home) |
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

## Workflow for “add these links”

1. Read current `data.js` target group(s).
2. Classify each new item into an existing group (company: code / wiki / other).
3. Encode sensitive URLs with `obf1:`; leave clearly public URLs plain.
4. Apply display-name conventions; insert into the correct position within group order.
5. Choose an existing logo path (or `""` if none).
6. Bump `data.js?v=...` in **`index.html` and `icons.html`**.
7. Grep the working tree for accidental plaintext sensitive hosts before commit.
8. Commit and push **only when the user asks** to submit/upload/上线.

### Suggested user prompt on a new machine

```text
Clone/open carloscn.github.io. Read AGENTS.md and follow it.
Add to the 公司 group (obf1: for internal URLs; order Code → Wiki → other):
- name: ...
  url: ...
  type: code | wiki | other
Then commit and push if I say 提交/上传/上线.
```

(Users supply the actual URLs only in the chat task — not by editing this file.)

## Git / deploy

- Default branch: `master` (GitHub Pages deploys from here).
- Site updates via push to `master`.
- Prefer clear English commit messages describing *behavior*, without listing internal hosts.
- If remote advanced, pull/rebase then push; do not force-push unless the user explicitly requests it.
- `CNAME` / custom domain may exist; do not remove it unless the user asks or deploy is blocked and they approve.
- Feature branches may be used for larger UI work; merge to `master` when the user asks to go live.

## Out of scope / avoid

- Do not reintroduce dead short-link dependencies.
- Do not commit credentials, cookies, or backup dumps.
- Do not “helpfully” document real internal link lists in markdown, issues, or commits.
- Do not run destructive git commands (`reset --hard`, force-push) without explicit user request.
- Do not remove the text/icon dual-view without user request.

## Quick checklist before push

- [ ] Sensitive URLs are `obf1:` only in committed files
- [ ] Company group order: Code → Wiki → other
- [ ] Display names follow conventions
- [ ] Cache-bust query updated on **both** `index.html` and `icons.html`
- [ ] No gitignored local backups staged
- [ ] Commit message has no internal hostnames
- [ ] User asked to commit/push/上线
