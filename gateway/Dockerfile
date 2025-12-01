FROM node:18-slim AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install && \
    npm cache clean --force

COPY src/ ./src/

FROM node:18-slim AS production

RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    dumb-init \
    curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN groupadd -g 1001 nodejs && \
    useradd -r -u 1001 -g nodejs gateway

WORKDIR /app

COPY package*.json ./

RUN npm install --only=production && \
    npm cache clean --force

COPY src/ ./src/

RUN chown -R gateway:nodejs /app

USER gateway

EXPOSE 5921

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:5921/health || exit 1

ENTRYPOINT ["dumb-init", "--"]

CMD ["npm", "start"]
