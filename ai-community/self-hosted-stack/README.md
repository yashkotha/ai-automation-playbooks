# Self-Hosted AI Stack

Full setup guide for running your own AI automation infrastructure. One VPS handles n8n, Postgres, Redis, vector search, local LLMs, S3-compatible storage, and static hosting — all behind Caddy with automatic HTTPS.

## Stack

| Layer | Tool |
|-------|------|
| Compute | VPS (Ubuntu 22.04) |
| Automation | n8n |
| Database | Postgres 16 |
| Cache / Queue | Redis 7 |
| Vector DB | Qdrant |
| Local LLMs | Ollama |
| Object Storage | MinIO |
| Reverse Proxy | Caddy |
| Docker UI | Portainer |
| Monitoring | Uptime Kuma |
| AI APIs | OpenAI / Claude / Resend |

---

## 1. Get a VPS

### Minimum specs (n8n only)
- 2 vCPU, 4 GB RAM
- Ubuntu 22.04 LTS
- 40 GB SSD

### Recommended specs (full stack with Ollama)
- 4 vCPU, 8–16 GB RAM
- Ubuntu 22.04 LTS
- 80–160 GB SSD
- GPU optional but not required for small models (Mistral 7B, Phi-3)

### Providers and approximate monthly cost

| Provider | Spec | Cost/mo |
|----------|------|---------|
| Hetzner CX22 | 2 vCPU, 4 GB | $4 |
| Hetzner CX32 | 4 vCPU, 8 GB | $9 |
| Hetzner CCX23 | 4 vCPU dedicated, 16 GB | $34 |
| Hostinger KVM 2 | 2 vCPU, 8 GB | $8 |
| DigitalOcean General 2x4 | 2 vCPU, 4 GB | $24 |
| Vultr High Freq 4GB | 2 vCPU, 4 GB | $24 |

Hetzner is consistently the cheapest and most reliable for EU and US. Hostinger is good value globally. DigitalOcean has the nicest UX but costs 3–5x more for the same specs.

---

## 2. Initial Server Setup

### Connect and create a non-root user

```bash
ssh root@YOUR_VPS_IP

adduser deploy
usermod -aG sudo deploy
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy
```

From now on, SSH as `deploy`, not `root`.

### SSH key-only authentication

On your local machine:
```bash
ssh-keygen -t ed25519 -C "vps-deploy"
ssh-copy-id deploy@YOUR_VPS_IP
```

Disable password auth on the VPS:
```bash
sudo nano /etc/ssh/sshd_config
```

Set these values:
```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

Restart SSH: `sudo systemctl restart sshd`

---

## 3. Security Hardening

### Firewall with ufw

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status
```

Only ports 22, 80, and 443 should be open. All internal services (n8n, Postgres, Redis, etc.) run inside Docker and are NOT exposed to the public internet. Caddy reverse-proxies everything.

### fail2ban (blocks brute force SSH attempts)

```bash
sudo apt install -y fail2ban

sudo tee /etc/fail2ban/jail.local <<EOF
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
EOF

sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Check bans: `sudo fail2ban-client status sshd`

### Non-root Docker

Docker is already installed as your `deploy` user in step 4. The `usermod -aG docker deploy` command means Docker commands run without `sudo`. Containers run as non-root users internally — n8n runs as `node`, Postgres as `postgres`, etc. Never run `docker run` as root.

---

## 4. Install Docker and Docker Compose

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
docker --version
```

Docker Compose v2 is included. Verify: `docker compose version`

---

## 5. Directory and Secrets Setup

### Create the project directory

```bash
mkdir -p ~/n8n-stack
cd ~/n8n-stack
```

### .env file for secrets

All credentials live in one `.env` file. Never commit this to git.

```bash
nano ~/n8n-stack/.env
```

