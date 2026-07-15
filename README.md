<div align="center">

# ☁️ cf-manager

**A Cloudflare control room that runs entirely inside Termux — Workers, KV, D1, R2, multi‑account switching, GitHub sync, and one‑tap rollback, all from a phone terminal.**

[![Version](https://img.shields.io/badge/version-3.0.0-7FD858?style=flat-square&logo=semanticrelease&logoColor=white)](./cfmanager.sh)
[![Bash](https://img.shields.io/badge/bash-4.4%2B-4EAA25?style=flat-square&logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)
[![Platform](https://img.shields.io/badge/platform-Termux%20%2F%20Android-3DDC84?style=flat-square&logo=android&logoColor=white)](https://termux.dev)
[![Cloudflare API](https://img.shields.io/badge/Cloudflare-API%20v4-F6821F?style=flat-square&logo=cloudflare&logoColor=white)](https://developers.cloudflare.com/api/)
[![License](https://img.shields.io/badge/license-AGPL--3.0%20%2B%20Commons%20Clause-blue?style=flat-square)](./LICENSE)

<a href="https://dumbCodesOnly.github.io/cfmanager/demo.html">
  <img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&size=15&duration=2600&pause=900&color=7FD858&center=true&vCenter=true&width=560&lines=%24+bash+cfmanager.sh;%E2%9C%93+switched+to+acme-staging;%E2%9C%93+edge-router+deployed+%C2%B7+v42+%C2%B7+3.1s;%E2%9A%A0+checkout-api+p95+spiked+after+v17;%E2%9C%93+checkout-api+restored+to+v16" alt="typing animation of a cf-manager session" />
</a>

**[▶ Watch the live sample session](https://dumbCodesOnly.github.io/cfmanager/demo.html)**

</div>

---

## What this is

`cf-manager` is a single self-contained bash script that gives Cloudflare a
mobile-first admin panel — no dashboard tab, no laptop required. Point it at
one or more Cloudflare accounts (via OAuth, the same flow `wrangler login`
uses) and manage Workers, KV, D1, R2, and Hyperdrive from wherever Termux runs.

It was built for the case the official CLI doesn't optimize for: a narrow
phone screen, spotty connectivity, and wanting to fix something *right now*
without opening a laptop.

## Features

| | |
|---|---|
| ☁️ **Workers** | create, edit, deploy, tail logs (via `wrangler` in a proot-distro container — workerd has no native Android build), inline template editor |
| ↩️ **One-tap rollback** | every deploy is backed up locally first; restoring a bad push is a single confirm |
| 🗂️ **KV** | namespace + key browsing, paginated so a 10,000-key store doesn't flood a small screen |
| 🗄️ **D1** | run queries directly, or copy table data between databases in batches during a rename/sync |
| 🪣 **R2** | bucket + object listing and management |
| ⚡ **Hyperdrive** | list, inspect, and edit configs |
| 👤 **Multi-account** | store several Cloudflare accounts, switch between them without re-authenticating |
| 🔐 **OAuth2 + PKCE login** | the same login flow `wrangler login` uses — no manually pasted API tokens required |
| 🔁 **GitHub sync** | push a worker's source straight to a linked repo branch — deploy and version control in one step |
| 🪝 **Deploy hooks & flows** | post-deploy steps, including auto-registering a freshly deployed worker with a [manager.js](https://github.com/dumbCodesOnly) admin panel |
| 📊 **Analytics tokens** | mint an account-scoped Analytics Read token once, push it to as many workers as you like |
| 🧵 **Live TUI** | a progress table that redraws in place during parallel deploys, tuned for small screens |

## Requirements

```bash
pkg install curl jq openssl git nano
```

Real-time worker logs (`wrangler tail`) need Node + wrangler inside a Linux
container, since `workerd` has no Android/Bionic build:

```bash
pkg install proot-distro
```

`cf-manager` will offer to install `proot-distro`, Ubuntu, Node, and
`wrangler` itself the first time you try to tail a worker — nothing to set
up in advance.

## Usage

```bash
bash cfmanager.sh
```

First run walks you through adding a Cloudflare account (OAuth login in your
phone's browser, redirected back to Termux). Everything after that is menu-driven.

<details>
<summary><b>What the main menu looks like</b></summary>

```
════════════════════════════════════════════════════
                    CF-MANAGER v3.0
════════════════════════════════════════════════════
  ☁ Account: acme-staging  | CF-Manager v3.0.0
────────────────────────────────────────────────────

  1. Switch account
  2. Workers
  3. KV / D1 / R2 / Hyperdrive
  4. Deploy hooks & flows
  5. Settings
  0. Exit

Choice:
```
</details>

## Configuration

A handful of user-tunable settings sit at the top of `cfmanager.sh` — no
separate config file:

| Setting | Default | Purpose |
|---|---|---|
| `CACHE_ENABLED` / `CACHE_TTL` | `true` / `600s` | cache list calls to cut down on API round-trips |
| `BACKUP_MAX_COUNT` | `10` | local worker backups kept per worker before pruning oldest |
| `API_PAGE_*` | varies | page sizes for KV/D1/R2/account listings |
| `D1_COPY_PAGE_SIZE` | `500` | rows per page when copying D1 table data |
| `TUI_POLL_INTERVAL` | `0.12s` | redraw rate for the live deploy progress table |
| `PREFERRED_EDITOR` | `$EDITOR` or `nano` | editor used for inline worker editing |

## Storage & security notes

- Accounts, saved panel passwords, and minted analytics tokens are stored as
  **plain JSON** under `~/script/cfmanager/.cfmanager/`, each file `chmod 600`
  (owner-read/write only). This is a personal, local tool — there is no
  master-password vault as of v3.0; an older encrypted vault from prior
  versions is migrated to plaintext automatically on first run.
- Manager-panel admin secrets (`manager_targets.json`) follow the same
  plaintext/mode-600 model, deliberately kept separate from the main
  accounts store since they're re-enterable panel secrets rather than
  Cloudflare credentials.
- Treat this the way you'd treat any local credential store: don't sync
  `~/script/cfmanager/.cfmanager/` to somewhere you don't control.

## License

Licensed under **AGPL-3.0** with the **Commons Clause**. In short:

- Free to use, run, modify, and self-host — including for internal business use.
- If you modify it and let others use your version over a network, you must
  publish the source of your changes (AGPL-3.0's network-copyleft term).
- You may **not** sell it — the Commons Clause blocks offering a paid product
  or service whose value comes substantially from this script's
  functionality without a separate agreement with the licensor.

See [`LICENSE`](./LICENSE) for the full, binding text — this is a summary,
not legal advice.
