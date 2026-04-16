# AGENTS.md -- Tab Out: Setup & Onboarding Guide for Coding Agents

You're installing **Tab Out** for the user. Your job is not just to set it up -- it's to get them excited about using it.

---

## Step 0 -- Introduce the product

Before doing anything technical, tell the user what they're about to get:

> **Tab Out** replaces your new tab page with a clean dashboard of everything you have open, grouped by domain.
>
> Here's what makes it great:
> - **See all your open tabs at a glance** grouped by domain on a grid
> - **Homepages group** pulls Gmail, X, LinkedIn, YouTube, GitHub homepages into one card for easy cleanup
> - **Close tabs with style** satisfying swoosh sound + confetti burst
> - **Duplicate detection** flags when you have the same page open twice
> - **Click any tab title to jump to it** even across different Chrome windows
> - **Save for later** bookmark individual tabs to a checklist before closing them
> - **100% local** no server, no accounts, no data sent anywhere
>
> It's just a Chrome extension. Setup takes about 1 minute.

---

## Step 1 -- Clone the repo

```bash
git clone https://github.com/RidingSnailWukong/tab-out.git
cd tab-out
```

---

## Step 2 -- Install the Chrome extension

This is the one step that requires manual action from the user. Make it as easy as possible.

**First**, print the full path to the `extension/` folder:
```bash
echo "Extension folder: $(cd extension && pwd)"
```

**Then**, copy the `extension/` folder path to their clipboard:
- macOS: `cd extension && pwd | pbcopy && echo "Path copied to clipboard"`
- Linux: `cd extension && pwd | xclip -selection clipboard 2>/dev/null || echo "Path: $(pwd)"`
- Windows: `cd extension && echo %CD% | clip`

**Then**, open the extensions page:
```bash
open "chrome://extensions"
```

**Then**, walk the user through it step by step:

> I've copied the extension folder path to your clipboard. Now:
>
> 1. You should see Chrome's extensions page. In the **top-right corner**, toggle on **Developer mode** (it's a switch).
> 2. Once Developer mode is on, you'll see a button called **"Load unpacked"** appear in the top-left. Click it.
> 3. A file picker will open. **Press Cmd+Shift+G** (Mac) or **Ctrl+L** (Windows/Linux) to open the "Go to folder" bar, then **paste** the path I copied (Cmd+V / Ctrl+V) and press Enter.
> 4. Click **"Select"** or **"Open"** and the extension will install.
>
> You should see "Tab Out" appear in your extensions list.

**Also**, open the file browser directly to the extension folder as a fallback:
- macOS: `open extension/`
- Linux: `xdg-open extension/`
- Windows: `explorer extension\\`

---

## Step 3 -- Show them around

Once the extension is loaded:

> You're all set! Open a **new tab** and you'll see Tab Out.
>
> Here's how it works:
> 1. **Your open tabs are grouped by domain** in a grid layout.
> 2. **Homepages** (Gmail inbox, X home, YouTube, etc.) are in their own group at the top.
> 3. **Click any tab title** to jump directly to that tab.
> 4. **Click the X** next to any tab to close just that one (with swoosh + confetti).
> 5. **Click "Close all N tabs"** on a group to close the whole thing.
> 6. **Duplicate tabs** are flagged with an amber "(2x)" badge. Click "Close duplicates" to keep one copy.
> 7. **Save a tab for later** by clicking the bookmark icon before closing it. Saved tabs appear in the sidebar.
>
> That's it! No server to run, no config files. Everything works right away.

---

## Key Facts

- Tab Out is a pure Chrome extension. No server, no Node.js, no npm.
- Saved tabs are stored in `chrome.storage.local` (persists across sessions).
- 100% local. No data is sent to any external service.
- To update: `cd tab-out && git pull`, then reload the extension in `chrome://extensions`.

---

## Development Commands

There is no build system, bundler, test runner, or package manager. All code is plain vanilla JS/CSS/HTML loaded directly by Chrome.

