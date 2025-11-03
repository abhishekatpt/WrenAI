# Deploy WREN AI with a custom WrenUI image

This is a short guide to deploy WREN AI and replace the default WrenUI with a locally-built image.

## Quick flow

1. Download and run the launcher (macOS darwin-arm64 example):

   ```sh
   curl -L https://github.com/Canner/WrenAI/releases/latest/download/wren-launcher-darwin-arm64.tar.gz | tar -xz && ./wren-launcher-darwin-arm64
   ```

2. Enter your OpenAI credentials when prompted and select the LLM model.

   The launcher will start the default Docker instances for the WREN stack.

3. Stop and delete the WRENUI instance so you can replace it with your local image.


## Build a local `wren_ui` image

- Use Node.js 18 for building the UI.
- Copy/create a `.env` file inside `wren-ui/` with the following minimal example:

  ```properties
  DB_TYPE=sqlite
  SQLITE_FILE=testdb.sqlite3
  OTHER_SERVICE_USING_DOCKER=true
  ```

- From the repository root, build the Docker image (no cache):

   ```sh
   docker build -t wrenui_local -f wren-ui/Dockerfile wren-ui --no-cache
   ```


## Prepare the docker environment file


Create or edit `docker/.env` with the values below (this example file is used by the compose stacks):

```env
COMPOSE_PROJECT_NAME=wrenai
PLATFORM=linux/amd64

PROJECT_DIR=.

# service port
WREN_ENGINE_PORT=8080
WREN_ENGINE_SQL_PORT=7432
WREN_AI_SERVICE_PORT=5555
WREN_UI_PORT=3000
IBIS_SERVER_PORT=8000

# ai service settings
QDRANT_HOST=qdrant

# vendor keys
OPENAI_API_KEY=<YOUR_OPENAI_KEY>

# version (change these to the latest if needed)
WREN_PRODUCT_VERSION=0.28.0
WREN_ENGINE_VERSION=0.20.2
WREN_AI_SERVICE_VERSION=0.27.14
IBIS_SERVER_VERSION=0.20.2
WREN_UI_VERSION=0.31.2
WREN_BOOTSTRAP_VERSION=0.1.5

# user id (uuid v4)
USER_UUID=

# for other services
POSTHOG_API_KEY=
POSTHOG_HOST=https://app.posthog.com
TELEMETRY_ENABLED=true

GENERATION_MODEL=gpt-4o-mini
LANGFUSE_SECRET_KEY=
LANGFUSE_PUBLIC_KEY=

# the port exposed to the host
HOST_PORT=3000
AI_SERVICE_FORWARD_PORT=5555

# Wren UI
EXPERIMENTAL_ENGINE_RUST_VERSION=false

# Wren Engine
LOCAL_STORAGE=.
```

## Start the stack with the local UI image

1. If you rebuilt `wrenui_local`, update any compose files or service definitions to use `wrenui_local` as the image (or use an override compose file).

2. From the repository root, bring the stack up:

   ```sh
   docker compose -f docker/docker-compose-local.yaml up -d
   ```

3. Confirm the `wren-ui` service is running and pointing to your local image:

   ```sh
   docker compose -f docker/docker-compose-local.yaml ps
   ```


## Notes & tips

- If the compose file references an official image tag, you can either edit the `docker-compose.yaml` to use `wrenui_local` or build and tag your image with the same name used by compose.
- For macOS Apple Silicon, the compose `PLATFORM=linux/amd64` is often required if the images are amd64-only.
- Keep secrets (API keys) out of repo files and use environment variables or a secrets manager when possible.


## Example: Replace image with an override (optional)

Create `docker/docker-compose.override.yaml` with content similar to:

```yaml
services:
  wren-ui:
    image: wrenui_local:latest
    build: ./wren-ui
```

Then run the same `docker compose` up command; Docker Compose will prefer the override settings.


---

If you want, I can also update the existing `docker/README.md` or the `docker-compose.yaml` to use `wrenui_local` automatically. Tell me which file you want edited next.
