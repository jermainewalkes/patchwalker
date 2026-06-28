<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="patchwalker-lockup-ondark.svg">
    <img src="patchwalker-lockup.svg" width="340" alt="PatchWalker">
  </picture>
</p>

<p align="center"><strong>Centralised, agentless patch management for Windows and Linux.</strong></p>

<p align="center">
  <a href="https://patchwalker.com">Website</a> &middot;
  <a href="../../releases/latest">Download</a> &middot;
  <a href="https://patchwalker.com/docs.html">Getting started</a> &middot;
  <a href="https://patchwalker.com/support.html">Support</a>
</p>

<p align="center">
  <img src="https://img.shields.io/github/v/release/jermainewalkes/patchwalker?label=download&color=2563eb" alt="Latest release">
  <img src="https://img.shields.io/badge/controller-Windows-2563eb" alt="Runs on Windows">
  <img src="https://img.shields.io/badge/manages-Windows%20%26%20Linux-555" alt="Manages Windows and Linux">
  <img src="https://img.shields.io/badge/agents-none-2ea44f" alt="Agentless">
  <img src="https://img.shields.io/badge/licence-Freeware-2ea44f" alt="Freeware">
</p>

---

## What is PatchWalker?

PatchWalker keeps an entire fleet of Windows and Linux machines patched and compliant from a single **Windows Server** that acts as your **Controller**, with **no agents** to deploy, update or troubleshoot on your endpoints. It manages Windows over WinRM (HTTPS) and Linux over SSH, so you enrol a machine once and manage its operating-system updates, third-party software, schedules, compliance and more from a single web console.

It's **free to use**, runs entirely on your own infrastructure, and sends no telemetry anywhere.

> This repository hosts **release downloads only**. PatchWalker is proprietary freeware; the application source is not published here.

## Screenshots

**Dashboard** &mdash; estate health, pending updates and per-group compliance at a glance:

