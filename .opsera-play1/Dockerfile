# =============================================================================
# 🐦 OPSERA DevOps Copilot - Multi-Stage Dockerfile for Vite React SPA
# Skill Version: 4.0.0
# =============================================================================

# Stage 1: Build the application
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies (use npm install for better compatibility with Lovable apps)
RUN npm install --legacy-peer-deps

# Copy source code
COPY . .

# Accept build-time environment variables as ARGs (Vite requires these at build time)
ARG VITE_SUPABASE_URL
ARG VITE_SUPABASE_PUBLISHABLE_KEY

# Convert ARGs to ENVs (Vite reads from process.env during build)
ENV VITE_SUPABASE_URL=$VITE_SUPABASE_URL
ENV VITE_SUPABASE_PUBLISHABLE_KEY=$VITE_SUPABASE_PUBLISHABLE_KEY

# Build the application
RUN npm run build

# Stage 2: Serve with NGINX
FROM nginx:alpine

# Create nginx cache directories and set permissions (RULE 33)
RUN mkdir -p /var/cache/nginx/client_temp \
    /var/cache/nginx/proxy_temp \
    /var/cache/nginx/fastcgi_temp \
    /var/cache/nginx/uwsgi_temp \
    /var/cache/nginx/scgi_temp && \
    chmod -R 777 /var/cache/nginx && \
    chmod 777 /var/log/nginx && \
    chmod 777 /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chmod 777 /var/run/nginx.pid

# Copy custom NGINX configuration
COPY .opsera-play1/nginx.conf /etc/nginx/conf.d/default.conf

# Copy built assets from builder stage
COPY --from=builder /app/dist /usr/share/nginx/html

# Expose port 8080 (non-privileged)
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1

# Start NGINX
CMD ["nginx", "-g", "daemon off;"]
