# React Containerization with Docker Multi-stage Builds

This project demonstrates a production-ready setup for a React application using:

- Docker multi-stage builds
- Nginx static file serving for production
- Runtime environment variable injection
- Image size optimization via Alpine base images and minimal runtime stage

## 1. Build stage Dockerfile

`Dockerfile` uses:

- `node:20-alpine` to build React assets
- `nginx:1.27-alpine` to serve only built files

This keeps the final image small because Node and source files are not shipped in the runtime layer.

## 2. Production-ready Nginx configuration

`nginx/default.conf` includes:

- SPA routing fallback (`try_files ... /index.html`)
- Asset caching for immutable static files
- No-cache policy for `env-config.js`
- Basic security headers
- `/health` endpoint

## 3. Environment variables

There are two env modes:

- Build-time: `VITE_API_BASE_URL` (used during `npm run build`)
- Runtime: `API_BASE_URL`, `APP_ENV` (injected into `env-config.js` at container startup)

React reads values from `window.__ENV__` first, then falls back to build-time variable.

## 4. Build and run with Docker

### Build image

```bash
docker build -t react-docker-multistage --build-arg VITE_API_BASE_URL=https://build.api.local .
```

### Run container

```bash
docker run --rm -p 8080:80 -e API_BASE_URL=https://runtime.api.local -e APP_ENV=production react-docker-multistage
```

Open http://localhost:8080

## 5. Build and run with Docker Compose

```bash
docker compose up --build
```

Use `.env.example` as a template for your own `.env` file.
