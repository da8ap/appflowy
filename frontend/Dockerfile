# AppFlowy Frontend Dockerfile
# Builds AppFlowy Flutter web app

FROM ghcr.io/cirruslabs/flutter:stable as builder

WORKDIR /app

# Clone AppFlowy repository
ARG APPFLOWY_VERSION=main
RUN git clone --depth 1 --branch ${APPFLOWY_VERSION} https://github.com/AppFlowy-IO/appflowy.git /tmp/appflowy

# The frontend is in frontend/appflowy_flutter
RUN if [ -d "/tmp/appflowy/frontend/appflowy_flutter" ]; then \
        echo "Found AppFlowy Flutter app" && \
        cp -r /tmp/appflowy/frontend/appflowy_flutter /app/appflowy_flutter; \
    else \
        echo "ERROR: Could not find frontend/appflowy_flutter" && \
        ls -la /tmp/appflowy/frontend/ && \
        exit 1; \
    fi

WORKDIR /app/appflowy_flutter

# Build Flutter web app
RUN flutter pub get && \
    flutter build web --release

# Runtime stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/appflowy_flutter/build/web /usr/share/nginx/html

# Copy nginx configuration for frontend
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
