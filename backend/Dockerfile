FROM node:18-slim AS builder

WORKDIR /app

COPY package*.json ./
COPY tsconfig.json ./

RUN npm install && \
    npm cache clean --force

COPY src/ ./src/

RUN npm run build

FROM node:18-slim AS production

RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    dumb-init \
    curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN groupadd -g 1001 nodejs && \
    useradd -r -u 1001 -g nodejs backend

WORKDIR /app

COPY package*.json ./

RUN npm install --only=production && \
    npm cache clean --force

COPY --from=builder /app/dist ./dist

RUN chown -R backend:nodejs /app

USER backend

EXPOSE 3847

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:3847/api/health || exit 1

ENTRYPOINT ["dumb-init", "--"]

CMD ["npm", "start"]
