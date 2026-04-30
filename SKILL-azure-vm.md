# Azure VM Skill

## When to Use
- Deploying or updating the VocabVista backend API
- Configuring nginx on the Azure VM
- Managing the PostgreSQL database
- Troubleshooting domain/DNS issues (api.kreativeland.com, etc.)
- SSH access to the VM

## VM Details

### Production VM (Current)
- **Public IP:** 20.255.60.68
- **OS:** Ubuntu 24.04
- **SSH user:** `azureuser`
- **SSH password:** `?3198yXKb84f`
- **Note:** No SSH keys on current laptop (GalaxyBook2)

### Old VM (Reference, still running)
- **Public IP:** 20.205.61.171
- **Tailscale IP:** 100.87.60.60
- **Note:** Use Tailscale IP for SSH from this laptop (GFW may block public IP)

## VocabVista Backend

### Service
- **Path:** `/opt/vocabvista/backend/`
- **Systemd service:** `vocabvista.service`
- **Process:** FastAPI on `127.0.0.1:8002`

### Deploy
The backend at `/opt/vocabvista/backend/` is **NOT a git repo** -- it was set up via tarball copy during migration. Deploy by uploading files via SFTP/SCP or paramiko.

```bash
# Copy new code (from local machine via scp)
scp -r D:/Projects/VocabVista/backend/* azureuser@20.255.60.68:/opt/vocabvista/backend/

# Restart service
sudo systemctl restart vocabvista.service

# Check status
sudo systemctl status vocabvista.service
sudo journalctl -u vocabvista.service -n 50
```

### SSH via Paramiko (Windows)
Since `sshpass` is not available on Windows and no SSH keys exist on the current laptop, use paramiko for programmatic SSH:

```python
import paramiko
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('20.255.60.68', username='azureuser', password='?3198yXKb84f', timeout=15)

# Run commands
stdin, stdout, stderr = ssh.exec_command('sudo systemctl restart vocabvista.service')
print(stdout.read().decode())

# Upload files via SFTP
sftp = ssh.open_sftp()
sftp.put('D:/Projects/VocabVista/backend/main.py', '/opt/vocabvista/backend/main.py')
sftp.close()
ssh.close()
```

### Database
- **PostgreSQL 16** at `localhost:5432`
- **Database:** `vocabvista`
- **User:** `vocabvista`
- **Contents:** ~15,011 words, 4 users, 6,786 vocab entries (as of 2026-04-29)

### API Endpoints
- **Base URL:** https://api.kreativeland.com
- **Health check:** https://api.kreativeland.com/api/health
- **Expected response:** `{"status":"ok","version":"2.0.0"}`

## Nginx Configuration

### Config Files
- **api.kreativeland.com:** `/etc/nginx/sites-available/api.kreativeland.com`
- **Enabled configs:** `/etc/nginx/sites-enabled/` (symlinks from sites-available)

### api.kreativeland.com Config
```nginx
# HTTP -> HTTPS redirect (port 80)
server {
    listen 80;
    server_name api.kreativeland.com;
    # Returns 301 to https:// for api.kreativeland.com
    # Returns 404 for other hosts
}

# HTTPS reverse proxy (port 443)
server {
    listen 443 ssl;
    server_name api.kreativeland.com;
    # SSL certs via Let's Encrypt
    # Proxy to 127.0.0.1:8002
}
```

### Common Nginx Commands
```bash
# Test config
sudo nginx -t

# Restart
sudo systemctl restart nginx

# Check status
sudo systemctl status nginx

# Reload (no downtime)
sudo systemctl reload nginx

# List enabled sites
ls -la /etc/nginx/sites-enabled/
```

### Troubleshooting Domain Redirects
If domains are redirecting to the wrong place (e.g., AI Playground):
1. Check `/etc/nginx/sites-enabled/` for unexpected symlinks
2. Check for `default_server` directives in configs
3. Remove unwanted symlinks: `sudo rm /etc/nginx/sites-enabled/<name>`
4. Test and restart: `sudo nginx -t && sudo systemctl restart nginx`

## Let's Encrypt SSL
- Certs managed by certbot
- Renewal: `sudo certbot renew`
- Certs location: `/etc/letsencrypt/live/api.kreativeland.com/`

## Cloudflare DNS
Domains using Cloudflare:
- `api.kreativeland.com` -> A record -> 20.255.60.68
- Other domains may also point to this VM

## Known Issues
- **GFW:** SSH to public IP may fail from this laptop. Use Tailscale IP for old VM.
- **Catch-all configs:** Be careful when adding new nginx configs -- `default_server` will catch traffic for unknown domains
