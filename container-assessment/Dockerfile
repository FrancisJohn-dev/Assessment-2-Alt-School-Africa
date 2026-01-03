# -------- Build Stage --------
FROM node:20-alpine AS builder
WORKDIR /app

# Copy package files and install dependencies
COPY app/package*.json ./
RUN npm install

# Copy app source code
COPY app/ .
RUN npm run build || echo "no build step"

# -------- Runtime Stage --------
FROM node:20-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy from builder stage
COPY --from=builder /app /app

# Switch to non-root user
USER appuser

# Expose port and health check
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s CMD wget -qO- http://localhost:3000/health || exit 1

# Start the app
CMD ["node", "index.js"]
