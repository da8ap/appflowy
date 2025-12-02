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

# DEBUG: Show pubspec.yaml before modification
RUN echo "=== BEFORE MODIFICATION ===" && \
    grep -A 2 -B 2 "leak_tracker" pubspec.yaml || echo "leak_tracker not found" && \
    echo "=== dependency_overrides section ===" && \
    grep -A 10 "dependency_overrides:" pubspec.yaml || echo "dependency_overrides not found"

# Fix dependency conflict: remove leak_tracker from dependencies, add to dependency_overrides
RUN python3 << 'PYTHON_SCRIPT'
import re
import sys

try:
    print("=== PYTHON SCRIPT STARTING ===")
    
    with open('pubspec.yaml', 'r') as f:
        content = f.read()
    
    original_content = content
    print(f"Read pubspec.yaml: {len(content)} characters")
    
    # Remove leak_tracker from dependencies section
    before_remove = content
    content = re.sub(r'^\s+leak_tracker:\s+\^10\.0\.0\s*$', '', content, flags=re.MULTILINE)
    if before_remove != content:
        print("SUCCESS: Removed leak_tracker from dependencies")
    else:
        print("WARNING: leak_tracker not found in dependencies (might already be removed)")
    
    # Add leak_tracker to dependency_overrides if not already there
    if 'leak_tracker: ^11.0.1' not in content:
        if 'dependency_overrides:' in content:
            # Find dependency_overrides section and add leak_tracker
            content = re.sub(
                r'(dependency_overrides:\s*\n)',
                r'\1  leak_tracker: ^11.0.1\n',
                content,
                count=1
            )
            print("SUCCESS: Added leak_tracker to existing dependency_overrides")
        else:
            # Add dependency_overrides section at the end
            content += '\ndependency_overrides:\n  leak_tracker: ^11.0.1\n'
            print("SUCCESS: Created dependency_overrides section with leak_tracker")
    else:
        print("INFO: leak_tracker override already exists")
    
    with open('pubspec.yaml', 'w') as f:
        f.write(content)
    
    # Verify the change was made
    if original_content == content:
        print("ERROR: pubspec.yaml was not modified!")
        sys.exit(1)
    
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

# Get Flutter dependencies - capture full error output
RUN echo "=== RUNNING FLUTTER PUB GET ===" && \
    flutter pub get 2>&1 | tee /tmp/pub_get_output.log || \
    (echo "=== FLUTTER PUB GET FAILED ===" && \
     cat /tmp/pub_get_output.log && \
     echo "=== PUBSPEC.YAML CONTENT ===" && \
     cat pubspec.yaml && \
     exit 1)

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
