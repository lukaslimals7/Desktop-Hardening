# Desktop-Hardening
A comprehensive security hardening suite for Ubuntu, Linux Mint, and Debian-based desktop environments.

# Ubuntu/Debian Desktop Hardening

A comprehensive security hardening suite for Ubuntu, Linux Mint, and Debian-based desktop environments. Adapted from the [konstruktoid/hardening](https://github.com/konstruktoid/hardening) server hardening project, modified to preserve desktop functionality while maintaining a strong security posture.

## Supported Distributions

- Ubuntu 22.04 / 24.04
- Linux Mint (all recent versions)
- Debian 11 / 12 / 13
- Any Debian-based distribution using `apt` and `systemd`

## Desktop Environment Support

Works with any desktop environment:
- GNOME
- KDE Plasma
- XFCE
- Cinnamon
- MATE
- Any other DE using systemd

## Requirements

- Bash shell
- `systemctl` (systemd)
- Root privileges
- Internet connection (for package installation)

## Installation

```bash
git clone https://github.com/lukaslimals7/Desktop-Hardening.git
cd Desktop-Hardening
```

No additional dependencies are required beyond what is already present on a standard Debian-based desktop installation. The script will install any missing prerequisite packages automatically.

## Usage

1. Review and edit the configuration file:

```bash
nano ubuntu-desktop.cfg
```

2. Set the `CHANGEME` variable to confirm you have reviewed the code:

```bash
CHANGEME='reviewed'
```

3. (Optional) Enable automatic IP detection for firewall rules:

```bash
AUTOFILL='Y'
```

4. Run the hardening script:

```bash
sudo bash ./ubuntu-desktop.sh
```

A log file is created at `desktop-hardening-<hostname>-<date>.log` in the working directory.

## What This Hardens

### Kernel and System
- Kernel lockdown mode (confidentiality)
- Conntrack hash table sizing for improved connection tracking
- Hardened sysctl parameters (ASLR, IP spoofing protection, ICMP hardening, IPv6 hardening, BPF JIT hardening, ptrace restrictions)
- Core dumps disabled
- Systemd resource limits configured

### Network Security
- UFW firewall configured (default deny incoming, SSH restricted to admin IPs)
- TCP wrappers (hosts.allow / hosts.deny)
- Disabled unused network protocols (DCCP, SCTP, RDS, TIPC)
- DNS-over-TLS and DNSSEC enabled via systemd-resolved
- NTP configured with low-distance servers

### SSH Hardening
- Strong ciphers enforced (ChaCha20, AES-256-GCM, AES-256-CTR)
- Strong MACs enforced (HMAC-SHA2-512/256 with ETM)
- Strong key exchange algorithms (Curve25519, ECDH, DHE)
- Root login disabled
- Maximum authentication attempts limited
- Weak host keys removed
- Login grace time reduced

### Access Control
- Sudo hardened (PTY required, logging enabled, no password feedback, short timeouts)
- `su` restricted to sudo group
- User account defaults tightened (UMASK 077, password rotation policies)
- Home directory permissions set to 0750

### Password Policy
- Minimum password length: 15 characters
- Minimum character classes: 3
- SHA-512 encryption with high iteration counts
- Failed login lockout configured
- Known-bad password dictionary installed (cracklib)

### File System Security
- `/boot` and `/home` mounted with `nosuid,nodev`
- `/var/log`, `/var/log/audit`, `/var/tmp` mounted with `noexec,nosuid,nodev`
- `/run/shm` and `/dev/shm` hardened
- `.rhosts` and `hosts.equiv` files removed

### Monitoring and Auditing
- Auditd configured with comprehensive rules (sudo, PAM, login, modules, network, systemd)
- AIDE file integrity monitoring with daily checks
- Rkhunter rootkit detection
- PSAD intrusion detection
- Persistent journald logging with compression

### Package Security
- APT configured to reject unauthenticated packages
- Insecure repositories disabled
- Seccomp sandbox enabled
- Automatic removal of unused packages

### Application Restrictions
- AppArmor enforced for all profiles
- Compiler binaries restricted (mode 0750)
- Core dumps disabled system-wide

### USB Security
- USBGuard is **not** installed (desktop users require full USB functionality)
- USB storage kernel module is **not** blacklisted

### Bluetooth and Audio
- Bluetooth kernel modules are **preserved** (keyboards, mice, headphones)
- Sound kernel modules are **preserved**

### Desktop-Specific Adjustments
| Setting | Server | Desktop |
|---|---|---|
| `/tmp` mount | `noexec` tmpfs | Unchanged (preserves app compatibility) |
| `/proc` `hidepid` | `hidepid=2` (hide all processes) | Unchanged (preserves process visibility) |
| Idle action | Screen lock after 15 min | No action |
| Kill user processes | Yes | No |
| Auto-logout (TMOUT) | 600 seconds | Not set |
| Ctrl-Alt-Del | Masked | Enabled (recovery) |
| Root account | Locked | Unlocked |
| Root access restriction | 127.0.0.1 only | Unrestricted |
| Cron access | Root only | All users |
| PATH restriction | Minimal | Default |
| SUID bit removal | 412 binaries | Skipped |
| Issue/motd banner | Replaced with warning | Preserved |
| SSH password auth | Disabled | Enabled |
| SSH X11 forwarding | Disabled | Enabled |
| SSH TCP forwarding | Disabled | Enabled |

## What This Does NOT Do

- Does not install USBGuard (preserves USB device functionality)
- Does not remove development tools (git, compilers, rsync are kept)
- Does not remove Bluetooth or sound drivers
- Does not convert `/tmp` to noexec tmpfs
- Does not hide processes via `hidepid=2`
- Does not lock the root account
- Does not enforce automatic logout

## Uninstalling Changes

This script modifies system configuration files. Before running, backups are created for:

- `/etc/fstab` (saved as `/etc/fstab.bck`)
- `/etc/ssh/sshd_config` (saved with date suffix)
- `/etc/ssh/ssh_config` (saved with date suffix)

To reverse specific changes, consult the backup files and the corresponding script in `scripts/`.

## Project Structure

```
desktop/
├── ubuntu-desktop.sh      # Main entry point
├── ubuntu-desktop.cfg     # Configuration variables
├── misc/                  # Supporting configuration files
│   ├── sysctl.conf        # Hardened kernel parameters
│   ├── audit-base.rules   # Base audit rules
│   ├── audit-aggressive.rules  # Aggressive audit rules
│   ├── audit.header       # Audit rules header
│   ├── audit.footer       # Audit rules footer
│   ├── logrotate.conf     # Log rotation settings
│   └── passwords.list     # Known-bad password dictionary
└── scripts/               # Hardening function scripts
    ├── pre                # Pre-flight checks
    ├── kernel             # Kernel hardening
    ├── sysctl             # Sysctl configuration
    ├── sshdconfig         # SSH hardening
    ├── packages           # Package management
    └── ...                # (47 scripts total)
```

## Security Notice

This script makes significant changes to system configuration. It is recommended to:

1. Test on a non-production system first
2. Ensure you have physical or console access recovery options
3. Review all configuration changes before and after running
4. Keep a backup of critical configuration files

## Contributing

Contributions are welcome. Please ensure changes are tested and do not break desktop functionality.

## License

Apache License 2.0. See [LICENSE](../LICENSE) for details.

## Credits

Developed by **Lukas**.

Adapted from the [konstruktoid/hardening](https://github.com/konstruktoid/hardening) project by Thomas Sjogren, licensed under Apache License 2.0.
