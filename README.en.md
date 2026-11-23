# KPVault

**KPVault is a Windows desktop app for managing passwords.** Every account lives in an encrypted `.kdbx` file (the KeePassXC format) that opens with a single master password; KPVault is the "how do I actually use it every day" layer — search entries, copy with one keystroke, back up automatically, sync across machines, and fill login forms straight from the browser.

All cryptography, database access and merging is delegated to the `keepassxc-cli` already installed on your machine. The app never parses `.kdbx` and implements no crypto of its own, so what you hold is always a **standard KeePass database** — openable with the official KeePassXC client or any other KeePass tool.

> 中文说明见 [README.md](README.md)。

### What it solves

| Everyday annoyance | What KPVault does |
| :--- | :--- |
| Hundreds of accounts, can't find the one you need | Type-to-filter (title / username / URL / group); `Ctrl+C` sends the current entry's password to the clipboard |
| Worried about a corrupted file or a bad delete | Automatic backup before every write (keeps 20), a random backup is test-decrypted every 7 days, failure shows a persistent banner |
| Work and home machines drift apart | Built-in WebDAV sync (Nutstore): upload after each write; conflicting edits are never silently overwritten — they become a conflict copy for you to merge |
| Retyping passwords on login pages | Bundled browser extension: one click to fill; can also fill only the username (no decryption) or only the password |
| Migrating from another password manager | CSV import / export, group hierarchy preserved |

**Who it's for**: people already using (or willing to use) the KeePass format who want a smoother desktop UI.

**Who it's not for**: anyone unwilling to install KeePassXC locally (the installer deliberately does not bundle it); macOS / Linux / mobile users (Windows x64 only right now); anyone expecting a cloud service, multi-user collaboration or vendor support — this is a single-user tool written for its author.

