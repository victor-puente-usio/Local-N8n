# WSL on Windows 11 — Detailed Ubuntu Install & Usage Guide

This guide covers installing, configuring, and using Windows Subsystem for Linux (WSL) with an Ubuntu distribution on Windows 11. It focuses only on WSL itself — no Docker, no extra tooling — so you get a clear, step-by-step WSL reference.

Contents
...
Quick install (recommended)

1. Open PowerShell as Administrator.
2. Run the single command that installs WSL and the default Ubuntu distribution:

```powershell
wsl --install
```

3. Reboot if the installer requests it.
4. Launch the installed Ubuntu app from the Start menu or by running `wsl` in PowerShell/Command Prompt. On first launch you'll be prompted to create a UNIX username and password.

Notes:
- On Windows 11, `wsl --install` installs WSL2 and a recent Ubuntu LTS by default. This is the fastest and most reliable route for most users.

Manual install steps (when you need control)

If you need to pick a specific Ubuntu version or perform an explicit manual setup, follow these steps:

1. Open PowerShell as Administrator and enable required Windows features:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

2. Restart the machine.
3. (If required) Install the WSL2 Linux kernel update package from Microsoft. Download from Microsoft's WSL docs page.
4. Install a specific Ubuntu distro from the Microsoft Store (for example `Ubuntu 22.04 LTS`) or install via the command:

```powershell
wsl --install -d Ubuntu-22.04
```

5. Ensure your default WSL version is set to 2:

```powershell
wsl --set-default-version 2
```

First-time Ubuntu setup

On first launch of the Ubuntu distro:

- Create the UNIX user and password when prompted. This account will be your default login for the distro.
- Update package indexes and upgrade installed packages:

```bash
sudo apt update && sudo apt upgrade -y
```

- Install common packages you expect to use in the shell (examples):

```bash
sudo apt install -y build-essential curl git ca-certificates lsb-release
```

Managing WSL distributions and versions

- List installed distros and WSL versions:

```powershell
wsl -l -v
```

- Set the default distro for `wsl` (replace `<Name>` with the distro name):

```powershell
wsl --setdefault <Name>
```

- Convert an existing distro to WSL2:

```powershell
wsl --set-version <Name> 2
```

Enabling and using systemd

Recent WSL releases support systemd which improves service management and compatibility with many Linux expectations.

1. Inside the Ubuntu distro, create or edit `/etc/wsl.conf` and add:

```ini
[boot]
systemd=true
```

2. From Windows PowerShell run:

```powershell
wsl --shutdown
```

3. Start the distro again (open Ubuntu) and verify systemd is active:

```bash
systemctl status --no-pager
```

Notes:
- Enabling systemd changes how background services are managed. If a particular service needs to run automatically, use systemd service units.
- If your Windows/WSL build does not support systemd, you can still run services manually (for example via `nohup` or supervised scripts).

Filesystem and performance best practices

- Prefer keeping development repositories and frequent I/O files inside the WSL filesystem (your distro home: `~/`) for best performance.
- Accessing Windows files from WSL is possible via `/mnt/c/Users/...` but is slower for heavy file operations.
- Use `\wsl$\` paths from Windows to access files stored inside the WSL filesystem when needed.

Locale, time, and environment

- The WSL distro inherits some settings from Windows but you may want to configure locale/timezone inside the distro:

```bash
sudo timedatectl set-timezone "America/Chicago"
sudo update-locale LANG=en_US.UTF-8
```

Networking, localhost, and hosts file

- WSL2 supports localhost forwarding: services bound to `0.0.0.0` or `127.0.0.1` inside WSL are reachable from Windows via `localhost` when `localhostForwarding` is enabled.
- To add local host mappings (custom domains) edit the Windows hosts file as Administrator: `C:\Windows\System32\drivers\etc\hosts` and add lines such as:

```
127.0.0.1 my-local.test
```

- If you experience networking issues (e.g., after system sleep), a full WSL shutdown and restart often resolves them:

```powershell
wsl --shutdown
```

Resource tuning with `.wslconfig`

Create or update `C:\Users\<you>\.wslconfig` to control WSL2 resource limits. Example file:

```
[wsl2]
memory=6GB # Limits VM memory
processors=4 # Number of logical processors
swap=2GB
localhostForwarding=true
```

After changing `.wslconfig` run:

```powershell
wsl --shutdown
```

Exporting, importing, and backups

- Export a distro for an offline backup:

```powershell
wsl --export <DistroName> C:\path\to\backup\ubuntu.tar
```

- Import the archive as a new distro (choose a folder for the installation files):

```powershell
wsl --import <NewName> <InstallLocation> C:\path\to\backup\ubuntu.tar --version 2
```

- Unregister/remove a distro (this deletes all data — export first if you need it):

```powershell
wsl --unregister <DistroName>
```

Common problems and troubleshooting

- "Command not found" for `wsl` or `wsl --install`: ensure Windows is up-to-date and you're running PowerShell as Administrator.
- Slow file I/O: keep active projects inside the WSL filesystem (`~/`) rather than on mounted Windows drives.
- Broken services after Windows sleep/hibernation: run `wsl --shutdown` and reopen the distro.
- Permission and owner issues with files on Windows mounts: perform Git and development workflows inside WSL to avoid permission mismatches.

First-run verification checklist

- `wsl -l -v` — confirm distro names and that the distro runs on WSL version 2.
- `wsl --status` — show global WSL configuration.
- `cat /etc/os-release` inside the distro — confirm Ubuntu version.
- `systemctl status` (if systemd enabled) — check service manager status.
- `wsl --export <DistroName> backup.tar` — test the export command (you can delete the file afterward).

If you'd like, I can also provide:
- A minimal `.wslconfig` tuned for low-memory laptops.
- A short checklist file with copy/paste commands you can run after installing WSL.
