# Portable Home Server Migration Plan

## Goal

Make this Windows PC's Jellyfin/Caddy setup recoverable on replacement hardware with minimal manual work. This document is a plan, not proof that backups or restore tests have succeeded.

## Current architecture (known)

- Operating system: Windows 10 Home (current machine; plan a supported Windows version for future hardware).
- Jellyfin executable: `C:\Program Files\Jellyfin\Server\jellyfin.exe`
- Jellyfin data directory: `C:\ProgramData\Jellyfin\Server`
- Jellyfin local HTTP endpoint: `127.0.0.1:8096`
- Caddy directory: `C:\Caddy`
- Public access: custom domain through Caddy HTTPS on TCP 443.
- Media storage: currently on existing local storage; a NAS is planned later.
- Public example configuration: [Caddyfile.example](./Caddyfile.example).

Confirm current paths, ports, service names, DNS, router rules and storage locations before using this plan. Do not assume the LAN IP will remain the same.

## Portability principles

1. Treat media, application data, configuration, and secrets as separate categories.
2. Keep a private, verified backup outside the system disk. A copy on the same C: drive is not disaster recovery.
3. Keep secrets (tokens, private keys, credentials, live config containing sensitive details) out of the public repository. Store them in an encrypted/private backup with access restricted to the owner.
4. Prefer stable media paths. When the NAS is introduced, use a stable share name/path and update Jellyfin libraries deliberately; verify permissions and playback before relying on it.
5. Use scripts that are idempotent where practical, log their actions, check prerequisites, and stop on errors. Do not automate destructive changes without confirmation.
6. Keep a human-readable recovery guide alongside the scripts.
7. Record software versions and installer sources; re-check compatibility before installing or restoring plugins, drivers, or hardware acceleration settings.
8. Do not rely on a full Windows disk clone as the only backup. A clean OS install plus restore of application data/config is generally more portable across different hardware.

## Backup sets

### A. Jellyfin state

With Jellyfin stopped cleanly, back up the active data directory, currently believed to be `C:\ProgramData\Jellyfin\Server`. Include the database, configuration, metadata, plugins and other required state. Exclude cache/logs only deliberately. Verify that the backup contains real files and the main database, and review the copy tool's exit status.

### B. Caddy and startup

Back up privately:
- The working `C:\Caddy\Caddyfile` (redact domain/IP details before any public sharing if desired).
- `caddy.exe` version information and its trusted download/source.
- WinSW service wrapper configuration and service install/start instructions.
- DNS-provider settings and router port-forwarding notes (never publish credentials or tokens).

Keep the public `Caddyfile.example` as a template only; it is not the live config.

### C. Inventory

Maintain a text inventory of:
- Windows edition/build and machine hardware.
- Installed applications and versions, installer sources, plugins, and relevant drivers.
- Jellyfin library names and media paths (avoid including private media filenames in public docs).
- Ports, local endpoints, firewall rules and startup mechanisms.
- Required account access and where recovery credentials are securely stored.

### D. Media

Until the NAS exists, include media storage in the backup plan or document exactly where the media lives and how it will be copied. The Jellyfin database backup alone does not include media files.

## Replacement-PC recovery order

1. Install a supported Windows version and updates; confirm network connectivity.
2. Install the documented prerequisites and Jellyfin version, checking compatibility first.
3. Stop Jellyfin before restoring its saved data directory to the correct active data path.
4. Reinstall Caddy and its service wrapper using trusted sources; restore the private Caddyfile and service configuration.
5. Recreate DNS/router/firewall settings carefully; expose only required ports. Keep Jellyfin's backend port private where possible.
6. Reconnect media storage using the documented paths and permissions.
7. Start Jellyfin and Caddy. Validate local Jellyfin access, login, users, libraries, playback, HTTPS certificates and remote access.
8. Check logs and confirm automatic startup after a reboot.
9. Keep the old backup unchanged until the replacement has passed these tests.

## NAS migration (future)

Before moving media:
- Choose a NAS share protocol and a stable share path appropriate for Windows.
- Create a dedicated NAS account with only the permissions Jellyfin needs.
- Ensure the share is available before Jellyfin scans libraries or starts scheduled tasks.
- Move/copy a small test folder first, verify file integrity and playback, then migrate the remaining media.
- Update library paths in a controlled change and preserve the old location until verification completes.

## Readiness checklist

- [ ] Full Jellyfin data backup completed and verified.
- [ ] Backup copied to a separate physical device.
- [ ] Caddy config and service setup backed up privately.
- [ ] Media locations documented and media backup strategy defined.
- [ ] Software/version inventory recorded.
- [ ] Recovery procedure tested on a spare drive or test environment.
- [ ] Replacement/recovery test confirms login, libraries, playback, HTTPS and auto-start.

Do not mark the system disaster-recovery-ready until the separate backup and a restore test are successful.

## Change log

- **2026-10-20:** Initial portability plan added. Current setup details are documented as known facts; future NAS and recovery steps remain planned, not yet tested.
