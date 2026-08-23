# Service – Norish
**Norish** is a recipe app for planning meals, sharing groceries, and cooking together — recipe import from various sources (like reels etc.), meal planning, and shared grocery lists.

See the [Norish GitHub repository](https://github.com/norish-recipes/Norish) and the [Norish documentation](https://docs.norish.dev/) for more information about the application, its features, and configuration options.

## Setup (Through Dokploy)
Norish is deployed as a custom service in Dokploy.

**General:** \
Service Type: `Compose/DockerCompose` \
Application Name: `norish` \
Dokploy Path: `Norish/norish` \
Provider: `Raw (Docker Compose)` \
Domain: `recipe.example.com`

**Docker Compose:**
```yaml
services:
  norish:
    image: norishapp/norish:latest
    container_name: norish-app
    restart: unless-stopped
    user: "1000:1000"
    volumes:
      - norish_data:/app/uploads
    environment:
      AUTH_URL: ${APP_URL}
      DATABASE_URL: postgres://postgres:${POSTGRES_PASSWORD}@db:5432/norish
      MASTER_KEY: ${MASTER_KEY}
      CHROME_WS_ENDPOINT: ws://chrome-headless:3000
      REDIS_URL: redis://redis:6379
      UPLOADS_DIR: /app/uploads
      AI_ENABLED: "true"
      AI_PROVIDER: openai
      AI_MODEL: gpt-5-mini
      AI_API_KEY: ${OPENAI_API_KEY}
    depends_on:
      - db
      - redis

  db:
    image: postgres:17-alpine
    container_name: norish-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: norish
    volumes:
      - db_data:/var/lib/postgresql/data

  chrome-headless:
    image: zenika/alpine-chrome:latest
    container_name: chrome-headless
    restart: unless-stopped
    command:
      - "--no-sandbox"
      - "--disable-gpu"
      - "--disable-dev-shm-usage"
      - "--remote-debugging-address=0.0.0.0"
      - "--remote-debugging-port=3000"
      - "--headless"

  redis:
    image: redis:8.4.0
    container_name: norish-redis
    restart: unless-stopped
    volumes:
      - redis_data:/data

volumes:
  db_data:
  norish_data:
  redis_data:
```

**Environment Variables:**
```
APP_URL=https://recipe.example.com
MASTER_KEY=             ← generate with: openssl rand -base64 32
POSTGRES_PASSWORD=      ← generate with: openssl rand -hex 32
OPENAI_API_KEY=         ← generate at: platform.openai.com/api-keys
```

## AI Features
AI is enabled via OpenAI's `gpt-5-mini` model (`AI_API_KEY` above). This powers:
- AI-fallback recipe import when a page can't be parsed structurally
- auto-tagging, nutrition estimation, provenance, allergy detection
- Video phraseing and summarization for reels and other video content

> [!NOTE]
> Image generation in particular is billed per picture so I have disabled it for now.
