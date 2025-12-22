### The common setup (Docker Engine inside WSL2)

1. **Enable `systemd` in your WSL distro** (makes running `dockerd` as a service straightforward). Microsoft documents WSL’s `systemd` support and the `wsl.conf` flag. ([Microsoft Learn][1])

Create/edit `/etc/wsl.conf` in WSL:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Then restart WSL from **Windows**:

```powershell
wsl --shutdown
```

2. **Install Docker Engine in WSL** (follow Docker’s official Ubuntu install steps). ([Docker Documentation][2])
   For Ubuntu, this is the standard “install from apt repo” flow (from Docker docs): ([Docker Documentation][2])

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
| sudo tee /etc/apt/sources.list.d/docker.list >/dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Start Docker:

```bash
sudo systemctl enable --now docker
```

Test:

```bash
docker run --rm hello-world
```

3. **Optional: run docker without `sudo`**
   Docker’s post-install steps show adding your user to the `docker` group (with an important security warning), and also point to rootless mode as an alternative. ([Docker Documentation][3])

```bash
sudo usermod -aG docker $USER
```

Then restart your WSL shell.

### Practical notes

* This gives you **Linux containers** in WSL2. Running **Windows containers** generally requires the Windows Docker engine (typically via Docker Desktop).
* Easiest workflow is to run `docker` **inside WSL**. If you want to invoke it from Windows, a simple approach is:

  ```powershell
  wsl -d Ubuntu -- docker ps
  ```
[2]: https://docs.docker.com/engine/install/ubuntu/?utm_source=chatgpt.com "Install Docker Engine on Ubuntu"
[3]: https://docs.docker.com/engine/install/linux-postinstall/?utm_source=chatgpt.com "Linux post-installation steps for Docker Engine"
