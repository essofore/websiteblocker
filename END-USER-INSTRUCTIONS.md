# WebsiteBlocker — End User Instructions

This guide explains how to install and run `WebsiteBlocker` on Windows from the packaged ZIP.
You will need to use the terminal (console) and enter commands to do this. We do not provide any GUI.

## What It Does
- Periodically evaluates your `config.yaml` and applies Windows Firewall rules to block domains **outside** the configured time windows i.e., the time-windows are a whitelist or _allowlist_ when you are allowed to access the websites.
- On startup, clears previously managed rules (default). On graceful shutdown, clears rules (default).

## What’s in the ZIP
- `WebsiteBlocker.exe` and supporting files (`.dll`, `.deps.json`, `.runtimeconfig.json`).
- `config.yaml`.

## Requirements / Pre-requisites
- **Administrator** rights (required to modify Windows Firewall via `netsh`).
- **.NET 8 Desktop Runtime** must be installed.
  - Download: https://dotnet.microsoft.com/download/dotnet/8.0
  - Verify: `dotnet --list-runtimes` should show `Microsoft.NETCore.App 8.0.x` and `Microsoft.WindowsDesktop.App 8.0.x`.

## Install
1) Create a folder (example): `C:\Program Files\WebsiteBlocker`.
2) Extract the entire ZIP into that folder. Ensure `WebsiteBlocker.exe` and `config.yaml` sit side‑by‑side in the `bin` sub-directory.
3) Edit `config.yaml` as desired. Example:

```
settings:
  blockRefreshMinutes: 5
  clearOnStartup: true
  clearOnExit: true
  windows:
    mon: ["18:00-19:00"]
    tue: ["18:00-19:00"]
    wed: ["18:00-19:00"]
    thu: ["18:00-19:00"]
    fri: ["18:00-19:00"]
    sat: []
    sun: []

rules:
  - domain: ["facebook.com", "instagram.com", "tiktok.com"]
```

Notes:
- `settings.windows` provides the time windows during which access is granted to the domains. Change this for your use-case.
- `clearOnStartup` (default `true`) unblocks all previously managed rules once when the app starts. Strongly recommend to leave **as-is** unless you know what you are doing.
- `clearOnExit` (default `true`) unblocks all managed rules when the app stops gracefully. Strongly recommend to leave **as-is** unless you know what you are doing.
- `domain` Change as necessary and list all the domains you want to block. Free version is capped at 3 domains.

## Run

1) Open an **elevated Command Prompt**. **elevated = `Run as Administrator`**.
2) Create the service (auto‑start on boot):
   - `sc create WebsiteBlocker binPath="C:\Program Files\WebsiteBlocker\bin\WebsiteBlocker.exe" start=auto`
3) Start the service:
   - `sc start WebsiteBlocker`
4) Stop/Start later:
   - `sc stop WebsiteBlocker`
   - `sc start WebsiteBlocker`
5) Uninstall:
   - `sc stop WebsiteBlocker` (ignore any errors if already stopped)
   - `sc delete WebsiteBlocker`
6) Check Status:
   - `sc query WebsiteBlocker`

## Verifying
- View created firewall rules:
  - `netsh advfirewall firewall show rule name=all | findstr SiteTimeGate`
- View logs in Windows Event Log (Application, Source: WebsiteBlocker):
  - PowerShell (elevated):
    - `Get-WinEvent -LogName Application | Where-Object { $_.ProviderName -eq "WebsiteBlocker" } | Select-Object -First 50 TimeCreated, Id, LevelDisplayName, Message`

## Updating Configuration
- Edit `config.yaml` in place. The service automatically reloads when the file timestamp changes (evaluates roughly every 60 seconds).
- To force a clean slate (e.g., after big changes), restart the service. With `clearOnStartup: true`, existing rules are cleared first.

## Uninstall / Cleanup
1) Stop and delete the service (see above).
2) With `clearOnExit: true`, stopping the service clears rules it managed. If rules remain (e.g., forced termination), you can:
   - Start the app once and stop it cleanly to trigger cleanup; or
   - Manually remove rules by name using `netsh` if needed.

## Troubleshooting
- “The framework 'Microsoft.NETCore.App, version 8.0.0' was not found”: Install the .NET 8 Desktop Runtime (see Requirements).
- Permission errors: Ensure you’re running as Administrator.
- No rules take effect: Verify domains resolve to IPs and that the current local time falls outside allowed windows.
- Network adapters/VPNs: Rules are IP‑based; policy refresh (default every 5 minutes when blocked) updates to new IPs over time.
- Some security or antivirus software may stop WebsiteBlocker from working because it modifies your system settings.Please let us know if this happens.

## Escape Hatch (SOS)

This is a self-help tool, not a lockbox. You can bypass it anytime — we even show you how. The goal is to make the healthy choice easier, not to take away control.
You have several jailbreak options:

1. **Easiest**: Stop the service by running `sc stop WebsiteBlocker` from an **elevated Command Prompt**. This will remove all protections put in place by the program.
2. Edit `config.yaml`. The service automatically reloads when the file timestamp changes (evaluates roughly every 60 seconds).
3. As last resort, you can run `sos-kill-switch.ps1` from **elevated PowerShell prompt** as follows:

```
.\scripts\sos-kill-switch.ps1 -RulePrefix "SiteTimeGate:" -ServiceName "WebsiteBlocker"
```

If Powershell refuses to run the script, run:

```
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

## I don't want to be able to bypass the tool

Give admin access of the computer and the `config.yaml` file to someone else. This will prevent you from "cheating" and at same time have a SOS option.

## [Questions?](https://github.com/essofore/websiteblocker/issues)
