# Production Dockerfile for Gateway Service
# Multi-stage build for optimized production image

FROM node:18-slim AS builder

# Set working directory
WORKDIR /app

# Copy package files for dependency installation
COPY package*.json ./

# Install ALL dependencies
RUN npm install && \
    npm cache clean --force

# Copy source code
COPY src/ ./src/

# Production stage
FROM node:18-slim AS production

# Install security updates and minimal tools
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    dumb-init \
    curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user for security
RUN groupadd -g 1001 nodejs && \
    useradd -r -u 1001 -g nodejs gateway

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm install --only=production && \
    npm cache clean --force

# Copy application source
COPY src/ ./src/

# Change ownership to non-root user
RUN chown -R gateway:nodejs /app

# Switch to non-root user
USER gateway

# Expose the gateway port
EXPOSE 5921

# Add health check for production monitoring
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:5921/health || exit 1

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]

# Start the production server
CMD ["npm", "start"]
