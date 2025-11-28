# ============================================
# BASE - Node.js con pnpm abilitato
# ============================================
FROM node:20-alpine AS base
RUN corepack enable && corepack prepare pnpm@9.15.5 --activate

# ============================================
# BUILDER - Turbo prune del monorepo
# ============================================
FROM base AS builder
WORKDIR /app

# Copia l'intero monorepo
COPY . .

# Prune usando pnpm dlx
RUN pnpm dlx turbo@2.6.1 prune medusa-backend --docker

# ============================================
# INSTALLER - Installazione + Build
# ============================================
FROM base AS installer
WORKDIR /app

# Copia i file di dipendenze dal builder
COPY --from=builder /app/out/json/ ./
COPY --from=builder /app/out/pnpm-lock.yaml ./pnpm-lock.yaml

# Installa TUTTE le dipendenze (incluse devDependencies)
RUN pnpm install --frozen-lockfile

# Copia il codice sorgente completo
COPY --from=builder /app/out/full/ ./

# ⚠️ CRITICO: Crea il file .env usando le variabili iniettate da Coolify
# Coolify passa le variabili come secrets montati, quindi sono disponibili qui
# RUN cat > apps/backend/.env <<EOF
# DATABASE_URL=${DATABASE_URL}
# JWT_SECRET=${JWT_SECRET}
# COOKIE_SECRET=${COOKIE_SECRET}
# REDIS_URL=${REDIS_URL}
# STORE_CORS=${STORE_CORS}
# ADMIN_CORS=${ADMIN_CORS}
# AUTH_CORS=${AUTH_CORS}
# BACKEND_PUBLIC_URL=${BACKEND_PUBLIC_URL}
# MEDUSA_WORKER_MODE=${MEDUSA_WORKER_MODE}
# MINIO_ACCESS_KEY=${MINIO_ACCESS_KEY}
# MINIO_BUCKET=${MINIO_BUCKET}
# MINIO_ENDPOINT=${MINIO_ENDPOINT}
# MINIO_SECRET_KEY=${MINIO_SECRET_KEY}
# MEDUSA_DISABLE_ADMIN=${MEDUSA_DISABLE_ADMIN}
# EOF

# Esporta tutte le env disponibili (potrebbe essere overkill)
RUN printenv > apps/backend/.env

# Esegui il build con Turborepo
RUN pnpm dlx turbo@2.6.1 run build --filter=medusa-backend && \
    rm -rf node_modules/.cache && \
    rm -f apps/backend/.env

# ============================================
# RUNNER - Immagine finale per produzione
# ============================================
FROM base AS runner
WORKDIR /app

# Copia node_modules e file compilati dall'installer
COPY --from=installer /app/node_modules ./node_modules
COPY --from=installer /app/apps ./apps
COPY --from=installer /app/package.json ./package.json
COPY --from=installer /app/pnpm-workspace.yaml ./pnpm-workspace.yaml
COPY --from=installer /app/pnpm-lock.yaml ./pnpm-lock.yaml

# Esponi la porta del backend Medusa
EXPOSE 9000

# Avvia il backend
# Le variabili d'ambiente reali verranno iniettate da Coolify a runtime
CMD ["pnpm", "--filter", "medusa-backend", "start"]