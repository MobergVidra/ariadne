<h1 align="center">Ariadne</h1>
<p align="center"><em>Navigate the labyrinth of development without losing the thread.</em></p>

A unified dashboard for your development flow. Aggregates your open tasks, pull requests, reviews, tickets, and error reports from every service you work with into one local app — so you can spend less time jumping between tabs and more time actually shipping.

**Services it connects to:**

- **GitHub** — pull requests, reviews, comments, notifications (across all repos or a configured subset)
- **Jira Cloud** — tickets assigned to or watched by you, sprint/status filtering, full ticket view with comments
- **Azure DevOps** — pull requests and repo activity
- **AWS CodeCommit** — pull requests from AWS-hosted repos
- **Sentry** — unresolved errors with level/project filters, one-click "create Jira ticket" deep-links
- **Anthropic Claude** *(optional)* — AI chat and PR summaries via the Anthropic API or the `claude` CLI

Built to solve the "what should I actually be working on right now?" problem — surfacing stale reviews, unanswered comments, blocked PRs, overdue tickets, and error spikes in a single glance.

## Latest release — v1.2.0

- [Linux (tar.gz)](https://github.com/MobergVidra/ariadne/releases/download/v1.2.0/ariadne-v1.2.0-linux.tar.gz)
- [macOS (dmg)](https://github.com/MobergVidra/ariadne/releases/download/v1.2.0/ariadne-v1.2.0-macos.dmg)
- [Windows (zip)](https://github.com/MobergVidra/ariadne/releases/download/v1.2.0/ariadne-v1.2.0-windows.zip)

See [all releases](https://github.com/MobergVidra/ariadne/releases).

## Install

| OS | Artifact | First launch |
|---|---|---|
| Linux (x86_64) | `ariadne-<version>-linux.tar.gz` | extract, run `./ariadne/ariadne` |
| macOS | `ariadne-<version>-macos.dmg` | mount, drag to Applications. **First launch:** right-click the app → **Open** (unsigned build; Gatekeeper warning is expected) |
| Windows | `ariadne-<version>-windows.zip` | extract, run `ariadne.exe`. **First launch:** SmartScreen will warn — click **More info → Run anyway** |

> **Upgrading from the pre-rebrand `task-helper` v1.0.0 build?** Your data directory (`~/Library/Application Support/task-helper/` on macOS, equivalents on Linux/Windows) is automatically renamed to the new Ariadne path on first launch; tokens stored in the OS keychain are migrated transparently. You won't need to re-enter anything.

## First run

1. Launch the app. The window opens to an empty dashboard with a warning about missing config.
2. Open **Settings** and fill in tokens for the services you use (see the token prerequisites below).
3. Click **Save**. Tokens are stored in the OS keyring (GNOME Keyring / macOS Keychain / Windows Credential Manager) — not on disk.
4. Sync runs automatically; use the tray or the header's **Sync Now** button for immediate refresh.

Non-secret settings + the cached PR/ticket/notification data live in the per-user app data dir:

- **Linux:** `~/.local/share/Ariadne/`
- **macOS:** `~/Library/Application Support/Ariadne/`
- **Windows:** `%APPDATA%\Ariadne\`

## Service tokens

Create these on the service side and paste into Settings:

- **GitHub** — personal access token with `repo` + `notifications` scopes. If your org uses SSO, authorize the token in GitHub → Settings → Developer settings → Personal access tokens → Configure SSO.
- **Jira Cloud** — API token from [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens). Also enter your base URL and email.
- **Azure DevOps** — PAT with Code: Read + Work Items: Read. Enter organization + project name.
- **AWS CodeCommit** — IAM access key + secret, IAM user needs CodeCommit read permissions, pick a region.
- **Sentry** — auth token from Sentry → Settings → Auth Tokens with scopes `event:read`, `project:read`, `org:read`. Also enter the org slug.
- **Anthropic** *(optional)* — API key from console.anthropic.com for AI chat + PR summaries. Alternatively, install the `claude` CLI (the app auto-detects it).

## Tray + notifications

- Close the window → app goes to the system tray; the background sync keeps running.
- Tray menu: **Dashboard / Pull Requests / Tickets / Notifications / Sentry / Settings** (only pages with configured services show up), plus **Start on login**, **Check for updates**, **Quit**.
- Desktop notifications fire only when the window is hidden. Per-event toggles live under **Settings → Desktop App**:
  - New GitHub/Jira notifications
  - PR merged
  - Sync errors
  - Stale PR crossed your threshold (default 4 days, configurable)
- Click any notification → reopens the window and navigates to the relevant PR or ticket.

## Updates

Check for updates manually from **Settings → Desktop App → Check for updates**. For now, grab the latest artifact from GitHub Releases.

## Uninstall

Drag/remove the app, then optionally clear user data:

- **Linux:** `rm -rf ~/.local/share/Ariadne ~/.config/autostart/Ariadne.desktop`
- **macOS:** `rm -rf "~/Library/Application Support/Ariadne" ~/Library/LaunchAgents/com.Ariadne.plist`
- **Windows:** delete `%APPDATA%\Ariadne`, remove the `Ariadne` entry under `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`


## Troubleshooting

### Linux: blank/white window on launch

`./ariadne/ariadne` opens but the window stays white. Almost always a missing QtWebEngine runtime dependency.

**Fedora / RHEL / CentOS:**
```bash
sudo dnf install -y \
  nss nspr alsa-lib cups-libs \
  atk at-spi2-atk at-spi2-core \
  libxkbcommon xcb-util-cursor \
  libXcomposite libXdamage libXfixes libXrandr libXcursor libXtst libXScrnSaver \
  libdrm libgbm pango cairo \
  mesa-libGL mesa-libEGL \
  gtk3 libsecret
```

**Debian / Ubuntu:**
```bash
sudo apt-get install -y \
  libnss3 libnspr4 libasound2 libcups2 \
  libatk1.0-0 libatk-bridge2.0-0 libatspi2.0-0 \
  libxkbcommon0 libxcb-cursor0 \
  libxcomposite1 libxdamage1 libxfixes3 libxrandr2 libxcursor1 libxtst6 \
  libdrm2 libgbm1 libpango-1.0-0 libcairo2 \
  libgl1 libegl1 \
  libgtk-3-0 libsecret-1-0
```

Then run `./ariadne/ariadne` again.

### Diagnostic script

If the install above doesn't fix it, run the diagnostic (prints distro, missing `.so` libs, log tail):

```bash
curl -sL https://raw.githubusercontent.com/MobergVidra/task-helper/master/ops/diagnose_linux.sh | bash
```

Paste the output when opening an issue.

### No log file appears

Logs live at:
- **Linux:** `~/.local/share/Ariadne/logs/ariadne.log`
- **macOS:** `~/Library/Application Support/Ariadne/logs/ariadne.log`
- **Windows:** `%APPDATA%\Ariadne\logs\ariadne.log`

If the log file is missing, the backend failed before it could initialize. Run the binary from a terminal so you can see stderr — on Linux that's `./ariadne/ariadne 2>&1 | tee /tmp/ariadne.log`.

### First launch blocked by Gatekeeper (macOS) / SmartScreen (Windows)

Builds are unsigned. See the first-launch column in the install table above.
