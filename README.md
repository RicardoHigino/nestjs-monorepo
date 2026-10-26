# NestJS Monorepo on Google Cloud Run

A reference setup for a **NestJS monorepo**: two applications and a shared library built from one codebase and one Docker image, then deployed as two separate **Cloud Run** services by a single **Cloud Build** pipeline.

## What It Demonstrates

- **Nest CLI monorepo mode** (`nest-cli.json`) with:
  - `apps/first-app`: exposes `GET /first`, `GET /second` and `GET /library`. It reuses a service from `second-app` and the shared library.
  - `apps/second-app`: exposes `GET /`.
  - `libs/my-library`: a shared Nest module and service imported through the `@library/my-library` path alias.
- **One image, many services**: `entrypoint.sh` reads the `SERVER_NAME` environment variable and starts either `first-app` or `second-app` from the same container.
- **CI/CD on GCP** (`cloudbuild.yaml`): builds the image, pushes it to Artifact Registry tagged with `$COMMIT_SHA`, then runs `gcloud run deploy` twice (for `first-app` and `second-app`), each with its own `SERVER_NAME`.

## Tech Stack

Node.js 18 · TypeScript · NestJS 9 (Express) · Jest · ESLint + Prettier · Docker · Google Cloud Build · Artifact Registry · Cloud Run

## Getting Started

```bash
npm install
npm run start:dev           # first-app (default project) in watch mode, port 8080
npm run start:second-app    # second-app
```

Build and run in production mode:

```bash
npm run build                                   # builds the default project (first-app)
npx nest build second-app                       # builds second-app
npm run start:first:prod    # node dist/apps/first-app/main
npm run start:second:prod   # node dist/apps/second-app/main
```

The port comes from the `PORT` environment variable and defaults to `8080`.

### Docker

```bash
docker build -t nestjs-monorepo .
docker run -p 8080:8080 -e SERVER_NAME=first-app nestjs-monorepo
docker run -p 8081:8080 -e SERVER_NAME=second-app nestjs-monorepo
```

### Cloud Build Deployment

Create a Cloud Build trigger with these substitutions: `_AR_HOSTNAME` (e.g. `us-central1-docker.pkg.dev`), `_AR_REPO`, `_REPO_NAME` and `_DEPLOY_REGION`.

## Project Structure

```
apps/
  first-app/src/     # main.ts, app.module/controller/service
  second-app/src/    # main.ts, second-app.module/controller/service
libs/
  my-library/src/    # shared module and service
Dockerfile           # node:18, build, entrypoint
entrypoint.sh        # picks the app from SERVER_NAME
cloudbuild.yaml      # build → push → deploy 2 Cloud Run services
nest-cli.json        # monorepo project definitions
```
