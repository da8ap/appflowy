# AppFlowy Frontend Dockerfile
# AppFlowy uses Flutter for frontend, this builds the web version
# Alternative: Use official image if available: appflowyio/appflowy-frontend:latest

FROM ghcr.io/cirruslabs/flutter:stable as builder

WORKDIR /app

# Clone AppFlowy repository (using stable tag for stability)
ARG APPFLOWY_VERSION=main
RUN git clone --depth 1 --branch ${APPFLOWY_VERSION} https://github.com/AppFlowy-IO/appflowy.git /tmp/appflowy

# Copy frontend code
RUN cp -r /tmp/appflowy/frontend /app/frontend

WORKDIR /app/frontend

# Build Flutter web app
# Note: Adjust build command based on actual AppFlowy frontend structure
RUN flutter pub get && \
    flutter build web --release || \
    (echo "Build failed, checking structure..." && ls -la && exit 1)

# Runtime stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/frontend/build/web /usr/share/nginx/html

# Copy nginx configuration for frontend
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

