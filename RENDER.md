# Render deployment

AgenticCoach runs as one Render Web Service. The existing Docker image starts
both application processes under supervisord:

- Next.js frontend: public port `10000`
- FastAPI backend: internal port `8001`
- Persistent runtime data: `/app/data`

This preserves the same-origin `/api/*` and `/ws/*` proxy used locally, so chat
WebSockets do not need to cross a separate frontend deployment boundary.

## Deploy with the Blueprint

1. Push the repository to GitHub.
2. In Render, choose **New → Blueprint**.
3. Select the repository and branch containing `render.yaml`.
4. Review the service and create it.
5. Attach or confirm the persistent disk at `/app/data`.
6. Add provider credentials in the Render service environment settings, for
   example `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or the credentials for the
   selected provider. Do not commit secrets to `render.yaml`.
7. Open the generated `onrender.com` URL.
8. Complete `deeptutor init`-equivalent configuration from the Web Settings
   page, or let the first boot create the runtime settings under `/app/data`.

The checked-in `render.yaml` configures a 10 GB persistent disk. Increase it
before production use if knowledge bases, uploads, generated files, or memory
will be large.

## Environment variables

The manifest sets the process wiring:

```text
FRONTEND_PORT=10000
BACKEND_PORT=8001
DEEPTUTOR_API_BASE_URL=http://127.0.0.1:8001
DEEPTUTOR_AUTH_ENABLED=false
```

Set `DEEPTUTOR_AUTH_ENABLED=true` before exposing a private or multi-user
deployment, then configure the first administrator through `/register`.

Provider API keys and other credentials belong in Render's secret environment
variables, not in the repository.

## Local validation

Build and run the same image locally:

```bash
docker build -t arisehub-agent-playground .
docker run --rm --name arisehub-agent-playground \
  -p 3782:10000 \
  -v arisehub-agent-playground-data:/app/data \
  -e FRONTEND_PORT=10000 \
  -e BACKEND_PORT=8001 \
  -e DEEPTUTOR_API_BASE_URL=http://127.0.0.1:8001 \
  arisehub-agent-playground
```

Open <http://127.0.0.1:3782> and verify login, chat streaming, uploads, and a
restart with the same volume.