> ⚠️ **Two hard prerequisites**
>
> 1. **KeePassXC is not bundled.** The app is only a UI: install [KeePassXC](https://keepassxc.org/download/) (≥ 2.7.0) first, or unzip the portable build shipped on the download page, and let the app locate `keepassxc-cli.exe`.
> 2. **The master password is stored nowhere and cannot be recovered.** Forget it and the database is gone for good; backups are encrypted with the same password and are equally unreadable.

## Contents

- [Quick start](#quick-start)
- [1. What it is](#1-what-it-is)
- [2. Feature overview](#2-feature-overview)
- [3. Requirements and downloads](#3-requirements-and-downloads)
- [4. Install and first run](#4-install-and-first-run)
- [5. UI tour](#5-ui-tour)
- [6. Daily use: entries, groups, search](#6-daily-use-entries-groups-search)
- [7. Backup and restore](#7-backup-and-restore)
- [8. WebDAV sync](#8-webdav-sync)
- [9. Browser autofill](#9-browser-autofill)
- [10. Session, tray and shortcuts](#10-session-tray-and-shortcuts)
- [11. Update checks](#11-update-checks)
- [12. Data and config locations](#12-data-and-config-locations)
- [13. FAQ](#13-faq)
- [14. License and third-party components](#14-license-and-third-party-components)

---

## Quick start

From zero to "filling passwords in the browser" takes about 5 minutes. You need two files, both on [GitHub Releases](https://github.com/billgoto/kpvault-releases/releases/latest): `KPVault-Setup.exe` (the app) and `KeePassXC-2.8.0-x64.zip` (required runtime component).

### Step 1: prepare KeePassXC

Unzip `KeePassXC-2.8.0-x64.zip` into a fixed directory (e.g. `D:\Portable\KeePassXC-2.8.0-x64`), or install [KeePassXC](https://keepassxc.org/download/) normally (**≥ 2.7.0**, 2.8.0 recommended).

> The app does not bundle KeePassXC — it is only the UI; the actual crypto is done by the local `keepassxc-cli`.

### Step 2: install the app and get past the first screens

1. Run `KPVault-Setup.exe` (install location is selectable; a desktop shortcut is created).
2. On first launch it probes for `keepassxc-cli`. If not found, click "Select KeePassXC folder" and pick the folder from step 1 (you can also point directly at `keepassxc-cli.exe`; the top bar also has "Specify KeePassXC folder").
3. Create or restore a database:
   - New user → "Create database" → set a master password (typed twice);
   - Existing database / new machine → "Restore from WebDAV…" to download the remote copy;
   - **Never create a fresh database on both machines** — you will end up with two unrelated databases.
4. Unlock and you are in.

> 🔑 The master password is stored nowhere and cannot be recovered. Losing it means losing the database; backups are just as unreadable.

### Step 3: add an account to confirm the read/write path works

Click "New" on the left (`Ctrl+N`) → fill title, username, password, URL → "Save" → click the entry → "Copy".

If copying works, the whole chain from UI to `keepassxc-cli` is fine; what remains is browser configuration.

### Step 4: configure the browser extension

The extension is an unpacked MV3 extension shipped with the app and must be loaded **once** by hand. It does not depend on the official KeePassXC-Browser extension.

**4.1 Get the pairing token in the app**

1. Top bar → "Settings" → "Browser fill".
2. Make sure "Enable local bridge" is checked (on by default) and the status reads "Running".
3. Click "Copy" next to "Pairing token" (64 characters).
4. Click "Open extension folder" (installed build: `<install dir>\resources\browser-extension\`).

**4.2 Load the extension in the browser**

1. Open `chrome://extensions` in Chrome (`edge://extensions` in Edge).
2. Enable "Developer mode".
3. Click "Load unpacked" and select the folder opened in 4.1 step 4.
4. Click the extension icon ("KPVault 填充") → paste the token → "Save and connect". Keep the port at `47823`, matching "Local port" in the app.

**4.3 Verify**

Open any login page → click the extension icon → the "Matched" tab should list entries for that site → click one to fill. If the database is locked, an unlock window appears first and the interrupted fill **resumes automatically** afterwards.

### If you get stuck

| Symptom | What to do |
| :--- | :--- |
| Settings shows "Not running" and mentions the port is in use | `47823` is taken — most often another KPVault still sitting in the tray. Quit it and re-enable the bridge. The port is fixed, never shifted |
| Extension says "cannot reach local app" | Clicking an account now launches the app for you (or use "Launch KPVault" in the popup). If it is already running, check 4.1: bridge enabled, port matching |
| Extension reports a token mismatch | The token is 64 characters — usually an incomplete copy. Use "Re-pair" in the extension and paste again |
| Want a confirmation before every fill | Turn on "Confirm in app before filling" in settings (off by default; the token is then the only barrier, see [§9](#9-browser-autofill)) |

Next: [§7](#7-backup-and-restore) backup and restore, [§8](#8-webdav-sync) WebDAV sync, [§6.4](#64-csv-import--export) migrating from another password manager.

---

## 1. What it is

In one line: **a UI layer for KeePassXC**. It wraps `keepassxc-cli` in a desktop app, never touching encryption or the database format.

Differences from using the official KeePassXC client directly:

|  | KeePassXC client | KPVault |
| :--- | :--- | :--- |
| Role | Full KeePass client (GUI + CLI + browser integration) | UI layer only: crypto, database access and merging are all forwarded to `keepassxc-cli` |
| Interaction | Multiple windows + menu bar + toolbar | Single window with a custom title bar: group tree left, details right, search-as-you-type, `Ctrl+C` copies the current password |
| Automatic backup | Optional "backup before save" | On by default: 20 backups kept, one randomly test-decrypted every 7 days, persistent banner on failure |
| Multiple devices | Sync the file through a cloud drive (external tool) | Built-in WebDAV: upload after write, periodic comparison; conflicting edits become a conflict copy with entry-level merge |
| Browser fill | Requires KeePassXC-Browser, official protocol | Bundled MV3 extension + local bridge (token pairing); the context menu can target exactly one field |

Layers:

| Layer | Implementation |
| :--- | :--- |
| Crypto engine | Local `keepassxc-cli` (create / unlock / CRUD / export / merge). The app never parses `.kdbx` and implements no cryptography |
| Desktop UI | Electron main process + Vue 3 / Pinia renderer; custom title bar, system tray |
| Data sync | Built-in WebDAV sync (upload after write, periodic remote comparison, conflict copies) plus detection of external file changes from any sync tool |
| Browser fill | Custom MV3 extension + in-app local bridge (`127.0.0.1` + pairing token), independent of KeePassXC-Browser |
| Distribution | `update.json` on GitHub Releases for version checks; the installer comes from the same Release. **Notify only — never downloads or installs by itself** |

**Deliberately not done**: bundling or distributing KeePassXC binaries, storing the master password, holding the password in the renderer process, auto-installing updates, silent automatic conflict merging.

Security baseline: the renderer reaches the main process only through a `contextBridge` allow-list (`sandbox: true`, `contextIsolation: true`, `nodeIntegration: false`); the master password lives in main-process memory as a `Buffer` and is zeroed on lock; the tray menu never shows entry content.

---

## 2. Feature overview

| Feature | Notes |
| :--- | :--- |
| Login-style unlock | Dedicated unlock page, autofocus, Enter to submit, password reveal, strength hint; decryption happens locally only |
| Entry management | Create / edit / view / delete / move; fields are group, title, username, password, URL, notes |
| Random password generator | "Generate" next to the password field: 20 chars by default, length 8–64, symbols optional, excludes look-alikes `I l 1 O 0`; at least one character from each class |
| Change master password | Settings → Master password: current + new (≥ 8 chars, twice) → whole database re-encrypted. A backup is taken first (still openable with the old password); the current session switches to the new password instead of bouncing back to the unlock page |
| Groups | Create subgroup, rename, delete a whole group (including its entries, irreversible); groups collapse individually (count shown next to the name); action buttons appear on hover only |
| Search | Live filter on title / username / URL / group, with a clear button; "Search notes" runs a full-text search through the CLI |
| CSV import / export | CSV with a `group` column interchanges both ways: export the whole database in plain text (re-importable, hierarchy restored); import writes entry by entry, duplicates get a suffix, progress is visible |
| View and copy | Decrypt on demand, one-click copy; the clipboard is cleared after **25 seconds**, and only if it still holds what we put there |
| Automatic backup | Before every write, keep the latest 20; manual backups are never pruned; one random backup test-decrypted every 7 days |
| WebDAV sync | Works with Nutstore and other WebDAV services: upload after write, periodic comparison, conflict copies; a new machine can restore straight from the remote copy |
| External changes and merge | Detects changes written by a sync tool and offers "Reload"; conflict copies merge at entry level, with a backup first and the copy renamed (not deleted) afterwards |
| Browser autofill | Popup lists entries under Matched / All / Favorites / Recent, with search, fill, copy username or password, save the current login, and in-page unlock; a context-menu entry fills without relying on input detection |
| Session protection | Idle auto-lock (1/5/15/30 min), lock on sleep / screen lock / user switch, lock on system clock jump, manual lock |
| Tray and shortcuts | Closing the window minimizes to tray, tray menu to quit, optional global hotkey, in-app shortcuts |
| Update check | On by default: 30 s after start and every 24 h; the version label highlights when an update exists and links to the download page |

---

## 3. Requirements and downloads

| Item | Requirement |
| :--- | :--- |
| OS | Windows x64 (only the NSIS installer is published; the CLI locator also contains Linux paths in source) |
| KeePassXC | **[≥ 2.7.0](https://keepassxc.org/download/)**, 2.8.0 recommended; installed or portable, as long as `keepassxc-cli.exe` can be found |
| Browser (optional) | Chromium-based (Chrome / Edge) to load the unpacked fill extension |
| WebDAV (optional) | Any standard WebDAV service: Nutstore, pCloud, Synology / fnOS / QNAP NAS, Nextcloud, self-hosted or an OpenList gateway. Most need an **app password**, not the login password |

**Downloads**: <https://github.com/billgoto/kpvault-releases/releases/latest> (public repo, anonymous access; speeds from mainland China can be flaky — a proxy helps)

| File | Purpose |
| :--- | :--- |
| `KPVault-Setup.exe` | The installer; always overwritten under the same name, so the link never changes |
| `KeePassXC-2.8.0-x64.zip` | Required portable KeePassXC; unzip anywhere permanent |

Source: <https://gitee.com/goobill/kpvault> · Releases (manifest + installer): <https://github.com/billgoto/kpvault-releases/releases/latest>

---

## 4. Install and first run

1. **Prepare KeePassXC**: install it from [keepassxc.org](https://keepassxc.org/download/), or unzip `KeePassXC-2.8.0-x64.zip` into a fixed folder (e.g. `D:\Portable\KeePassXC-2.8.0-x64`).
2. **Install the app**: run `KPVault-Setup.exe` (NSIS; selectable install path, desktop shortcut created).
3. **First launch probes for `keepassxc-cli`** in this order: path remembered in config → common install locations (`Program Files`, `LOCALAPPDATA\Programs`, chocolatey, scoop) → `PATH`. If nothing is found you get the first onboarding screen:
   - "Select KeePassXC folder": pick the unzipped portable folder; the app looks one level down for `keepassxc-cli.exe`;
   - "Select exe file": point directly at `keepassxc-cli.exe`;
   - "Detect again" / "Download": after installing or upgrading the component.
   - The top bar also keeps "Detect KeePassXC again" and "Specify KeePassXC folder" permanently; a manual choice is remembered.
4. **Create or restore the database**: the location is fixed at `%APPDATA%\KPVault\data\main.kdbx` (since v1.44 it no longer lives in the install directory) and cannot be changed (to avoid clobbering cloud-sync copies). The second onboarding screen offers:
   - "Create database" → set a master password (with strength hint, typed twice) → "Create and unlock";
   - "Restore from WebDAV…" → **use this on a new machine**: enter the server address (the provider dropdown can prefill it), username and app password, and download the existing database; for a home NAS tick "Allow self-signed certificate". Do not create a fresh database on the second machine.
5. **Unlock**: every later launch opens the login-style unlock page; type the master password (eye icon toggles plain text), press Enter or click "Unlock".

> 🔑 **The master password is never stored and cannot be recovered.** Losing it means losing the database permanently — keep it safe.

---

## 5. UI tour

The window uses a **fully custom title bar**: no system menu bar, no maximize button; the whole top bar is draggable and only "Minimize" / "Close" sit on the right.

| Area | Contents |
| :--- | :--- |
| Top bar | App name, KeePassXC status, detect / specify folder, backup, settings, version (update check), lock, window controls |
| Left sidebar | Search box (clear button appears while filtering), new entry, search notes, new group, CSV import / export, entry list grouped by group (collapsible) |
| Content area | Entry details / edit form, plus mutually exclusive overlays: settings, update, conflict copies, backup, import, export |
| Notice bar | Shown above the main view when the database changed externally or a conflict copy exists: "Reload" / "View copy" |

Clicking anywhere in the sidebar (select entry, switch group, search, new) collapses the current overlay and returns to the list.

---

## 6. Daily use: entries, groups, search

### 6.1 Entries

- **New**: "New" in the sidebar or `Ctrl+N`, fill group, title, username, password, URL, notes → "Save". Missing groups are created automatically; a backup is taken before the write.
- **View**: click an entry. The password is masked by default; "Show" decrypts on demand, "Copy" puts it on the clipboard (cleared after 25 s).
- **Edit**: click "Edit". An empty password field means **do not change**; only a non-empty value updates it. The group cannot be changed here — use "Move to…".
- **Generate password**: "Generate" on the password label row fills a random password and reveals it; a parameter row appears (length 8–64, symbols on/off), click "Generate again" to apply changes. Defaults: 20 chars, at least one from each class, `I l 1 O 0` excluded.
- **Move**: "Move to…" with the target group path.
- **Delete**: "Delete" then "Confirm delete". The entry is really removed (the app also purges the copy the database would keep in its recycle bin). **There is no undo** — restore a backup if needed.

### 6.2 Groups

**Collapse / expand**: click the group header row; a right-pointing triangle means collapsed. The number next to the name is the entry count, visible even when collapsed. With 3 or more groups the toolbar offers "Collapse all / Expand all". While searching, groups are **forced open** so hits cannot hide inside a collapsed group; the previous state returns when the filter is cleared.

Hovering a group name reveals three buttons (they do not occupy layout or widen the row):

| Button | Action |
| :--- | :--- |
| `＋` | Create a subgroup under this group |
| `✎` | Rename the group (entries are moved one by one, with progress) |
| `×` | Delete the group and all its entries, double confirmation, **irreversible** |

The root group is shown as "(root group)" and cannot be renamed or deleted.

### 6.3 Search

- The top search box filters **title / username / URL / group** live; `Ctrl+F` focuses it.
- "Search notes" runs a full-text search through the CLI; long result sets are truncated, so narrow the keyword.
- While filtering, an `×` appears on the right of the box; click it or press `Esc` to restore the full list.

### 6.4 CSV import / export

For interchanging with other password managers or bulk-importing an account list. Two sidebar buttons: "Import" and "Export".

**Import**

- The first line must be a header. Recognized: `name` (or `title`) / `url` / `username` / `password` / `group` (optional); **column order does not matter**, and Chinese headers are accepted too (标题 / 网址 / 用户名 / 密码 / 分组).
- Delimiter is auto-detected (`,` / `;` / Tab); encodings: UTF-8, Excel CSV with BOM, and GBK.
- A preview shows title / username / URL / whether a password exists (**passwords never appear in the UI**); choose a target group and write:
  - no `group` column → everything goes into the target group (default "导入");
  - with `group` column → written as "target group / file group"; leave the target empty to reproduce the file hierarchy exactly.
- Entries are written one by one (about 1 s each, x/N progress) and only one backup is taken. **Existing names are never overwritten** — a ` (2)` suffix is added.
- Passwords stay in main-process memory only and are cleared when the import ends or the session locks.

**Export**

- One decryption exports the whole database with fields `name,url,username,password,group,notes`, UTF-8 with BOM (so Excel shows Chinese correctly). **The exported file can be imported back.**
- Entries in the recycle bin are skipped; TOTP, attachments, custom fields and timestamps are not included.
- ⚠️ **The export is plain text**: anyone with the CSV sees every password. It is not a backup — keep it on an encrypted disk and delete it promptly.

---

## 7. Backup and restore

"Backup" in the top bar opens the panel:

| Action | Notes |
| :--- | :--- |
| Back up now | Manual copy, never auto-pruned |
| Verify one | Test-decrypt a backup right now |
| Open folder | Opens `%APPDATA%\KPVault\backups\` |
| Restore | Confirm twice; the backup is **verified as decryptable before touching anything** and the current database is saved aside, so a restore is itself reversible |

Rules: an automatic backup is taken before every write and the latest **20** are kept; every **7 days** one random backup is test-decrypted, and a failure raises a persistent banner. Backups are encrypted copies of the same database — the master password is still required to open them.

---

## 8. WebDAV sync

Enable it under "Settings → WebDAV sync" and fill in four things: **server address, username, password, remote folder**.

| Field | Notes |
| :--- | :--- |
| Provider | Just a **shortcut**: Nutstore / pCloud / custom. The address field below is always the source of truth |
| Server address | See 8.1. Self-hosted and NAS addresses cannot be preset — type them |
| Username | Nutstore: account email; NAS: device account; pCloud: account email |
| Password | Depends on the service: Nutstore / pCloud need an **app password** (Nutstore: web → account info → security → third-party apps → add); a NAS uses its account password |
| Remote folder | Default `/kpvault`, database file is always `main.kdbx`. **It must live inside a shared folder** (see 8.3) |
| Allow self-signed certificate | Home NAS HTTPS is usually self-signed; turn this on if it will not connect. It skips certificate validation — only for your own LAN server |
| Upload after each write | On by default |
| Remote check interval | 30 s / 1 min / 5 min / 15 min, default 1 min |

**Every setting is now "edit, then Save"**: the Save button at the top right enables as soon as something changed, and editing a field no longer writes the config immediately (session, language, shortcuts, updates, browser autofill and WebDAV alike). "Test connection" tries the values you typed without saving them, so you can verify before saving; "Sync now" triggers one manual run. The password is stored encrypted via the OS `safeStorage`; if unavailable, the UI says plainly that it will be stored in plain text in the config file.

### 8.1 What it can talk to

Anything that speaks **standard WebDAV** works. Known-good settings you can copy:

| Platform | Server address | Account | Watch out for |
| :--- | :--- | :--- | :--- |
| Nutstore | `https://dav.jianguoyun.com/dav` | account email | **App password** is mandatory; disable WeChat 2FA; free tier allows 600 requests / 30 min (this app compares once a minute by default) |
| pCloud | `https://webdav.pcloud.com` (EU: `https://ewebdav.pcloud.com`) | account email + password | With 2FA enabled, approve the login from your inbox once |
| Synology | `https://LAN-IP:5006` | DSM account | Install the **WebDAV Server** package; 5006 is HTTPS, 5005 is HTTP; self-signed cert → turn on "Allow self-signed certificate" |
| fnOS (飞牛) | `https://LAN-IP:5006`, or the address FN Connect gives you (`https://dav.xxx.fnos.net`) | fnOS account | Enable it under System settings → File sharing protocols → WebDAV; **team folders need "allow mounting over sharing protocols" ticked, it is off by default** |
| QNAP / TerraMaster | `https://LAN-IP:5006` (port depends on the package) | NAS account | Enable the WebDAV service first |
| Nextcloud / ownCloud | `https://domain/remote.php/dav/files/your-username` (ownCloud: `/remote.php/webdav`) | account + **app password** | The address includes your username; create an app password under Settings → Security |
| OpenList / AList / rclone gateway | `http://LAN-IP:5244/dav` | gateway account | One integration covers ~30 drives (Aliyun Drive, Baidu, Quark, 115, 123, OneDrive, Google Drive…); you run the gateway yourself |
| nginx / Apache mod_dav | `https://your-domain/dav` | Basic account | Most freedom; you own the certificate, port and path prefix |

> OneDrive, Google Drive, Dropbox, Aliyun Drive and Baidu Netdisk have **no WebDAV**; talking to them directly needs OAuth, which this app does not do. Use the OpenList gateway row above instead.

### 8.2 Three steps for a home NAS (Synology / fnOS / QNAP)

1. **Enable the service**: install WebDAV Server on Synology (default 5006); on fnOS open System settings → File sharing protocols → WebDAV. Note the port.
2. **Prepare a folder**: create something like `kpvault` on the NAS and make sure your account can **write** to it.
3. **Fill in the app**: address `https://LAN-IP:5006`, NAS account and password, tick "Allow self-signed certificate", set the remote folder to `/shared-folder/kpvault`, then "Test connection" and "Sync now".

### 8.3 Common failures

| Symptom | Cause and fix |
| :--- | :--- |
| "Certificate is not trusted" | The NAS uses a self-signed certificate → turn on "Allow self-signed certificate". Never for a public service |
| `HTTP 409` failure (sometimes with "the remote folder could not be created") | The remote folder points at the NAS **root**. A NAS root is a list of shared folders and cannot take a new directory → use `/shared-folder/kpvault` |
| Team folders will not sync | Team folders on fnOS / Synology disallow WebDAV mounting by default; tick it in the folder properties |
| `HTTP 403` | Wrong credential type: Nutstore / pCloud need an **app password** rather than the login password; on Nutstore also disable WeChat 2FA; check the account can write to that folder |
| `HTTP 401` | Wrong username or password; on Nextcloud the usual cause is a missing username in the address |
| "The remote changed" on every sync | That server's ETag is unstable (common when a gateway fronts a third-party drive) → click "Sync now" once to rebuild the baseline; if it keeps happening, switch services |

### Merging a conflict copy

Click "View copy" in the notice bar → pick one → click "Merge" twice. Merging calls `keepassxc-cli merge` and is **entry-level last-write-wins**: when the same entry changed on both sides the newer version wins and the other change is silently lost; a deletion on one side may be resurrected by the other. A backup is taken first and the copy is renamed `.merged-<time>` afterwards (**not deleted**); every change is listed.

### External changes

If the file is changed by a sync tool or another program, the notice bar says the shown content is stale; click "Reload". If there were local writes in the meantime, you are told that an external overwrite was detected and can restore from backup and merge.

> Enabling WeChat two-factor authentication on Nutstore makes WebDAV return **403**; disable it in the Nutstore web UI first.

---

## 9. Browser autofill

Optional feature, made of a **custom extension plus an in-app local bridge**: the bridge listens on `127.0.0.1:47823` only and validates the extension Origin and the pairing token; the extension is an unpacked MV3 extension and must be loaded manually. **The port is fixed at `47823`; if it is taken the app reports an error instead of shifting** (shifting used to leave the extension pointing at a dead port).

**Setup**

1. App "Settings → Browser fill": enable the local bridge (on by default) and click "Copy" for the pairing token.
2. Click "Open extension folder" (packaged: `<install dir>\resources\browser-extension\`).
3. Chrome / Edge → `chrome://extensions` → enable "Developer mode" → "Load unpacked" → select that folder.
4. Click the extension icon → paste the 64-character token → "Save and connect"; keep the port at `47823`. The settings page has a "Copy" next to "Local port"; if the ports differ, the extension probes and corrects itself. If the token no longer works, use "Re-pair" or regenerate it in settings.

**Popup**: search box; four tabs `Matched` (current site) / `All` / `Favorites` (★) / `Recent`; per-row "Username" / "Password" copy buttons (the password is written to the clipboard **by the main process and cleared after 25 s**; the plaintext never goes back to the browser); "Refresh list"; "Save this page's login" (reads the form on the current page and stores it as an entry).

**In page**: a fill overlay appears next to login inputs; when the database is locked you can type the master password in the popup or in a standalone unlock window, and the interrupted fill, save or copy **resumes automatically**. The list still renders while locked (from the local cache), but plaintext requires unlocking.

**Post-login save prompt**: after submitting a login form, if the site has no such account the bubble asks "Save this account to KPVault?"; if the account exists with a **different password** it asks "Overwrite with the new password?"; **if the password matches, nothing is shown at all**.

**Context menu**: right-click the input you want to fill → `KPVault` → `Fill username` / `Fill password` (separate items). This path does not rely on "we recognized the input" (the overlay depends on selectors like `input[type=password]`, which custom widgets and unlucky focus timing defeat); the browser decides what counts as editable, so it works on unusual page structures.

The semantics are simple: **fill exactly the field you right-clicked**. No pairing heuristics, no "does this look like a login form" — "Fill username" on a password field writes the username there, and `input` / `textarea` / `contenteditable` are all supported. One match on the site fills directly; several matches open a small candidate panel; if the database is locked an unlock window appears first and the value is written back into the same field afterwards.

The only difference between the two items is the data source: **the username comes straight from the extension cache with no decryption at all**, so it works while locked; the password must be decrypted.

**When the app is not running**: clicking an account (page overlay, context-menu "Fill password", or the popup list) **launches the app and finishes the fill afterwards** — the browser asks "Open KPVault?" the first time; click "Open" (tick "Always allow" to stop being asked). If the app comes up locked, its window is already in front with the password field focused; enter the master password and the fill happens automatically, without switching back. Waiting caps at 20 s (plus up to 90 s for unlocking). If you do not want web pages launching the app, untick "Launch KPVault when unreachable" at the bottom of the popup — you will then only see a hint and can start it yourself. Auto-launch and the button both rely on the `kpvault://` protocol registered by the installer (development mode and portable/green builds do not register it). If the app is running but unreachable, check that the bridge is enabled, that no "port in use" error is shown, and that the port matches (default `47823`, the app never shifts it; the extension probes 47823–47832 and corrects itself).

> ⚠️ **Security trade-off**: "Confirm in app before filling" is **off** by default, so clicking fill/copy hands plaintext to the browser or the clipboard directly; the pairing token is the only barrier — any local program that obtains it can read plaintext. If you do not trust the machine, turn that switch on (fill and copy then show a confirmation dialog first).

---

## 10. Session, tray and shortcuts

**Auto-lock**: after 1/5/15/30 minutes idle (default 5). Sleep, shutdown, screen lock, switching away from the app, and a detected system clock jump over 5 minutes lock immediately. "Lock" in the top bar or `Ctrl+L` locks manually.

**Tray**: closing the window **minimizes to tray** by default (configurable to quit instead); left click shows/hides the window, the right-click menu has show main window, lock, open database folder, global hotkey toggle, and **Quit**. The tray is the default exit path, and its icon reflects locked/unlocked.

| Shortcut | Action |
| :--- | :--- |
| `Ctrl+F` | Focus the search box |
| `Ctrl+N` | New entry |
| `Ctrl+Shift+N` | New group |
| `Ctrl+L` | Lock now |
| `Ctrl+C` | Copy the current entry's password when no text is selected (normal copy otherwise) |
| `Delete` | Delete the current entry (still needs "Confirm delete") |
| `Esc` | Cancel group input / leave editing / clear filters |
| `Ctrl+Shift+Space` | Global show/hide window (**off by default**, enable in settings or the tray; registration fails if taken by the system or an IME) |
| `Ctrl+X / V / A / Z / Y` | Standard editing (restored by the app since the system menu is gone) |

Inside inputs only `Esc` / `Ctrl+F` / `Ctrl+L` are intercepted; everything else passes through so normal editing is unaffected.

---

## 11. Update checks

- On by default. One check 30 seconds after start, then every 24 hours; the version button in the top bar checks manually.
- The manifest `update.json` lives in the same GitHub `latest` Release as the installer (anonymous, constant URL) and the client only trusts that address; the allowed download domains are hard-coded and never taken from the manifest.
- When an update exists the version button turns into "New version vX.Y.Z" and highlights; open it to read the notes, then "Open download page".
- **The app never downloads or installs by itself**: the installer is unsigned, and silent installation would be riskier than the convenience is worth. Download `KPVault-Setup.exe` and install over the old version. **Since v1.44 the database lives in `%APPDATA%\KPVault\data\`, untouched by upgrades and uninstalls**; when upgrading from v0.2.2 or earlier the installer migrates `<install dir>\data\main.kdbx` **before** the old version is uninstalled (skipped entirely if the target already exists — never overwritten). Regular backups are still recommended, but you no longer have to move the database by hand.

---

## 12. Data and config locations

| Item | Location |
| :--- | :--- |
| Password database | `%APPDATA%\KPVault\data\main.kdbx` (fixed, not configurable; no longer in the install dir since v1.44) |
| App config | `%APPDATA%\KPVault\config.json` (no master password) |
| Uninstall-protection copy | `%APPDATA%\KPVault-vault-keep\main.kdbx` (only reclaimed after an uninstall with `--delete-app-data`; unused in a normal uninstall) |
| URL protocol registration | Written at install time to `HKCU\Software\Classes\kpvault` (`kpvault://launch`), removed on uninstall |
| Automatic / manual backups | `%APPDATA%\KPVault\backups\` |
| Sync state | `%APPDATA%\KPVault\sync-state.json` |
| Browser extension | `<install dir>\resources\browser-extension\` |

The config holds the WebDAV app password and the browser pairing token, both preferably encrypted with the OS `safeStorage`; **never sync these files to somewhere you do not trust**.

---

## 13. FAQ

| Symptom | What to do |
| :--- | :--- |
| "Cannot find keepassxc-cli" on start | Make sure [KeePassXC](https://keepassxc.org/download/) ≥ 2.7.0 is installed or unzipped; use "Specify KeePassXC folder" for the portable folder (or pick the exe directly), then "Detect again" |
| WebDAV returns 403 | Use the **app password**, not the login password; disable Nutstore WeChat two-factor auth |
| WebDAV says the certificate is not trusted | Self-signed cert on a home NAS → tick "Allow self-signed certificate" in the WebDAV section of Settings. Never for a public service |
| WebDAV sync fails with `HTTP 409` | The remote folder sits on the NAS root; a NAS root is a list of shared folders, so use `/shared-folder/kpvault` |
| Extension says "cannot reach local app" | App not running → clicking an account launches it (or use "Launch KPVault"; click "Open" in the browser dialog the first time). Running → check the bridge is enabled, no "port in use" error (often another KPVault in the tray), and the port matches (default `47823`, never shifted). If it says the port was corrected, click "Refresh list" once. "Re-pair" is only for a wrong token |
| Closing the window does not exit | It minimizes to tray by default; use "Quit" in the tray menu, or change "When closing the window" to "Quit directly" |
| Forgot the master password | Unrecoverable. Backups are encrypted databases too, so they cannot be opened either — the database must be recreated |
| Switched to a new computer | Use "Restore from WebDAV…" on the second onboarding screen; **do not create a new database** |
| "No database yet" after upgrading | **Only with v0.2.2 and earlier**: the database used to live in the install directory and the uninstaller removed it. Copy the newest `auto-*.kdbx` from `%APPDATA%\KPVault\backups\` to `%APPDATA%\KPVault\data\main.kdbx`, or use "Restore from WebDAV…"; **do not click "Create database"**. If the file is gone, the in-app backup panel cannot restore directly (it requires the current database to exist) |
| Deleted an entry and want it back | No undo in the app; restore a backup (the current database is saved aside first) |
| Two files on the download page | `KPVault-Setup.exe` is the app, `KeePassXC-2.8.0-x64.zip` is the required runtime — you need both |

---

## 14. License and third-party components

- This project declares `GPL-3.0-only` in `package.json`.
- **The installer does not distribute KeePassXC or its Qt dependencies**; users install KeePassXC themselves. Its name and icon belong to the KeePassXC project — follow its own license when using or redistributing them.
- The browser fill extension is an in-house implementation and does not use the official KeePassXC-Browser extension or protocol.