```dotenv
# --- Postgres ---
POSTGRES_USER=n8n
POSTGRES_PASSWORD=CHANGE_ME_STRONG_PASSWORD
POSTGRES_DB=n8n

# --- n8n ---
N8N_HOST=n8n.yourdomain.com
N8N_PROTOCOL=https
WEBHOOK_URL=https://n8n.yourdomain.com
N8N_ENCRYPTION_KEY=CHANGE_ME_32_CHAR_RANDOM_STRING
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=CHANGE_ME_STRONG_PASSWORD

# --- Redis ---
REDIS_PASSWORD=CHANGE_ME_REDIS_PASSWORD

# --- MinIO ---
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=CHANGE_ME_MINIO_PASSWORD

# --- Domains (used by Caddy) ---
DOMAIN=yourdomain.com

# --- Optional: S3 backup target ---
BACKUP_S3_BUCKET=your-backup-bucket
BACKUP_S3_REGION=us-east-1
BACKUP_AWS_ACCESS_KEY=your-key
BACKUP_AWS_SECRET_KEY=your-secret
```

Generate strong passwords: `openssl rand -base64 32`

Add `.env` to `.gitignore` immediately:
```bash
echo ".env" >> .gitignore
echo "backups/" >> .gitignore
```

---

## 6. Docker Compose — Full Stack

Save as `~/n8n-stack/docker-compose.yml`:

```yaml
version: "3.8"

networks:
  n8n_network:
    driver: bridge

volumes:
  postgres_data:
  n8n_data:
  redis_data:
  qdrant_data:
  ollama_data:
  minio_data:
  portainer_data:
  uptime_kuma_data:

services:

  # --- Postgres ---
  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - n8n_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # --- Redis ---
  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - n8n_network
    healthcheck:
      test: ["CMD", "redis-cli", "--no-auth-warning", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # --- n8n ---
  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      - N8N_HOST=${N8N_HOST}
      - N8N_PROTOCOL=${N8N_PROTOCOL}
      - WEBHOOK_URL=${WEBHOOK_URL}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - QUEUE_BULL_REDIS_HOST=redis
      - QUEUE_BULL_REDIS_PORT=6379
      - QUEUE_BULL_REDIS_PASSWORD=${REDIS_PASSWORD}
      - EXECUTIONS_MODE=queue
      - N8N_LOG_LEVEL=info
      - GENERIC_TIMEZONE=America/New_York
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - n8n_network

  # --- n8n worker (scales queue execution) ---
  n8n_worker:
    image: n8nio/n8n:latest
    restart: unless-stopped
    command: worker
    depends_on:
      - n8n
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - QUEUE_BULL_REDIS_HOST=redis
      - QUEUE_BULL_REDIS_PORT=6379
      - QUEUE_BULL_REDIS_PASSWORD=${REDIS_PASSWORD}
      - EXECUTIONS_MODE=queue
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - n8n_network

  # --- Qdrant (vector DB) ---
  qdrant:
    image: qdrant/qdrant:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage
    networks:
      - n8n_network

  # --- Ollama (local LLMs) ---
  ollama:
    image: ollama/ollama:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    networks:
      - n8n_network
    # Uncomment if you have an NVIDIA GPU:
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: 1
    #           capabilities: [gpu]

  # --- MinIO (S3-compatible object storage) ---
  minio:
    image: minio/minio:latest
    restart: unless-stopped
    command: server /data --console-address ":9001"
    ports:
      - "127.0.0.1:9000:9000"
      - "127.0.0.1:9001:9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_ROOT_USER}
      - MINIO_ROOT_PASSWORD=${MINIO_ROOT_PASSWORD}
    volumes:
      - minio_data:/data
    networks:
      - n8n_network

  # --- Portainer (Docker management UI) ---
  portainer:
    image: portainer/portainer-ce:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    networks:
      - n8n_network

  # --- Uptime Kuma (self-hosted monitoring) ---
  uptime_kuma:
    image: louislam/uptime-kuma:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:3001:3001"
    volumes:
      - uptime_kuma_data:/app/data
    networks:
      - n8n_network
```

### Start everything

```bash
cd ~/n8n-stack
docker compose up -d
docker compose ps
```

All services will be `healthy` or `running` within 30–60 seconds.

---

## 7. Caddy Reverse Proxy with HTTPS

Caddy auto-provisions Let's Encrypt certs. No certbot, no manual renewals.

### Install Caddy

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' \
  | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' \
  | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update && sudo apt install caddy
```

### Caddyfile — all services

`/etc/caddy/Caddyfile`:

```caddyfile
# n8n automation
n8n.yourdomain.com {
    reverse_proxy localhost:5678
    encode gzip
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
        X-Content-Type-Options nosniff
        X-Frame-Options SAMEORIGIN
        Referrer-Policy strict-origin-when-cross-origin
    }
}

