# Docker Deployment

This fork is designed to deploy from a prebuilt GHCR image. GitHub Actions builds and publishes the image after each push to `main`, so the cloud server does not need to run `docker build`.

## Image

Default image:

```text
ghcr.io/cangjun123/gpt_image_playground:latest
```

GitHub Actions also publishes branch and commit tags such as:

```text
ghcr.io/cangjun123/gpt_image_playground:main
ghcr.io/cangjun123/gpt_image_playground:main-<commit>
```

## Server Setup

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
cp .env.example .env
```

Edit `.env`:

```env
APP_PORT=7070
APP_IMAGE=ghcr.io/cangjun123/gpt_image_playground:latest
DEFAULT_API_URL=http://39.102.124.3:18101
API_PROXY_URL=http://39.102.124.3:18101
ENABLE_API_PROXY=true
LOCK_API_PROXY=true
```

If the GHCR package is private, log in first:

```bash
docker login ghcr.io
```

Use your GitHub username and a Personal Access Token with `read:packages`. Public packages do not require login.

Start:

```bash
docker compose pull
docker compose up -d
```

Open:

```text
http://<server-ip>:7070
```

## Maintenance Flow

After local changes are ready:

```bash
git add .
git commit -m "your change"
git push origin main
```

Then open GitHub Actions and wait for `Build and Publish Docker Image` to finish successfully.

Update the server:

```bash
cd ~/gpt-image-playground
docker compose pull
docker compose up -d
docker compose logs -f
```

If `compose.yaml` or `.env.example` changed and you keep a full clone on the server, pull the repo too:

```bash
git fetch origin
git reset --hard origin/main
docker compose pull
docker compose up -d
```

Keep server-only settings in `.env`; do not edit source files on the deployment server.

## Rollback

Set `APP_IMAGE` in `.env` to a known working tag:

```env
APP_IMAGE=ghcr.io/cangjun123/gpt_image_playground:main-<commit>
```

Then run:

```bash
docker compose pull
docker compose up -d
```

## Local Build Fallback

If GHCR is unavailable or you intentionally want to build on the server:

```bash
docker compose -f compose.yaml -f compose.build.yaml up -d --build
```

This uses the local `deploy/Dockerfile` and tags the image as `gpt-image-playground:local`.

## Notes

- `DEFAULT_API_URL` is the default API address shown in the UI.
- `API_PROXY_URL` is the target used by the container's Nginx `/api-proxy/` endpoint.
- Keep both values either with `/v1` or without `/v1` consistently with your relay. For your current relay, `http://39.102.124.3:18101` is the working value.
- `ENABLE_API_PROXY=true` enables same-origin proxying and supports streaming because Nginx buffering is disabled in `deploy/nginx.conf`.
- API keys are still entered in the browser UI. Do not put user API keys in `.env`.
