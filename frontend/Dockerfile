# AppFlowy Frontend Dockerfile
# Builds AppFlowy Flutter web app
# Uses Flutter 3.27.4 as required by AppFlowy

FROM ubuntu:22.04 AS builder

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    unzip \
    xz-utils \
    zip \
    libglu1-mesa \
    build-essential \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Install Rust (required for WASM builds)
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="/root/.cargo/bin:${PATH}"

# Install wasm-pack (required for WASM builds)
RUN curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh

# Install Flutter 3.27.4 (specific version required by AppFlowy)
WORKDIR /opt
RUN curl -L https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.27.4-stable.tar.xz | tar xJ
ENV PATH="/opt/flutter/bin:${PATH}"

WORKDIR /app

# Clone AppFlowy repository
ARG APPFLOWY_VERSION=main
RUN git clone --depth 1 --branch ${APPFLOWY_VERSION} https://github.com/AppFlowy-IO/appflowy.git /tmp/appflowy

# Copy frontend directory (need the whole frontend for build tools)
RUN cp -r /tmp/appflowy/frontend /app/frontend

WORKDIR /app/frontend/appflowy_flutter

# Fix dependency conflict: override leak_tracker to compatible version
# AppFlowy already has dependency_overrides, append to it
RUN if ! grep -q "leak_tracker:" pubspec.yaml; then \
        sed -i '/^dependency_overrides:/a \  leak_tracker: ^11.0.1' pubspec.yaml; \
    fi

# Get Flutter dependencies
RUN flutter pub get

# Build WASM components first (required before Flutter web build)
WORKDIR /app/frontend
RUN crates=("lib-dispatch" "lib-infra" "flowy-notification" "flowy-date" "flowy-error" "collab-integrate" "flowy-document") && \
    for crate in "${crates[@]}"; do \
        echo "Building $crate with wasm-pack..." && \
        cd rust-lib/$crate && \
        wasm-pack build || { echo "Build failed for $crate"; exit 1; } && \
        cd /app/frontend; \
    done

# Build Flutter web app
WORKDIR /app/frontend/appflowy_flutter
RUN flutter build web --release

# Runtime stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/frontend/appflowy_flutter/build/web /usr/share/nginx/html

# Copy nginx configuration for frontend
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
