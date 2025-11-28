# Dockerfile
FROM node:20-alpine AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

# Pruner stage
FROM base AS builder
RUN apk add --no-cache libc6-compat
WORKDIR /app
RUN pnpm add -g turbo
COPY . .
RUN turbo prune medusa-storefront --docker

# Installer stage
FROM base AS installer
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY --from=builder /app/out/json/ ./
COPY --from=builder /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
RUN pnpm install --frozen-lockfile

COPY --from=builder /app/out/full/ ./

# Creat BASE .env before build
RUN mkdir -p apps/storefront
RUN printenv > apps/storefront/.env

ENV NEXT_TELEMETRY_DISABLED=1

RUN pnpm turbo build --filter=medusa-storefront

# Runner stage
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# copy everything needed INCLUDED .env with the fetched key
COPY --from=installer --chown=nextjs:nodejs /app/apps/storefront ./apps/storefront
COPY --from=installer --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=installer --chown=nextjs:nodejs /app/package.json ./package.json

# Don't override .env, add ony required runtime variables
RUN printenv | grep -E '^(PORT|HOSTNAME|NODE_ENV)=' >> apps/storefront/.env || true
RUN chown nextjs:nodejs apps/storefront/.env

USER nextjs

EXPOSE 3000

ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

WORKDIR /app/apps/storefront

CMD ["node_modules/.bin/next", "start"]