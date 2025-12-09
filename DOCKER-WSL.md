# Docker Desktop with WSL (Ubuntu) — Windows 11

This guide explains installing, configuring, and using Docker Desktop on Windows 11 with WSL (Ubuntu). It focuses only on Docker Desktop integration and usage from within a WSL Ubuntu distro — it does not cover installing WSL or Ubuntu itself.

Contents
- Install Docker Desktop (Windows)
- Enable WSL integration and configure Ubuntu
- Run `docker` and `docker compose` from WSL
- File sharing and recommended project locations
- Resource and performance settings
- Working with Docker contexts
- Compose best practices for WSL
- Common troubleshooting and fixes

Install Docker Desktop (Windows)

1. Download Docker Desktop for Windows from the official site: https://www.docker.com/get-started
2. Run the installer and follow the prompts. During installation select the option to use the WSL 2 based engine (this is enabled by default on Windows 11 installers).
3. Finish the installer and launch Docker Desktop. You may be prompted to sign in with a Docker ID — you can skip this step for local use.

Enable WSL integration and configure Ubuntu

1. Open Docker Desktop -> Settings (gear icon) -> General and confirm "Use the WSL 2 based engine" is checked.
2. Go to Settings -> Resources -> WSL Integration.
   - Toggle "Enable integration with my default WSL distro" (or individually enable the Ubuntu distro you use).
   - If you have multiple WSL distros, enable integration only for the ones you want Docker available in.
3. (Optional) Under Settings -> Resources -> File Sharing ensure paths you want accessible are shared. Note: sharing Windows drives is usually not necessary when you keep code inside the WSL filesystem.

Run `docker` and `docker compose` from WSL

1. Open your Ubuntu WSL shell.
2. Verify Docker is available and connected to Docker Desktop:

```bash
docker --version
docker compose version
docker info
```

3. If Docker commands return an error like "Cannot connect to the Docker daemon", open Docker Desktop and ensure it's running and WSL integration is enabled. Restart Docker Desktop if needed.

File sharing and recommended project locations

- For best performance, keep project files inside the WSL filesystem (e.g., `~/projects`) rather than on Windows-mounted drives (`/mnt/c/...`).
- From Windows, access WSL files via `\\wsl$\Ubuntu-22.04\home\<user>\projects` in File Explorer.
- Avoid editing code simultaneously from Windows editors pointing to Windows-mounted copies; prefer using VS Code Remote - WSL to edit files in-place inside WSL.

Resource and performance settings (Docker Desktop)

Docker Desktop exposes resource settings in Settings -> Resources. Key controls:
- CPUs: limit how many CPU cores Docker can use.
- Memory: cap the RAM available to the Docker VM.
- Swap: configure swap space.
- Disk: location and size for images/containers.

Recommendation: allocate enough memory and CPUs for your workloads, but leave headroom for Windows and other apps. Typical dev machines: 2–4 CPUs and 4–8 GB memory for Docker.

Working with Docker contexts

- Docker contexts allow switching between different Docker endpoints. Docker Desktop typically sets up a `desktop-windows`/`desktop-linux` context that connects Docker CLI inside WSL to Docker Desktop's daemon.

Check contexts:

```bash
docker context ls
```

Use a context:

```bash
docker context use <context-name>
```

Compose best practices for WSL

- Use `docker compose` (the plugin) instead of the legacy `docker-compose` binary when possible.
- Keep compose files and project sources inside WSL for best I/O performance.
- For multi-container development, use named volumes in Compose rather than mapping to host directories for runtime data that doesn't require host visibility.

Common troubleshooting and fixes

- Docker CLI shows "Cannot connect to the Docker daemon": Start Docker Desktop on Windows and confirm WSL integration for your Ubuntu distro.
- `docker compose` not found: ensure Docker Desktop is up-to-date. You can also install the Compose plugin via Docker's docs.
- Slow filesystem operations: move your project into the WSL filesystem instead of working on `/mnt/c/...`.
- Permission errors accessing files owned by root in WSL: fix ownership with `sudo chown -R $(id -u):$(id -g) <path>`.
- Networking issues (ports not reachable): check if Windows firewall or other processes are binding ports on Windows side; you may need to free those ports.

Advanced tips

- When using multiple WSL distros, enabling integration only for the active distro reduces accidental Docker usage from other distros.
- If you need consistent UID/GID between Windows and WSL users for mounted volumes, ensure user IDs match or adjust ownership after mounting.

References
- Docker Desktop docs: https://docs.docker.com/desktop/windows/wsl/