- **Reload after code changes**: Go to `chrome://extensions`, find "Tab Out", click the circular reload icon. Then open a new tab to see changes.
- **View console logs**: Right-click the new tab page → "Inspect" → Console tab. Dashboard logs are prefixed with `[tab-out]`.
- **Debug background service worker**: On `chrome://extensions`, click "Service worker" link under Tab Out to open DevTools for `background.js`.
- **Test with personal config**: Create `extension/config.local.js` (gitignored) to define `LOCAL_LANDING_PAGE_PATTERNS` and `LOCAL_CUSTOM_GROUPS` arrays for custom tab grouping rules.

---

## Code Architecture

### Overview

Tab Out is a Chrome Manifest V3 extension with zero build dependencies. The entire runtime lives in the `extension/` folder. There are exactly two JavaScript files and the split between them is critical to understand:

### File Roles

**`extension/manifest.json`** — Chrome extension manifest (V3). Declares permissions (`tabs`, `activeTab`, `storage`), overrides the new-tab page with `index.html`, and registers `background.js` as a service worker.

**`extension/background.js`** (~94 lines) — A lightweight service worker whose sole responsibility is keeping the toolbar badge up to date. It listens to `chrome.tabs.onCreated`, `onRemoved`, and `onUpdated` events, counts "real" tabs (excluding `chrome://`, `chrome-extension://`, `about:`, `edge://`, `brave://` URLs), and sets the badge text with color-coded background (green ≤10, amber 11–20, red 21+).

**`extension/app.js`** (~1480 lines) — The entire dashboard logic in a single file. This is the brain of the extension. It is loaded by `index.html` (which IS the new tab page) and has direct access to `chrome.tabs` and `chrome.storage` APIs — no message-passing bridge needed. The file is organized into these logical sections (top to bottom):