# Qdrant REST API
qdrant.yourdomain.com {
    reverse_proxy localhost:6333
    basicauth {
        # Generate with: caddy hash-password
        admin $2a$14$HASH_HERE
    }
}

# Ollama API
ollama.yourdomain.com {
    reverse_proxy localhost:11434
    basicauth {
        admin $2a$14$HASH_HERE
    }
}

# MinIO console
minio.yourdomain.com {
    reverse_proxy localhost:9001
}

# MinIO S3 API
s3.yourdomain.com {
    reverse_proxy localhost:9000
}

# Portainer
portainer.yourdomain.com {
    reverse_proxy https://localhost:9443 {
        transport http {
            tls_insecure_skip_verify
        }
    }
}

# Uptime Kuma
status.yourdomain.com {
    reverse_proxy localhost:3001
}

# Custom API or backend
api.yourdomain.com {
    reverse_proxy localhost:3000
}
```

### DNS setup

Create A records for every subdomain, all pointing to the same VPS IP:

```
n8n.yourdomain.com     A  YOUR_VPS_IP
qdrant.yourdomain.com  A  YOUR_VPS_IP
ollama.yourdomain.com  A  YOUR_VPS_IP
minio.yourdomain.com   A  YOUR_VPS_IP
s3.yourdomain.com      A  YOUR_VPS_IP
portainer.yourdomain.com A YOUR_VPS_IP
status.yourdomain.com  A  YOUR_VPS_IP
api.yourdomain.com     A  YOUR_VPS_IP
```

Or use a wildcard: `*.yourdomain.com A YOUR_VPS_IP`

### Apply and reload

```bash
sudo caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
```

Check cert status: `sudo caddy list-modules | head` and `sudo journalctl -u caddy -f`

---

## 8. Connecting Services in n8n

### Postgres (internal, no credentials node needed)
Already wired via environment variables. n8n writes to its own database automatically.

### Redis
Used internally for queue mode. No n8n credential needed.

### Qdrant (for AI vector search workflows)
In n8n: use HTTP Request node.
- URL: `http://qdrant:6333` (internal Docker network)
- No auth needed internally

### Ollama (for local LLM calls)
In n8n: use HTTP Request node.
- URL: `http://ollama:11434/api/generate`
- Method: POST
- Body: `{ "model": "mistral", "prompt": "{{$json.input}}" }`

Pull a model first:
```bash
docker exec -it n8n-stack-ollama-1 ollama pull mistral
docker exec -it n8n-stack-ollama-1 ollama pull nomic-embed-text
```

### OpenAI
- Settings -> Credentials -> New -> OpenAI API
- API Key: `sk-...`

### Anthropic / Claude
- Credential type: Header Auth
- Name: `x-api-key`
- Value: your API key

### Resend (email)
- Credential type: Header Auth
- Name: `Authorization`
- Value: `Bearer re_...`

### MinIO (S3-compatible)
In n8n: AWS S3 credential
- Access Key: your `MINIO_ROOT_USER`
- Secret Key: your `MINIO_ROOT_PASSWORD`
- Region: `us-east-1` (can be any string for MinIO)
- Endpoint: `https://s3.yourdomain.com`
- Force path style: `true`

---

## 9. Automated Backups

### Daily backup script

```bash
nano ~/n8n-stack/backup.sh
```

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/home/deploy/backups"
DATE=$(date +%Y-%m-%d_%H-%M-%S)
COMPOSE_DIR="/home/deploy/n8n-stack"

mkdir -p "$BACKUP_DIR"

# Load env
source "$COMPOSE_DIR/.env"

echo "[$DATE] Starting backup..."

# 1. Dump Postgres
docker exec n8n-stack-postgres-1 pg_dump \
  -U "$POSTGRES_USER" "$POSTGRES_DB" \
  | gzip > "$BACKUP_DIR/postgres_$DATE.sql.gz"

# 2. Tar n8n data volume
docker run --rm \
  -v n8n-stack_n8n_data:/source:ro \
  -v "$BACKUP_DIR":/backup \
  alpine tar czf "/backup/n8n_data_$DATE.tar.gz" -C /source .

# 3. Remove backups older than 14 days
find "$BACKUP_DIR" -name "*.gz" -mtime +14 -delete

