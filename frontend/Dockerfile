# =====================
# Build stage
# =====================
FROM node:18-alpine AS build

WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Copy source and build
COPY . .
RUN npm run build

# =====================
# Production stage
# =====================
FROM nginx:alpine

# Create a non-root user
RUN addgroup -S nginx && adduser -S nginx -G nginx

# Copy nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Copy build artifacts
COPY --from=build /app/build /usr/share/nginx/html

# Make directories writable for OpenShift random UID
RUN chmod -R g+rwX /usr/share/nginx/html \
    && chmod -R g+rwX /var/cache/nginx \
    && chmod -R g+rwX /var/log/nginx \
    && chmod -R g+rwX /tmp /var/run/nginx \
    && chown -R nginx:nginx /usr/share/nginx/html /var/cache/nginx /var/log/nginx /tmp /var/run/nginx

# Switch to non-root user
USER nginx

# Expose port 8080
EXPOSE 8080

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1

# Start nginx with PID in /tmp
CMD ["nginx", "-g", "daemon off; pid /tmp/nginx.pid;"]
