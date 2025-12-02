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
    python3 \
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

# Fix dependency conflict: remove leak_tracker from dependencies, add to dependency_overrides
RUN python3 << 'PYTHON_SCRIPT'
import re

with open('pubspec.yaml', 'r') as f:
    content = f.read()

# Remove leak_tracker from dependencies section
content = re.sub(r'^\s+leak_tracker:\s+\^10\.0\.0\s*$', '', content, flags=re.MULTILINE)

# Add leak_tracker to dependency_overrides if not already there
if 'leak_tracker:' not in content or 'dependency_overrides' not in content.split('leak_tracker:')[0]:
    # Find dependency_overrides section and add leak_tracker
    if 'dependency_overrides:' in content:
        content = re.sub(
            r'(dependency_overrides:\s*\n)',
            r'\1  leak_tracker: ^11.0.1\n',
            content,
            count=1
        )
    else:
        # Add dependency_overrides section at the end
        content += '\ndependency_overrides:\n  leak_tracker: ^11.0.1\n'

with open('pubspec.yaml', 'w') as f:
    f.write(content)
PYTHON_SCRIPT

# Get Flutter dependencies
RUN flutter pub get

# Build WASM components first (required before Flutter web build)
WORKDIR /app/frontend
RUN BASE_DIR=$(pwd) && \
    crates=("lib-dispatch" "lib-infra" "flowy-notification" "flowy-date" "flowy-error" "collab-integrate" "flowy-document") && \
    for crate in "${crates[@]}"; do \
        echo "Building $crate with wasm-pack..." && \
        cd "$BASE_DIR/rust-lib/$crate" || { echo "Failed to enter directory $crate"; exit 1; } && \
        wasm-pack build || { echo "Build failed for $crate"; exit 1; } && \
        cd "$BASE_DIR"; \
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