echo "[$DATE] Backup complete."

# 4. Optional: sync to S3 (requires aws CLI)
# aws s3 sync "$BACKUP_DIR" "s3://$BACKUP_S3_BUCKET/n8n-backups/" \
#   --region "$BACKUP_S3_REGION"

echo "[$DATE] Done."
```

Make executable and schedule:
```bash
chmod +x ~/n8n-stack/backup.sh

# Run at 2 AM daily
crontab -e
```

Add this line:
```
0 2 * * * /home/deploy/n8n-stack/backup.sh >> /home/deploy/backups/backup.log 2>&1
```

### Restore from backup

Postgres:
```bash
gunzip -c /home/deploy/backups/postgres_2025-01-01_02-00-00.sql.gz \
  | docker exec -i n8n-stack-postgres-1 psql -U n8n n8n
```

n8n data volume:
```bash
docker compose stop n8n n8n_worker
docker run --rm \
  -v n8n-stack_n8n_data:/target \
  -v /home/deploy/backups:/backup \
  alpine tar xzf /backup/n8n_data_2025-01-01_02-00-00.tar.gz -C /target
docker compose start n8n n8n_worker
```

---

## 10. Updating n8n Safely

Always backup before updating. The process: backup, pull, restart.

```bash
cd ~/n8n-stack

# 1. Backup first
./backup.sh

# 2. Pull latest image
docker compose pull n8n n8n_worker

# 3. Recreate containers (zero-downtime restart with queue mode)
docker compose up -d --no-deps n8n n8n_worker

# 4. Verify
docker compose ps
docker logs n8n-stack-n8n-1 --tail 30
```

Check the n8n changelog before major version upgrades: `https://github.com/n8n-io/n8n/releases`

Some major versions require database migrations. n8n runs them automatically on startup, but having a fresh backup makes rollback trivial.

---

## 11. Monitoring with Uptime Kuma

Uptime Kuma runs at `https://status.yourdomain.com`. After first login, add monitors:

| Monitor | Type | URL | Interval |
|---------|------|-----|----------|
| n8n | HTTP(s) | `https://n8n.yourdomain.com` | 1 min |
| n8n webhook | HTTP(s) | `https://n8n.yourdomain.com/healthz` | 1 min |
| MinIO | HTTP(s) | `https://minio.yourdomain.com` | 5 min |
| Portainer | HTTP(s) | `https://portainer.yourdomain.com` | 5 min |
| Postgres | Docker Container | container name | 1 min |

### Notification channels (in Uptime Kuma)
- Email (SMTP or Resend webhook)
- Telegram
- Slack
- Discord
- Ntfy (self-hosted push notifications)

### Log aggregation (lightweight approach)

Stream all Docker logs to a file for later inspection:
```bash
# Follow all services' logs
docker compose logs -f --tail=0 > /var/log/n8n-stack.log 2>&1 &
```

Or use `loki` + `promtail` + `Grafana` for full log search (adds ~500 MB RAM overhead).

Live logs for a specific service:
```bash
docker logs n8n-stack-n8n-1 --tail 100 -f
docker logs n8n-stack-postgres-1 --tail 50 -f
```

---

## 12. VPS Management with Claude Code (SSH via Bash)

You can manage the VPS directly from Claude Code using its Bash tool. This works for one-off commands, debugging, and deployment automation.

### Example prompts

**Deploy config changes:**
```
Using the Bash tool, SSH into deploy@31.97.114.203 and:
1. cd ~/n8n-stack
2. Pull the latest docker-compose.yml from this conversation (paste content)
3. docker compose up -d
4. Show me docker compose ps output
```

**Debug a failing workflow:**
```
SSH into deploy@31.97.114.203 and run:
docker logs n8n-stack-n8n-1 --tail 200 | grep -i error
Then check docker compose ps for any unhealthy containers.
Report what you find.
```

**Rolling update with verification:**
```
SSH into the VPS and:
1. Run ~/n8n-stack/backup.sh
2. docker compose pull n8n
3. docker compose up -d --no-deps n8n
4. Wait 15 seconds
5. curl -sf https://n8n.yourdomain.com/healthz && echo OK
6. Report success or failure
```

**Disk usage check:**
```
SSH in and run:
df -h
docker system df
du -sh ~/backups/*
```

