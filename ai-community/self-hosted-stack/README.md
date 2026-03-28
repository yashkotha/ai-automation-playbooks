# Self-Hosted AI Stack

Full setup guide for running your own AI automation infrastructure. One VPS handles n8n, custom APIs, WebSocket servers, and static hosting.

## Stack

| Layer | Tool |
|-------|------|
| Compute | VPS (Ubuntu 22.04) |
| Automation | n8n (Docker) |
| Reverse proxy | Caddy |
| AI | OpenAI API / Claude API |
| Email | Resend |

---

## 1. Get a VPS

Minimum specs for running n8n comfortably:
- 2 vCPU, 4GB RAM
- Ubuntu 22.04 LTS
- 40GB SSD

Good providers: Hetzner (cheapest, EU and US), Hostinger (cheap globally), DigitalOcean (easiest UX).

---

## 2. Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

---

## 3. Run n8n

```bash
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e N8N_HOST=n8n.yourdomain.com \
  -e N8N_PROTOCOL=https \
  -e WEBHOOK_URL=https://n8n.yourdomain.com \
  --restart unless-stopped \
  n8nio/n8n
```

---

## 4. Reverse Proxy with Caddy

Caddy handles HTTPS automatically via Let's Encrypt. No certbot needed.

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update && sudo apt install caddy
```

Caddyfile (`/etc/caddy/Caddyfile`):
```
n8n.yourdomain.com {
    reverse_proxy localhost:5678
}

api.yourdomain.com {
    reverse_proxy localhost:3000
}
```

Reload: `sudo systemctl reload caddy`

---

## 5. Multi-Service on One VPS

One VPS can host multiple services on different subdomains:

```
yourdomain.com        -> Framer or static site (external)
n8n.yourdomain.com   -> n8n automation (port 5678)
api.yourdomain.com   -> custom backend (port 3000)
```

Point all subdomains to the same VPS IP as A records in DNS. Caddy routes them.

---

## 6. Connect AI Services in n8n

In n8n: Settings -> Credentials -> New

**OpenAI:**
- Credential type: Header Auth
- Name: `Authorization`
- Value: `Bearer sk-...`

**Anthropic / Claude:**
- Credential type: Header Auth
- Name: `x-api-key`
- Value: your API key

**Resend:**
- Credential type: Header Auth
- Name: `Authorization`
- Value: `Bearer re_...`

---

## 7. Monitoring

Use UptimeRobot (free tier) to ping your n8n webhook URL. You'll get notified if the VPS goes down.

Live logs: `docker logs n8n --tail 100 -f`

---

## Monthly Cost Breakdown

| Item | Cost |
|------|------|
| VPS (Hetzner CX22) | ~$4 |
| Domain | ~$1 |
| Resend (3k emails free) | $0 |
| OpenAI API (light use) | $5-20 |
| **Total** | **~$10-25** |
