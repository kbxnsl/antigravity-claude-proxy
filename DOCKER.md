# Docker Deployment Guide

This guide explains how to deploy the Antigravity Claude Proxy using Docker and Portainer.

## Quick Start

### Using Docker Compose

```bash
# Set environment variables
export ACCOUNT_1_EMAIL=your-email@gmail.com
export ACCOUNT_1_REFRESH_TOKEN=your-refresh-token
export ACCOUNT_1_PROJECT_ID=your-project-id

# Start the container
docker-compose up -d

# Check logs
docker-compose logs -f

# Stop the container
docker-compose down
```

## Portainer Stack Deployment

### Step 1: Add New Stack

1. Log into your Portainer instance
2. Navigate to **Stacks** → **Add stack**
3. Enter a name (e.g., `antigravity-claude-proxy`)

### Step 2: Configure Repository

1. Select **Repository** as the build method
2. Enter your git repository URL:
   ```
   https://github.com/YOUR-USERNAME/antigravity-claude-proxy.git
   ```
3. Set **Compose path**: `docker-compose.yml`
4. (Optional) Specify branch if not using `main`

### Step 3: Set Environment Variables

In the **Environment variables** section, add your account credentials:

```bash
# Required: Account 1
ACCOUNT_1_EMAIL=your-email@gmail.com
ACCOUNT_1_REFRESH_TOKEN=1//your-refresh-token-here
ACCOUNT_1_PROJECT_ID=your-project-id
ACCOUNT_1_SOURCE=oauth

# Optional: Account 2
# ACCOUNT_2_EMAIL=another-email@gmail.com
# ACCOUNT_2_REFRESH_TOKEN=1//another-refresh-token
# ACCOUNT_2_PROJECT_ID=another-project-id
# ACCOUNT_2_SOURCE=oauth

# Optional: Settings (defaults shown)
COOLDOWN_DURATION_MS=60000
MAX_RETRIES=5
```

### Step 4: Deploy

Click **Deploy the stack** and wait for the container to start.

### Step 5: Verify

1. Check container logs in Portainer
2. Verify the service is running:
   ```bash
   curl http://YOUR-SERVER-IP:31080/health
   ```

## Port Configuration

The application listens on **port 31080** by default.

To change the port mapping in Portainer:
- Edit the stack
- Modify the ports section in docker-compose.yml:
  ```yaml
  ports:
    - "8080:31080"  # Maps host port 8080 to container port 31080
  ```

## Getting Account Credentials

### Option 1: Using OAuth (Recommended)

Run the accounts CLI locally to authenticate:

```bash
npm run accounts:add
```

This will guide you through OAuth authentication and display your:
- Email
- Refresh token
- Project ID

### Option 2: From Existing .env

If you already have a local `.env` file:

```bash
cat .env | grep ACCOUNT_1
```

## Environment Variables Reference

| Variable | Required | Description | Default |
|----------|----------|-------------|---------|
| `ACCOUNT_N_EMAIL` | Yes | Google account email | - |
| `ACCOUNT_N_REFRESH_TOKEN` | Yes | OAuth refresh token | - |
| `ACCOUNT_N_PROJECT_ID` | Yes | Google Cloud project ID | - |
| `ACCOUNT_N_SOURCE` | No | Account source type | `oauth` |
| `COOLDOWN_DURATION_MS` | No | Rate limit cooldown (ms) | `60000` |
| `MAX_RETRIES` | No | Max retry attempts | `5` |

Replace `N` with account number (1, 2, 3, etc.).

## Troubleshooting

### Container won't start

Check the logs in Portainer or via:
```bash
docker logs antigravity-claude-proxy
```

Common issues:
- Missing required environment variables
- Invalid refresh token (needs re-authentication)
- Port 31080 already in use

### Health check failing

Verify the container can reach the health endpoint:
```bash
docker exec antigravity-claude-proxy curl http://localhost:31080/health
```

### Account authentication errors

If you see authentication errors in logs:
1. Re-authenticate locally: `npm run accounts:add`
2. Copy the new refresh token to Portainer environment variables
3. Restart the stack

## Building Locally

### Build Image

```bash
docker build -t antigravity-claude-proxy .
```

### Run Container

```bash
docker run -d \
  -p 31080:31080 \
  -e ACCOUNT_1_EMAIL=your-email@gmail.com \
  -e ACCOUNT_1_REFRESH_TOKEN=your-token \
  -e ACCOUNT_1_PROJECT_ID=your-project \
  antigravity-claude-proxy
```

## Health Monitoring

The container includes a health check that:
- Runs every 30 seconds
- Checks the `/health` endpoint
- Marks container unhealthy after 3 failed attempts

View health status:
```bash
docker inspect antigravity-claude-proxy | grep Health
```

## Security Notes

- The container runs as a non-root user (`nodejs:1001`)
- Environment variables containing tokens are never committed to git
- `.env` file is in `.gitignore` and `.dockerignore`
- Use Portainer's built-in secrets management for production deployments

## Updating the Stack

To update to the latest version:

1. In Portainer, go to your stack
2. Click **Pull and redeploy**
3. Portainer will pull the latest code from git and rebuild

Or manually:
```bash
git pull
docker-compose up -d --build
```