### SSH config shortcut (on your local machine)

`~/.ssh/config`:
```
Host vps
    HostName 31.97.114.203
    User deploy
    IdentityFile ~/.ssh/vps-deploy
    ServerAliveInterval 60
```

Now `ssh vps` works from terminal. Claude Code can use `ssh vps` in Bash commands too.

---

## 13. Troubleshooting

### n8n won't start

```bash
docker compose logs n8n --tail 50
```

Common causes:
- Postgres not healthy yet — wait 30 seconds and retry
- Wrong `DB_POSTGRESDB_PASSWORD` — check `.env` matches Postgres container
- Port 5678 already in use — `sudo lsof -i :5678`

### Caddy not issuing certs

```bash
sudo journalctl -u caddy -f
```

Common causes:
- DNS hasn't propagated yet — wait up to 1 hour after adding A records
- Port 80 blocked by ufw — `sudo ufw allow 80`
- Caddyfile syntax error — run `sudo caddy validate --config /etc/caddy/Caddyfile`

### n8n workflows stuck in queue

Redis is the queue backend. If Redis is unhealthy, executions pile up.

```bash
docker compose restart redis
docker compose restart n8n_worker
```

Check the queue depth:
```bash
docker exec n8n-stack-redis-1 redis-cli -a YOUR_REDIS_PASSWORD llen bull:jobs:wait
```

### Postgres disk full

```bash
# Check Postgres volume usage
docker system df -v | grep postgres

# Vacuum to reclaim space
docker exec -it n8n-stack-postgres-1 psql -U n8n -c "VACUUM FULL;"
```

### MinIO bucket access denied

```bash
# Check MinIO logs
docker logs n8n-stack-minio-1 --tail 50

# Reset credentials (restart container picks up new .env values)
docker compose up -d --no-deps minio
```

### Ollama model download stuck

```bash
# Check download progress
docker exec -it n8n-stack-ollama-1 ollama list

# Kill and restart a stuck pull
docker compose restart ollama
docker exec -it n8n-stack-ollama-1 ollama pull mistral
```

### Container keeps restarting

```bash
docker inspect n8n-stack-n8n-1 --format='{{.RestartCount}} restarts, ExitCode: {{.State.ExitCode}}'
docker logs n8n-stack-n8n-1 --tail 20
```

### Out of disk space on VPS

```bash
# Find what's using space
du -sh /var/* | sort -rh | head -10

# Remove unused Docker images and volumes
docker system prune -af --volumes

# Clear old logs
sudo journalctl --vacuum-time=7d
```

---

## 14. Monthly Cost Breakdown

### Minimal (n8n only)

| Item | Cost/mo |
|------|---------|
| Hetzner CX22 (2 vCPU, 4 GB) | $4 |
| Domain (.com) | $1 |
| Resend (3k emails free) | $0 |
| OpenAI API (light use) | $5–20 |
| **Total** | **$10–25** |

### Standard (full stack, no Ollama)

| Item | Cost/mo |
|------|---------|
| Hetzner CX32 (4 vCPU, 8 GB) | $9 |
| Domain (.com) | $1 |
| Resend (50k emails) | $20 |
| OpenAI API (moderate use) | $20–50 |
| Claude API (moderate use) | $10–30 |
| S3-compatible backup (MinIO on same VPS) | $0 |
| **Total** | **$60–110** |

### Full stack with local LLMs (Ollama)

| Item | Cost/mo |
|------|---------|
| Hetzner CCX23 (4 vCPU dedicated, 16 GB) | $34 |
| Domain (.com) | $1 |
| Resend (50k emails) | $20 |
| OpenAI API (reduced, Ollama handles smaller tasks) | $5–15 |
| Claude API (complex reasoning only) | $5–20 |
| **Total** | **$65–90** |

### High volume (agency or productized service)

| Item | Cost/mo |
|------|---------|
| Hetzner AX41 (6 vCPU dedicated, 32 GB) | $54 |
| Domain | $1 |
| Resend (500k emails) | $90 |
| OpenAI / Claude APIs | $100–500 |
| Backblaze B2 (100 GB backup) | $1 |
| **Total** | **$246–646** |

Compare: n8n cloud starts at $20/mo and caps at 2,500 active workflows. Self-hosted is unlimited.