1. **Chrome Tabs API layer** (lines ~26–198): Functions that read/close/focus tabs. Key functions: `fetchOpenTabs()` populates the `openTabs` array; `closeTabsByUrls()` closes by hostname match; `closeTabsExact()` closes by exact URL (used for landing pages so closing "Gmail inbox" doesn't also close email threads); `focusTab()` switches to a tab across windows; `closeDuplicateTabs()` deduplicates; `closeTabOutDupes()` closes redundant Tab Out new-tab pages.

2. **Saved for Later persistence** (lines ~200–284): CRUD operations on `chrome.storage.local` under the `"deferred"` key. Each saved tab has `{id, url, title, savedAt, completed, dismissed}`. Functions: `saveTabForLater()`, `getSavedTabs()` (splits into active/archived), `checkOffSavedTab()`, `dismissSavedTab()`.

3. **UI helpers** (lines ~287–514): `playCloseSound()` synthesizes a swoosh via Web Audio API (bandpass-filtered noise sweep, no audio files); `shootConfetti()` creates 17 DOM particles with physics animation via `requestAnimationFrame`; `animateCardOut()` fades+scales a card then checks for empty state; `showToast()` for notifications; `timeAgo()` for relative dates; `getGreeting()`/`getDateDisplay()` for the header.

4. **Domain & title cleanup** (lines ~517–692): `FRIENDLY_DOMAINS` is a ~70-entry map of hostnames to display names. `friendlyDomain()` falls back to stripping TLDs and capitalizing. `stripTitleNoise()` removes notification counts like "(2)", email addresses, and X/Twitter formatting. `cleanTitle()` strips redundant domain suffixes from page titles. `smartTitle()` generates context-aware titles for GitHub (issues/PRs/files), X posts, YouTube videos, and Reddit threads.

5. **Card rendering** (lines ~695–897): `renderDomainCard()` builds the HTML for one domain group card including favicons (via Google's favicon service), duplicate badges, overflow chips ("+N more" for groups >8 tabs), and action buttons. `buildOverflowChips()` handles the expandable hidden tab list.

6. **Saved for Later column rendering** (lines ~900–1004): `renderDeferredColumn()` renders the right sidebar. `renderDeferredItem()` builds checklist items with checkbox, favicon, title, domain, time ago, and dismiss button. `renderArchiveItem()` for completed items.

7. **Main render pipeline** (lines ~1007–1173): `renderStaticDashboard()` is the entry point — it sets the greeting, fetches tabs, applies `LANDING_PAGE_PATTERNS` to separate homepages from content tabs, groups remaining tabs by hostname (with custom group support via `LOCAL_CUSTOM_GROUPS`), sorts groups (landing pages first, then landing-page domains, then by tab count), renders all domain cards into a CSS multi-column grid, updates footer stats, and renders the saved-for-later column.

8. **Event delegation** (lines ~1176–1476): A single `document.addEventListener('click')` handler routes all user interactions via `data-action` attributes on elements. Actions include: `close-tabout-dupes`, `expand-chips`, `focus-tab`, `close-single-tab`, `defer-single-tab`, `check-deferred`, `dismiss-deferred`, `close-domain-tabs`, `dedup-keep-one`, `close-all-open-tabs`. A separate listener handles archive toggle and search input filtering.

**`extension/index.html`** — The new tab page shell. Two-column layout: left column for open tabs grid, right column for "Saved for Later" checklist (hidden when empty). Loads Google Fonts (Newsreader for headings, DM Sans for body), `style.css`, optionally `config.local.js` (with `onerror` fallback), and `app.js`.

**`extension/style.css`** (~1159 lines) — All visual styling. Uses CSS custom properties (`:root` variables) for a warm paper-like color palette. Key layout: CSS `columns: 280px` for the responsive card grid (not CSS Grid or Flexbox grid); `position: sticky` for the saved-for-later sidebar; staggered `@keyframes fadeUp` animations for card entrance. The design has no dark mode.

**`extension/config.local.js`** (gitignored) — Optional personal configuration. Can export `LOCAL_LANDING_PAGE_PATTERNS` (array of `{hostname, pathExact?, pathPrefix?, test?, hostnameEndsWith?}` objects) and `LOCAL_CUSTOM_GROUPS` (array of `{hostname?, hostnameEndsWith?, pathPrefix?, groupKey, groupLabel}` objects) to customize which tabs are treated as landing pages or merged into custom groups.

### Data Flow

1. User opens new tab → Chrome loads `index.html` → `app.js` runs `renderDashboard()` at the bottom of the file
2. `fetchOpenTabs()` calls `chrome.tabs.query({})` → populates global `openTabs` array
3. `getRealTabs()` filters out browser-internal URLs → tabs fed into grouping logic
4. Grouping: each tab checked against `LANDING_PAGE_PATTERNS` (landing → special group), then `LOCAL_CUSTOM_GROUPS` (custom → named group), then grouped by `hostname`
5. `renderDomainCard()` called per group → HTML injected into `#openTabsMissions` container
6. User clicks any element → event bubbles to document-level listener → matched by `data-action` attribute → corresponding async handler called → Chrome tabs API mutated → UI updated

### Key Design Decisions

- **No framework**: Everything is vanilla JS with direct DOM manipulation and string template HTML. No React, no bundler, no transpilation.
- **Event delegation**: One click listener on `document` instead of per-element listeners. Actions are identified by `data-action` attributes.
- **Landing page separation**: Landing pages (Gmail inbox, GitHub home, etc.) are grouped separately so "close all Gmail tabs" doesn't also close individual email threads — `closeTabsExact()` vs `closeTabsByUrls()`.
- **Favicon sourcing**: Uses `https://www.google.com/s2/favicons?domain=...&sz=16` with `onerror="this.style.display='none'"` fallback.
- **Sound synthesis**: The close swoosh is generated entirely via Web Audio API (noise buffer + sweeping bandpass filter), requiring zero audio file assets.
