# Docerized Next.js with hot reload

Everything was done on windows 11 war machine

## the required stuff to install:

To recreate it without clonning it you will need to have installed:

- node.js
- docker desktop (maybe there is better version)

## First step

## First create the next.js app by running:

```bash
npx create-next-app@latest .
```

## Then put this in next.config.ts:

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
};

// this is the important stuff else is default ↓

module.exports = {
  output: "standalone",
};

// ↑

export default nextConfig;
```

The added property:

```ts
module.exports = {
  output: "standalone",
};
```

Is not that important, but I should reduce size of next.js in the container and make it work better.
( at least this is how I unserstood the explenations of that )

Then you can do: docker init, but I did all stuff manually

## create compose.yml

```yml
services:
  file-validation-dev2:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    container_name: file-validation-dev2
    ports:
      - 3000:3000
    environment:
      NODE_ENV: development
      DOCKER_ENV: "true"
    # This is for some old version of hot reloads called binding
    # volumes:
    #   - ./app:/file-validation/app:ro
    #   - ./package.json:/file-validation/package.json
    command: npm run dev
    develop:
      watch:
        - action: sync
          path: ./app
          target: /file-validation/app
        - action: rebuild
          path: ./package.json
    restart: unless-stopped
```

## Create Dockerfile

```DOCKERFILE
FROM node:22-alpine3.22 AS base

WORKDIR /file-validation

FROM base AS deps

COPY package.json ./

# for some reason if you don't copy the package-lock.json it doesn't work
COPY package-lock.json ./

RUN --mount=type=cache,target=/root/.npm,sharing=locked \
    npm ci --no-audit --no-fund && \
    npm cache clean --force


FROM deps AS development

ENV NODE_ENV=development \
    NPM_CONFIG_LOGLEVEL=warn

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

## At last add .dockerignore file

This makes so you don't copy unnecessary stuff to your docker container

```ignore

# Optimized .dockerignore for Node.js + React Todo App
# Based on actual project structure

# Version control
.git/
.github/
.gitignore

# Dependencies (installed in container)
node_modules/

# Build outputs (built in container)
dist/

# Environment files
.env*

# Development files
.vscode/
*.log
coverage/
.eslintcache

# OS files
.DS_Store
Thumbs.db

# Documentation
*.md
docs/

# Deployment configs
compose.yml
Taskfile.yml
nodejs-sample-kubernetes.yaml

# Non-essential configs (keep build configs)
*.config.js
!vite.config.ts
!esbuild.config.js
!tailwind.config.js
!postcss.config.js
!tsconfig.json
```

## RUN docker compose up --watch

now you run:

```bash
docker compose up --watch
```

After that the stuff should be working and everything in directory app should be auto synced / pulled in to your docker container

## Good luck travler
