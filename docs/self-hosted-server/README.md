# Self-hosted Jellyfin Server Guide

> This guide documents one Windows-based Jellyfin deployment as a reusable example. It is separate from the Jellyfin Desktop client code in this repository. Paths, ports, DNS records, hardware, and service names vary by installation.

## Architecture

```text
Family/friends' devices
        | HTTPS
        v
Public DNS name (your domain)
        |
        v
Caddy reverse proxy (TCP 443)
        |
        v
Jellyfin Server (TCP 8096, preferably not exposed directly to the internet)
        |
        v
Local media folders and Jellyfin data
```

## Example Windows layout

- Jellyfin application: `C:\Program Files\Jellyfin\Server`
- Jellyfin application data: `C:\ProgramData\Jellyfin\Server`
- Caddy directory: `C:\Caddy`
- Caddy configuration: `C:\Caddy\Caddyfile`

Confirm paths and the active data directory on your own system before using commands. The running Jellyfin process command line can show its `--datadir` argument.

## Installation and configuration principles

1. Install Jellyfin using the official distribution for the target operating system.
2. Create media libraries using stable local paths and grant Jellyfin only the file permissions it needs.
3. Set up Caddy as a reverse proxy for the Jellyfin HTTP endpoint. Replace example hostnames with your own domain; never publish real secrets or private configuration.
4. Configure DNS and router/firewall rules deliberately. Typically, expose HTTPS on TCP 443 to Caddy and keep Jellyfin's TCP 8096 private to the host/LAN.
5. Test local access first, then HTTPS from a separate network.
6. Configure individual Jellyfin user accounts for family and friends.
7. Check current Jellyfin, Caddy, operating-system, plugin and GPU-driver compatibility before updates or changes.

A sanitised starter configuration is available at [`Caddyfile.example`](./Caddyfile.example). It uses the placeholder hostname `jellyfin.example.com`; replace it with a domain you control. It is an example, not a copy of the live server configuration.

Do not copy a Caddyfile blindly from another installation. Validate the actual configuration before restarting Caddy, and keep a secure copy of the known-working file.

## Windows diagnostics

Run PowerShell as appropriate for your permissions.

Check whether Jellyfin is running:

```powershell
Get-Process jellyfin -ErrorAction SilentlyContinue
```

Inspect the running process and its data-directory argument:

```powershell
Get-CimInstance Win32_Process -Filter "Name='jellyfin.exe'" |
    Select-Object ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Check the local HTTP listener:

```powershell
Test-NetConnection 127.0.0.1 -Port 8096
```

Validate a Caddyfile from the Caddy installation directory:

```powershell
.\caddy.exe validate --config C:\Caddy\Caddyfile
```

Use the actual installed path if it differs.

## Backup and restore

Jellyfin's data directory may include configuration, database files, plugins, metadata, scheduled tasks and other state. A database-only copy is not a complete server backup.

Recommended procedure:

1. Tell active viewers that a short interruption is coming.
2. Stop Jellyfin cleanly using the installation's normal service/tray mechanism.
3. Confirm the Jellyfin server process has exited.
4. Copy the complete data directory, excluding cache/logs only if deliberately desired.
5. Check that the destination contains real files, including the main database and configuration.
6. Review Robocopy's summary; codes 0–7 generally indicate no fatal copy failure, while 8 or higher indicates failure.
7. Restart Jellyfin and test login, libraries, playback and remote HTTPS access.
8. Store another verified copy on a separate physical drive. A backup on the same system disk does not protect against disk failure.

Example copy command after Jellyfin is fully stopped:

```powershell
$source = "C:\ProgramData\Jellyfin\Server"
$destination = "E:\Backups\Jellyfin-YYYYMMDD"
robocopy $source $destination config data metadata plugins root /E /COPY:DAT /R:2 /W:2 /XJ
```

Replace the destination with a verified backup drive and a unique folder. Confirm the source paths before running. This example intentionally copies the named important folders; adjust the folder list to your backup policy. Do not run a live database copy and assume it is consistent.

Keep backups private. Jellyfin configuration and database files may contain personal information, user details, tokens or other sensitive data. Never commit live backups, media, passwords, tokens, private keys, reset files, or unredacted configuration to GitHub.

## Hardware acceleration

Hardware transcoding depends on the Jellyfin version, operating system, GPU, drivers and supported codecs. Confirm compatibility in current official documentation before changing settings. Test a sample transcode and monitor playback/logs; do not assume GPU detection means Jellyfin is configured to use it.

## Security and maintenance

- Keep the operating system supported and patched.
- Update Jellyfin, Caddy and plugins deliberately, after reviewing compatibility and maintaining a rollback copy.
- Use unique user accounts and least-privilege access.
- Keep Jellyfin's backend port private where possible.
- Do not disable VPN, firewall, DNS or router settings as a troubleshooting shortcut without checking their impact.
- Test automatic startup after a reboot if the server is intended to run unattended.
- Document changes, validation results and rollback steps.

## Change log

- **2026-10-20:** Added a sanitised Caddyfile example with a placeholder hostname and linked it from this guide.

## Official resources

- [Jellyfin documentation](https://jellyfin.org/docs/)
- [Jellyfin downloads](https://jellyfin.org/downloads/)
- [Caddy documentation](https://caddyserver.com/docs/)
