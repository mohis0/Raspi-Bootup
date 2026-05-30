# Raspi-Bootup

# Raspberry Pi 5 Initial Setup

## 1. Update Package Lists

```bash
sudo apt update
```

Refreshes the package list from repositories.

---

## 2. Upgrade Installed Packages

```bash
sudo apt upgrade -y
```

Updates installed packages automatically without confirmation.
**Note:** This may Take some time ...

---

## 3. Install Basic Dependencies

```bash
sudo apt install -y curl git wget vim htop unzip build-essential
```

Installs essential tools for downloading files, version control, editing and monitoring.

---

## 4. Install code-server

```bash
curl -fsSL https://code-server.dev/install.sh | sh

sudo systemctl enable --now code-server@$USER
```

## 5.Configure code-server

```bash
nano ~/.config/code-server/config.yaml
```

Verify:

```yaml
bind-addr: 127.0.0.1:8080
```

Optional: change the password.

```yaml
password: your_password
```

Apply changes:

```bash
sudo systemctl restart code-server@$USER
```

Check status:

```bash
systemctl status code-server@$USER
```

### Remote Access

On the client machine:

```bash
ssh -L 8080:127.0.0.1:8080 pi@<raspi-ip>
```

Open in browser:

```text
http://127.0.0.1:8080
```

Access code-server through a secure SSH tunnel.

---

