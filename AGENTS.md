# AGENTS.md — Carlos' Nav (GitHub Pages)

Guidance for AI agents and humans maintaining this personal navigation site.

## Project

- **What:** Static personal URL navigation homepage.
- **Host:** GitHub Pages from this public repository (`carloscn.github.io`).
- **Stack:** Plain HTML + CSS + `data.js` (no build step, no framework).

## Key files

| Path | Role | Committed? |
|------|------|------------|
| `index.html` | Layout, styles, render logic, URL de-scramble (`resolveUrl`) | Yes |
| `data.js` | Public navigation data (`window.NAV_DATA`) | Yes |
| `assets/logos/*` | Icons | Yes |
| `assets/pic/headpic.jpg` | Avatar / favicon | Yes |
| `data.company.local.js` | Optional **local-only** plaintext company links | **No** (`.gitignore`) |
| `*.infinity`, `link*.json` | Local backups | **No** (`.gitignore`) |

Do **not** commit ignored local backup files.

## Hard rules (privacy / public repo)

1. **This is a public repository.** Anything committed is searchable on GitHub.
2. **Never write sensitive / internal hostnames or paths as plaintext** in committed files (`data.js`, `index.html`, this `AGENTS.md`, commit messages, PR text, comments).
3. **Never paste real internal/company URLs into agent prompts that will be logged into docs you plan to commit.** When adding links, prefer giving them only in the live chat task, or keep plaintext only in gitignored local files.
4. **Do not use short-link services** for internal entries (they expire and still leave traces). Use in-page scramble instead.
5. **Do not add company/internal URL inventories to this file.** This document describes *how*, not *what* those links are.
6. Public, non-sensitive URLs (e.g. personal GitHub repos, common SaaS) may stay as normal `https://...` in `data.js`.

## URL scramble (`obf1:`)

Internal or sensitive URLs in `data.js` must use the `obf1:` scheme so casual GitHub text search does not hit the raw host/path.

### Format

- Stored value: `obf1:` + Base64( XOR(plaintext_utf8, key) )
- Key (must match `index.html`): `cn-nav-k7`
- Runtime: `resolveUrl()` in `index.html` reverses this for `href`.

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
- Bump the cache-bust query on the script tag in `index.html`, e.g.  
  `data.js?v=20260807d` → `data.js?v=20260807e`  
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

- `logo` paths are relative to the site root.
- Prefer existing logos under `assets/logos/`; if missing, leave empty or use a fitting existing icon rather than inventing fake assets.
- Typical logos: `gitlab.webp` for code forges, `wiki.webp` for wiki docs, `github.png` for GitHub, etc.

## Company group ordering

There is a group named **公司**. When editing it, keep this order:

1. **Code-related** entries first (repos / forge roots / code projects).
2. **Wiki / docs** entries next.
3. **Everything else** last (chat, office, expense tools, personal public repos, …).

Do not invent a second public company section. Optional local plaintext mirror: gitignored `data.company.local.js` (`window.NAV_COMPANY`) — keep in sync when convenient; never commit it.

## Workflow for “add these links”

1. Read current `data.js` company (or target) group.
2. Classify each new item: code / wiki / other (or the matching public group).
3. Encode sensitive URLs with `obf1:`; leave clearly public URLs plain.
4. Insert into the correct position within the group order.
5. Choose an existing logo path.
6. Bump `data.js?v=...` in `index.html`.
7. Grep the working tree for accidental plaintext sensitive hosts before commit.
8. Commit and push **only when the user asks** to submit/upload.

### Suggested user prompt on a new machine

```text
Clone/open carloscn.github.io. Read AGENTS.md and follow it.
Add to the 公司 group (obf1: for internal URLs; order Code → Wiki → other):
- name: ...
  url: ...
  type: code | wiki | other
Then commit and push if I say 提交/上传.
```

(Users supply the actual URLs only in the chat task — not by editing this file.)

## Git / deploy

- Default branch: `master`.
- Site updates via push to GitHub Pages.
- Prefer clear English commit messages describing *behavior*, without listing internal hosts.
- If remote advanced, pull/rebase then push; do not force-push unless the user explicitly requests it.
- `CNAME` / custom domain may exist; do not remove it unless the user asks or deploy is blocked and they approve.

## Out of scope / avoid

- Do not reintroduce dead short-link dependencies.
- Do not commit credentials, cookies, or backup dumps.
- Do not “helpfully” document real internal link lists in markdown, issues, or commits.
- Do not run destructive git commands (`reset --hard`, force-push) without explicit user request.

## Quick checklist before push

- [ ] Sensitive URLs are `obf1:` only in committed files
- [ ] Company group order: Code → Wiki → other
- [ ] Cache-bust query updated
- [ ] No gitignored local backups staged
- [ ] Commit message has no internal hostnames
- [ ] User asked to commit/push
