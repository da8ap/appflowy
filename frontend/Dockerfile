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
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Install ruamel.yaml for YAML parsing that preserves formatting
RUN pip3 install --no-cache-dir ruamel.yaml

# Install Rust (required for WASM builds)
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="/root/.cargo/bin:${PATH}"

# Install wasm-pack (required for WASM builds)
RUN curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh

# Install Flutter 3.27.4 (specific version required by AppFlowy)
WORKDIR /opt
RUN curl -L https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.27.4-stable.tar.xz | tar xJ && \
    chmod -R 755 /opt/flutter
ENV PATH="/opt/flutter/bin:${PATH}"

# Create non-root user (Flutter discourages running as root)
RUN useradd -m -s /bin/bash flutter && \
    chown -R flutter:flutter /opt/flutter

# Enable web support as non-root user (set HOME explicitly)
USER flutter
ENV HOME=/home/flutter
RUN flutter config --enable-web
USER root

WORKDIR /app

# Clone AppFlowy repository
ARG APPFLOWY_VERSION=main
RUN git clone --depth 1 --branch ${APPFLOWY_VERSION} https://github.com/AppFlowy-IO/appflowy.git /tmp/appflowy

# Copy frontend directory (need the whole frontend for build tools)
RUN cp -r /tmp/appflowy/frontend /app/frontend && \
    chown -R flutter:flutter /app/frontend

WORKDIR /app/frontend/appflowy_flutter
USER root

# Verify pubspec.yaml exists
RUN test -f pubspec.yaml || (echo "ERROR: pubspec.yaml not found!" && ls -la && exit 1)

# DEBUG: Show pubspec.yaml before modification
RUN echo "=== BEFORE MODIFICATION ===" && \
    echo "Current directory: $(pwd)" && \
    echo "pubspec.yaml exists: $(test -f pubspec.yaml && echo 'YES' || echo 'NO')" && \
    grep -A 2 -B 2 "leak_tracker" pubspec.yaml || echo "leak_tracker not found" && \
    echo "=== dependency_overrides section ===" && \
    grep -A 10 "dependency_overrides:" pubspec.yaml || echo "dependency_overrides not found"

# Fix dependency conflict: remove leak_tracker from dependencies, add to dependency_overrides
# Use ruamel.yaml to preserve original formatting
RUN python3 << 'PYTHON_SCRIPT'
from ruamel.yaml import YAML
import sys

try:
    print("=== PYTHON SCRIPT STARTING ===")
    
    yaml = YAML()
    yaml.preserve_quotes = True
    yaml.width = 4096
    
    # Read pubspec.yaml preserving formatting
    with open('pubspec.yaml', 'r') as f:
        data = yaml.load(f)
    
    print("Loaded pubspec.yaml successfully")
    
    # Remove leak_tracker from dependencies if it exists
    if 'dependencies' in data and 'leak_tracker' in data['dependencies']:
        del data['dependencies']['leak_tracker']
        print("SUCCESS: Removed leak_tracker from dependencies")
    else:
        print("INFO: leak_tracker not found in dependencies")
    
    # Ensure dependency_overrides exists
    if 'dependency_overrides' not in data:
        data['dependency_overrides'] = {}
        print("Created dependency_overrides section")
    
    # Add leak_tracker to dependency_overrides
    data['dependency_overrides']['leak_tracker'] = '^11.0.1'
    print("SUCCESS: Added leak_tracker: ^11.0.1 to dependency_overrides")
    
    # Write back preserving formatting
    with open('pubspec.yaml', 'w') as f:
        yaml.dump(data, f)
    
    print("=== PYTHON SCRIPT COMPLETED SUCCESSFULLY ===")
    
except Exception as e:
    print(f"ERROR: Python script failed: {e}")
    import traceback
    traceback.print_exc()
    sys.exit(1)
PYTHON_SCRIPT

# DEBUG: Show pubspec.yaml after modification
RUN echo "=== AFTER MODIFICATION ===" && \
    grep -A 2 -B 2 "leak_tracker" pubspec.yaml || echo "leak_tracker not found" && \
    echo "=== dependency_overrides section ===" && \
    grep -A 10 "dependency_overrides:" pubspec.yaml || echo "dependency_overrides not found"

# Verify pubspec.yaml is valid YAML after modification
RUN python3 -c "from ruamel.yaml import YAML; y = YAML(); y.load(open('pubspec.yaml'))" || \
    (echo "ERROR: pubspec.yaml is invalid YAML after modification!" && cat pubspec.yaml && exit 1)

# Get Flutter dependencies - capture full error output (run as flutter user)
USER flutter
ENV HOME=/home/flutter
RUN echo "=== RUNNING FLUTTER PUB GET ===" && \
    echo "Current directory: $(pwd)" && \
    echo "Flutter version: $(flutter --version | head -1)" && \
    flutter pub get 2>&1 | tee /tmp/pub_get_output.log || \
    (echo "=== FLUTTER PUB GET FAILED ===" && \
     cat /tmp/pub_get_output.log && \
     echo "=== PUBSPEC.YAML CONTENT ===" && \
     cat pubspec.yaml && \
     exit 1)
USER root

# Build WASM components first (required before Flutter web build)
# Rust/wasm-pack can run as flutter user
USER flutter
ENV HOME=/home/flutter
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

# Switch back to root for copying files
USER root

# Runtime stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/frontend/appflowy_flutter/build/web /usr/share/nginx/html

# Copy nginx configuration for frontend
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
