# AppFlowy Frontend Dockerfile
# AppFlowy uses Flutter for frontend, this builds the web version
# Alternative: Use official image if available: appflowyio/appflowy-frontend:latest

FROM ghcr.io/cirruslabs/flutter:stable as builder

WORKDIR /app

# Clone AppFlowy repository (using stable tag for stability)
# Try using a specific release tag for more stability
ARG APPFLOWY_VERSION=main
RUN git clone --depth 1 --branch ${APPFLOWY_VERSION} https://github.com/AppFlowy-IO/appflowy.git /tmp/appflowy || \
    (echo "Failed to clone main branch, trying latest tag" && \
     git clone --depth 1 https://github.com/AppFlowy-IO/appflowy.git /tmp/appflowy)

# Debug: Check what's in the cloned repository
RUN echo "=== AppFlowy repository structure ===" && \
    ls -la /tmp/appflowy/ && \
    echo "" && \
    echo "=== Top-level directories ===" && \
    find /tmp/appflowy -maxdepth 1 -type d | sort && \
    echo "" && \
    echo "=== Looking for frontend/pubspec.yaml ===" && \
    find /tmp/appflowy -maxdepth 3 -name "pubspec.yaml" -o -name "frontend" -type d | head -10

# Try to find and copy frontend code (handle different possible structures)
# AppFlowy frontend might be the root appflowy directory or in a subdirectory
RUN if [ -d "/tmp/appflowy/frontend" ] && [ -f "/tmp/appflowy/frontend/pubspec.yaml" ]; then \
        echo "Found frontend directory with pubspec.yaml" && \
        cp -r /tmp/appflowy/frontend /app/frontend; \
    elif [ -f "/tmp/appflowy/pubspec.yaml" ]; then \
        echo "Found pubspec.yaml at root, using entire repository as frontend" && \
        cp -r /tmp/appflowy /app/frontend; \
    elif [ -d "/tmp/appflowy/appflowy" ] && [ -f "/tmp/appflowy/appflowy/pubspec.yaml" ]; then \
        echo "Found appflowy directory with pubspec.yaml" && \
        cp -r /tmp/appflowy/appflowy /app/frontend; \
    else \
        echo "ERROR: Could not find frontend with pubspec.yaml. Full structure:" && \
        find /tmp/appflowy -type f -name "pubspec.yaml" && \
        find /tmp/appflowy -maxdepth 2 -type d && \
        exit 1; \
    fi

WORKDIR /app/frontend

# Check if pubspec.yaml exists and show structure
RUN echo "=== Frontend structure ===" && \
    ls -la && \
    if [ -f "pubspec.yaml" ]; then \
        echo "Found pubspec.yaml, building..." && \
        flutter pub get && \
        flutter build web --release; \
    else \
        echo "ERROR: pubspec.yaml not found. Contents:" && \
        find . -name "pubspec.yaml" && \
        ls -la && \
        exit 1; \
    fi

# Runtime stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/frontend/build/web /usr/share/nginx/html

# Copy nginx configuration for frontend
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