![PatchWalker dashboard](https://patchwalker.com/img/screenshots/dashboard.png)

**Updates** &mdash; review and install Windows and Linux updates across the estate:

![PatchWalker updates view](https://patchwalker.com/img/screenshots/updates.png)

More screenshots and a full feature tour at **[patchwalker.com/features.html](https://patchwalker.com/features.html)**.

## Why agentless?

Agent-based tools mean another piece of software to install on every machine, keep updated, secure and debug. PatchWalker uses the management channels your machines already have:

- **Windows** &mdash; WinRM over HTTPS (port 5986), authenticated with a per-machine certificate or the Controller's Kerberos identity.
- **Linux** &mdash; SSH (port 22), with key or password credentials.

Nothing is installed on the targets. Enrolment configures the secure channel and registers the machine; from then on it's managed.

## Features

### Updates
- Check and install **Windows Update** and **Linux package** updates across the estate.
- Per-machine or **bulk** install with a live worklist; on-demand or scheduled.
- Track **outdated third-party applications** (built-in catalog backed by the winget community manifests, or your own catalog source) and push updates.

### Software deployment
- Upload your own **installer packages** and deploy them to Windows and Linux targets.
- Group packages into ordered **bundles** for multi-step installs.

### Automation &amp; scheduling
- Recurring **install schedules** scoped to all machines, a group, or a hand-picked set.
- Each schedule installs **operating-system updates, software updates, or both**, inside a **maintenance window**, with a per-schedule **reboot policy**.
- **Auto-grouping rules** place newly enrolled machines into the right groups automatically.
- Run **ad-hoc or saved scripts** against machines.

### Visibility &amp; reporting
- A live **dashboard** of estate health, pending updates and per-group compliance, with click-through detail behind every figure.
- **CSV / print exports** and **scheduled compliance reports** by email.
- A built-in, browser-based **remote shell** (WinRM / SSH) for quick hands-on work.

### Air-gapped Ubuntu Pro / ESM
- A built-in **package mirror** and **Ubuntu Pro contracts proxy** let you attach machines to Ubuntu Pro and serve **ESM** updates to endpoints that can't reach the internet &mdash; the Controller fetches once and serves your estate.

### Operations
- **Encrypted backups** of configuration and state, on demand or on a schedule, to a local or network (UNC) folder.
- **Email / SMTP** notifications with editable mail templates.

## Security

Security is built into how PatchWalker connects, stores secrets and controls access:

- **Encrypted transports only** &mdash; WinRM over HTTPS (5986) and SSH (22). Plain-text WinRM is not used.
- **Strong target authentication** &mdash; per-machine **WinRM client certificates** (workgroup or domain), or **Kerberos** using the Controller's gMSA / managed domain identity, so no target passwords are stored at all. Optional **WinRM certificate pinning** ties a target to its expected listener certificate.
- **Secured enrolment** &mdash; machines enrol against a Controller-issued token, so a stranger can't register or impersonate the Controller.
- **Machine-bound secrets** &mdash; the Controller's sealing key is protected by the **TPM** where available, with a DPAPI (LocalMachine) fallback; the WinRM client certificate's private key is **non-exportable**. Stored credentials are encrypted with a **machine-scoped key**, so a backup lifted to another machine cannot decrypt them.
- **Encrypted backups** &mdash; AES-GCM with a key derived from your passphrase (PBKDF2); backups carry secrets and are only fully usable on the same Controller.
- **Access control** &mdash; **role-based permissions** with custom roles, **Microsoft Entra (Azure AD) single sign-on** via OpenID Connect with **group-to-role mapping**, and an always-available **break-glass administrator**. Local passwords are stored as **BCrypt** hashes, and repeated failed logins are **throttled**.
- **Full audit trail** &mdash; additions, changes, deletions and errors are logged; **audit entries are always retained** and errors are mirrored to the **Windows Event Log**. Two-tier logging separates operator-facing activity from low-level diagnostics.
- **TLS for the console** &mdash; import your own PFX to serve the web interface over HTTPS.
- **No telemetry** &mdash; PatchWalker runs entirely on your Controller and talks only to your machines and the update sources you enable. It never phones home.

## How it works

```
                         +-------------------------------+
                         |        PatchWalker            |
                         |        Controller             |
                         |  (one Windows machine)        |
                         |                               |
   Admin browser  <--->  |  Web console + job engine     |
                         |  inventory | schedules | logs |
                         +---------------+---------------+
                                         |
                 WinRM / HTTPS (5986)    |    SSH (22)
                    +--------------------+--------------------+
                    |                                         |
            +-------v-------+                         +-------v-------+
            |  Windows host |   ... your estate ...   |  Linux host   |
            |  (no agent)   |                         |  (no agent)   |
            +---------------+                         +---------------+
```

## Requirements

**Controller**
- 64-bit Windows &mdash; Windows Server 2016 or later, or Windows 10/11.
- The installer is self-contained (the .NET runtime is bundled &mdash; nothing else to install).
- Outbound internet for update metadata and optional mirroring.
- Domain join is optional: a **Domain / gMSA** install enables Kerberos management of domain machines; **Standalone** uses certificate-based WinRM and works in workgroups.

**Managed machines (no agent)**
- **Windows:** WinRM over HTTPS (5986) reachable from the Controller.
- **Linux:** SSH (22) reachable from the Controller.

## Getting started

1. **[Download](../../releases/latest)** the installer and verify its SHA-256 (below).
2. Run it on your chosen Controller; pick **Standalone** or **Domain / gMSA**.
3. Open the console, create the break-glass administrator, and enrol your first machine.

Full walkthrough: **[patchwalker.com/docs.html](https://patchwalker.com/docs.html)** &mdash; and the complete reference is built into the app under **Manual**.

## Download &amp; verify

Get the latest installer from **[Releases](../../releases/latest)**. Each release publishes the installer's SHA-256; confirm your copy matches before running it:

```powershell
# Windows (PowerShell)
Get-FileHash .\PatchWalkerSetup-<version>.exe -Algorithm SHA256
```
```bash
# Linux / macOS
shasum -a 256 PatchWalkerSetup-<version>.exe
```

> PatchWalker is not yet code-signed, so Windows SmartScreen may warn about an unknown publisher. After verifying the SHA-256, choose **More info &rarr; Run anyway**.

## Licence

PatchWalker is **proprietary freeware** &mdash; free to use for personal or business purposes, with no feature limits or licence keys. It is **not open source**: redistribution and modification are not permitted. See **[LICENSE.txt](LICENSE.txt)** for the full End User License Agreement (governed by the laws of England and Wales).

## Support &amp; feedback

Found a bug or have an idea? Use **Make a Wish** inside the app — it sends your feedback straight to the developer.

## Support development

PatchWalker is **free**, built and maintained by one person. If it saves you time, a donation keeps it improving and is hugely appreciated:

<p align="center">
  <a href="https://ko-fi.com/patchwalker"><img src="https://img.shields.io/badge/Support%20PatchWalker-Ko--fi-FF5E5B?style=for-the-badge&logo=kofi&logoColor=white" alt="Support PatchWalker on Ko-fi"></a>
</p>

<p align="center"><sub>&copy; 2026 Jermaine Walkes. PatchWalker is proprietary freeware.</sub></p>
