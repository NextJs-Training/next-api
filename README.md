---
title: NextJS Prisma
description: A NextJS app using Prisma with a PostgreSQL database
tags:
  - next
  - prisma
  - postgresql
  - typescript
---


# Prisma + Next.js + Serverless-Next.js AWS Component

## Requirements

- Node 10+
- Yarn (or NPM if you prefer)

## Getting Started

After cloning the repository, you can run `yarn` to install the dependencies and then start the application with `yarn dev`.

```bash
$ git clone https://github.com/Estevamsl/nextjs-prisma-api
$ yarn
$ yarn dev
```

## What's next? How do I make an app with this?

We try to keep this project as simple as possible, so you can start with the most basic configuration and then move on to more advanced configuration.

If you are not familiar with the different technologies used in this project, please refer to the respective docs. If you still are in the wind, please join our [Discord](https://t3.gg/discord) and ask for help.

- [Next-Auth.js](https://next-auth.js.org)
- [Prisma](https://prisma.io)

## ✨ Features

- Prisma
- NextJS
- Postgres
- TypeScript

## 💁‍♀️ How to use

```bash
# development
$ npm run start

# watch mode
$ npm run start:dev

# production mode
$ npm run start:prod
```

## TypeScript

This template is set up to get started with
[TypeScript](https://www.typescriptlang.org/). However, if you need JS only, you
can rename the `.ts` to `.js`, remove the types, and remove `tsconfig.json`.
Alternatively, you could keep `tsconfig.json` for better type hints.

## Database

### Provider

The project is set up to work with a `PostgreSQL` database. If you would like to
change that, head to the `schema.prisma` and change it.

### Connect

At the root of this project, next to your `package.json`,  create an `.env`
file. Insert your database environment variables, they will automatically get
deployed. Do not commit this file.

```
DATABASE_URL=postgres://user:password@host:port/db_name
```

## How do I deploy this?

### Vercel

We recommend deploying to [Vercel](https://vercel.com/?utm_source=t3-oss&utm_campaign=oss). It makes it super easy to deploy NextJs apps.

- Push your code to a GitHub repository.
- Go to [Vercel](https://vercel.com/?utm_source=t3-oss&utm_campaign=oss) and sign up with GitHub.
- Create a Project and import the repository you pushed your code to.
- Add your environment variables.
- Click **Deploy**
- Now whenever you push a change to your repository, Vercel will automatically redeploy your website!

### Docker

You can also dockerize this stack and deploy a container.

Please note that Next.js requires a different process for buildtime (available in the frontend, prefixed by `NEXT_PUBLIC`) and runtime environment, server-side only, variables. In this demo we are using two variables, `NEXT_PUBLIC_FOO` and `BAR`. Pay attention to their positions in the `Dockerfile`, command-line arguments, and `docker-compose.yml`.

1. In your [next.config.mjs](./next.config.mjs), add the `standalone` output-option to your config:

   ```diff
     export default defineNextConfig({
       reactStrictMode: true,
       swcMinify: true,
   +   output: "standalone",
     });
   ```

2. Remove the `env`-import from [next.config.mjs](./next.config.mjs):

   ```diff
   - import { env } from "./src/env/server.mjs";
   ```

3. Create a `.dockerignore` file with the following contents:
   <details>
   <summary>.dockerignore</summary>

   ```
   .env
   Dockerfile
   .dockerignore
   node_modules
   npm-debug.log
   README.md
   .next
   .git
   ```

  </details>

4. Create a `Dockerfile` with the following contents:
   <details>
   <summary>Dockerfile</summary>

   ```Dockerfile
   ########################
   #         DEPS         #
   ########################

   # Install dependencies only when needed
   # TODO: re-evaluate if emulation is still necessary on arm64 after moving to node 18
   FROM --platform=linux/amd64 node:16-alpine AS deps
   # Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
   RUN apk add --no-cache libc6-compat
   WORKDIR /app

   # Install dependencies based on the preferred package manager
   COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
   RUN \
     if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
     elif [ -f package-lock.json ]; then npm ci; \
     elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i; \
     else echo "Lockfile not found." && exit 1; \
     fi

   ########################
   #        BUILDER       #
   ########################

   # Rebuild the source code only when needed
   # TODO: re-evaluate if emulation is still necessary on arm64 after moving to node 18
   FROM --platform=linux/amd64 node:16-alpine AS builder

   ARG NEXT_PUBLIC_FOO
   ARG BAR

   WORKDIR /app
   COPY --from=deps /app/node_modules ./node_modules
   COPY . .

   # Next.js collects completely anonymous telemetry data about general usage.
   # Learn more here: https://nextjs.org/telemetry
   # Uncomment the following line in case you want to disable telemetry during the build.
   # ENV NEXT_TELEMETRY_DISABLED 1

   RUN \
     if [ -f yarn.lock ]; then yarn build; \
     elif [ -f package-lock.json ]; then npm run build; \
     elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm run build; \
     else echo "Lockfile not found." && exit 1; \
     fi

   ########################
   #        RUNNER        #
   ########################

   # Production image, copy all the files and run next
   # TODO: re-evaluate if emulation is still necessary after moving to node 18
   FROM --platform=linux/amd64 node:16-alpine AS runner
   # WORKDIR /usr/app
   WORKDIR /app

   ENV NODE_ENV production
   # Uncomment the following line in case you want to disable telemetry during runtime.
   # ENV NEXT_TELEMETRY_DISABLED 1

   RUN addgroup --system --gid 1001 nodejs
   RUN adduser --system --uid 1001 nextjs

   COPY --from=builder /app/next.config.mjs ./
   COPY --from=builder /app/public ./public
   COPY --from=builder /app/package.json ./package.json

   # Automatically leverage output traces to reduce image size
   # https://nextjs.org/docs/advanced-features/output-file-tracing
   COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
   COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

   USER nextjs

   EXPOSE 3000

   ENV PORT 3000

   CMD ["node", "server.js"]
   ```

  </details>

5. To build and run this image locally, run:

   ```bash
   docker build -t ct3a -e NEXT_PUBLIC_FOO=foo .
   docker run -p 3000:3000 -e BAR="bar" ct3a
   ```

6. You can also use a PaaS such as [Railway's](https://railway.app) automated [Dockerfile deployments](https://docs.railway.app/deploy/dockerfiles) to deploy your app.

### docker-compose

You can also use docker-compose to build and run the container.

1. Follow steps 1-4 above

2. Create a `docker-compose.yml` file with the following:

   <details>
   <summary>docker-compose.yml</summary>

   ```yaml
   version: "3.7"
   services:
     app:
       platform: "linux/amd64"
       build:
         context: .
         dockerfile: Dockerfile
         args:
           NEXT_PUBLIC_FOO: "foo"
       working_dir: /app
       ports:
         - "3000:3000"
       image: t3-app
       environment:
         - BAR=bar
   ```

   </details>

3. Run this using `docker-compose up`.


## 📝 Notes

This app is a simple todo list where the data is persisted to Postgres. [Prisma
migrations](https://www.prisma.io/docs/concepts/components/prisma-migrate#prisma-migrate)
can be created with `railway run yarn migrate:dev` and deployed with `railway run yarn migrate:deploy`. The Prisma client can be regenerated with
`yarn generate`.

[swr](https://swr.vercel.app/) is used to fetch data on the client and perform optimistic updates.
