# Docker deployment for forked repositories

This project can be deployed from your own fork without using the upstream GHCR image.

## Server setup

Install Docker on the cloud server:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin git
sudo systemctl enable --now docker
```

Clone your fork:

```bash
git clone <your-fork-url> gpt-image-playground
cd gpt-image-playground
```

Create the environment file:

```bash
cp .env.example .env
```

Edit `.env` if needed:

```env
APP_PORT=8080
DEFAULT_API_URL=http://39.102.124.3:18101
API_PROXY_URL=http://39.102.124.3:18101
ENABLE_API_PROXY=true
LOCK_API_PROXY=false
```

Build and start:

```bash
docker compose up -d --build
```

Open:

```text
http://<server-ip>:8080
```

## Update

After pushing changes to your fork:

```bash
git pull
docker compose up -d --build
```

## Notes

- `DEFAULT_API_URL` is the default API address shown in the UI.
- `API_PROXY_URL` is the target used by the container's Nginx `/api-proxy/` endpoint.
- Keep both values either with `/v1` or without `/v1` consistently with your relay. For your current relay, `http://39.102.124.3:18101` is the working value.
- `ENABLE_API_PROXY=true` enables same-origin proxying and supports streaming because Nginx buffering is disabled in `deploy/nginx.conf`.
- API keys are still entered in the browser UI. Do not put user API keys in `.env`.
