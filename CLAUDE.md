# Deck: handoff for planning (personal cross-device dashboard)

Written 2026-10-26 at the end of the Reserve (IT401) session. **Status: idea and requirements only. No code, no folder, no decisions locked.** The next session is a planning discussion; the build happens on the server PC.

Put this file in the new project folder on the PC (suggested: `C:\Latest Files\deck\HANDOFF.md`, or rename it `CLAUDE.md` there so new sessions load it automatically).

---

## 1. Who the user is (context for any Claude session)
- BSIT 4th-year student (BulSU, Malolos). Builds solo or as a team lead.
- **Recent projects:**
  - **Rewind** (`C:\Latest Files\School Files\rewind`): Android app (Kotlin, Jetpack Compose) + FastAPI server + vLLM (Qwen3-8B) + ChromaDB, run on the PC, reached through Tailscale Funnel; desktop control app "Rewind Hub" (`rewind-hub/`, FastAPI); APK "builds share" script (`rewind-hub/tools/build_share.py`). Defended; paused 2026-10-19 (see `rewind-docu/endgame/PROJECT_OVERVIEW.md` sections 8-9 for what's off and how to turn it on).
  - **Reserve** (`C:\Latest Files\School Files\BSIT 4EG2\IT401\Final Project\facility-reservation-system`, GitHub `baguiostrawberryjam/facility-reservation-system`): Django 6.1 web app, learned Django fundamentals there (models/ORM, migrations, forms, views, templates, Django Admin, auth, tests, services module).
- **Comfortable with:** Python, Django basics, Kotlin/Compose, Git/GitHub over SSH, PowerShell, Tailscale.
- **Devices:** Android phone, laptop (Jam's laptop; was the Rewind "witness"), desktop PC (Tailscale machine name `rewind`, has the GPU).
- **Current file habit:** Telegram (Saved Messages / Telegram Desktop) is the go-to way to move files between the three devices, because it works anywhere and downloads are easy.

## 2. Working rules the user set (apply here too unless they say otherwise)
- **YAGNI.** Don't overengineer. Never expand scope without permission. Suggest improvements, with a one-line justification, and wait for a yes. Every piece of work must be justifiable. State where the project sits between "quick and dirty" and "production grade" and engineer to that.
- **Learning matters.** Explain concepts briefly against what they already know; check understanding. (On Reserve they used a "reverse agent" workflow: Claude writes code to a `_scratch/` folder, the user reads and copies it in. **Ask whether they want that again or want Claude to build directly.**)
- Tag work as Required / Dependency / Added and keep a scope ledger and a commands log.
- Mockups first for UI (HTML prototypes in `_scratch/`), user comments, then build.
- Plain, human copy; no em dashes in UI text.

## 3. What the user wants (requirements so far)
| # | Need | Notes |
|---|---|---|
| N1 | **One dashboard reachable from phone, laptop and PC** | "Main dashboard": whenever I need anything from any of the three, I can do it |
| N2 | **Send to Deck from the phone** | Android share menu: files, links, text, photos, voice notes land on the PC |
| N3 | **Get files from anywhere**, Telegram-style | Must work away from home, not just on local Wi-Fi; easy download on any device |
| N4 | **Markdown viewer on the phone** | "A better Obsidian, but lightweight and scoped to my needs." Clean reading of `.md` files (handoffs, guides, notes). Editing scope: to decide |
| N5 | **Web version** as well as the phone | Laptop and PC use it in the browser |
| N6 | **Security: CIA triad** | Confidentiality: only the user, or people they give a token. Integrity: files not altered or corrupted. Availability: reachable when needed |
| N7 | **Clean, modern UI** | Same taste as Rewind/Reserve (below) |
| N8 | **PC as the server, on only when needed** | User turns it on before leaving and off when sleeping. **No always-on LLM / GPU use** (that's why Rewind can't stay on) |
| Ideas mentioned earlier | Clipboard bridge; Projects page that reads each repo's `CLAUDE.md`/handoffs; buttons that run scripts on the PC; study pipeline (lecture PDF → phone-sized study PDFs + checklist) | Not confirmed as scope yet |

## 4. Corrections and facts to carry into the discussion
- **Syncthing is not LAN-only.** It syncs over the internet through global discovery and relays, and also works over Tailscale. The user's real limitation is different: with Syncthing (or any PC-hosted server) **nothing is reachable while the PC is off**, unlike Telegram, which stores files in its cloud. This is the main design tension (see Q1).
- **Tailscale Serve vs Funnel.** All three devices can join the same tailnet, so **Tailscale Serve** (private to the tailnet, HTTPS with a real certificate) is enough and far safer than **Funnel**, which publishes the URL to the whole internet. Funnel is only needed for devices or people outside the tailnet (e.g. sharing a file with a classmate). On Rewind, Funnel had to rely on sign-in, rate limits and session revocation for that reason.
- Tailscale on the PC and laptop is currently stopped and set to Manual (Rewind pause). Turning it on: `PROJECT_OVERVIEW.md` §8.2 step 2 (PC) and §9.3 (laptop). Saved Funnel config: `tailscale-serve-config.json` in the Rewind backup folder on the PC.
- **A PWA can be an Android share target** (Web Share Target API, installed from Chrome). That may cover N2, N4 and N5 with **one web codebase** instead of a separate Kotlin app. Limits of a PWA: no background clipboard reading, limited background sync, needs to be "installed" once. Decide in planning (Q4).

## 5. Open questions for the planning session
1. **Availability while the PC is off (N3 vs N8).** Options: (a) accept it: Deck works only while the PC is on, and the phone keeps an offline cache of recent files and markdown; (b) add a small always-on piece (cheap VPS or object storage) for the Inbox only, encrypted before upload; (c) keep Telegram for "PC is off" moments and use Deck for everything else. Trade-offs: cost, complexity, privacy.
2. **Who besides the user gets access?** Only own devices (Tailscale Serve is enough), or occasionally others via a link (needs Funnel or share links with expiring tokens)?
3. **Storage:** which PC folders does Deck expose (a single Deck folder, or project folders too)? Size limits?
4. **Phone client:** PWA (one codebase, recommended to evaluate first) vs Kotlin app (better background behavior, more work, reuse of Rewind skills).
5. **Markdown (N4):** read-only first, or also editing? Which folders (repos' `CLAUDE.md`/`_scratch/guides`, a notes vault)? Mermaid diagrams? Search?
6. **Scripts runner and clipboard bridge:** in v1 or later?
7. **Encryption at rest:** Windows BitLocker on the PC drive vs app-level file encryption (affects search and previews).
8. **Framework:** Django (just learned; Admin, auth, ORM for free) vs FastAPI (used in Rewind Hub; lighter for an API + PWA).

## 6. Starting sketch (not decided; for discussion)
```
Phone (PWA or app) ─┐                     ┌─ Deck server on the PC (Django or FastAPI)
Laptop (browser)  ──┼── Tailscale (Serve, ─┤   • Inbox: uploads/links/text, SHA-256 checksums
PC (browser)      ──┘    HTTPS, tailnet)  │   • Files: browse/download chosen folders
                                          │   • Markdown: render .md (+ Mermaid) cleanly
                                          │   • Auth: per-device tokens (hashed, revocable, expiring)
                                          └─ Storage on the PC disk (BitLocker?)
```
- **Confidentiality:** tailnet-only access (Serve) + per-device tokens stored hashed; revoke a lost phone in one click; HTTPS by Tailscale; secrets in `.env`.
- **Integrity:** SHA-256 checksum stored per upload and verified on download; uploads written atomically (temp file → rename); append-only activity log.
- **Availability:** start Deck with one click/shortcut when leaving (like the Rewind Hub start at login); phone offline cache; nothing GPU-heavy.

## 7. Design language to reuse
Reserve's mockup and CSS are the reference: `...\facility-reservation-system\_scratch\mockups\portal.html` and `static\css\reserve.css`. Plus Jakarta Sans, glacier accent (`#90C5ED` shape / `#36698C` text on light), near-black filled buttons, rounded tiles (cards 16px, controls 10px), Cal.com-style compact layout with Linear restraint, light + dark mode with a toggle, Lucide icons, status always icon + word. Rewind's palette notes: `rewind-docu/ui-guide/COLOR_SYSTEM.md`. Taste skill: `rewind/.agents/skills/design-taste-frontend/SKILL.md` (built for landing pages; apply selectively).

## 8. First steps for the next session
1. Read this file; ask the open questions in section 5 (one or two at a time, recommendation first).
2. Agree on the grade (likely "personal tool, solid but not production": security done properly, no scaling work).
3. Write a scope ledger (Required / Dependency / Added) and a v1 cut.
4. Pick the folder on the PC, `git init`, and repeat the Reserve workflow (mockups → steps → commands log).
