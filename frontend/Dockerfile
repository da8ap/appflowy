# AppFlowy Frontend Dockerfile
# Builds AppFlowy Flutter web app

FROM debian:stable-slim AS builder

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
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install ruamel.yaml for YAML parsing
RUN pip3 install --no-cache-dir --break-system-packages ruamel.yaml

# Create non-root user FIRST (Flutter must not run as root)
RUN useradd -m -s /bin/bash flutteruser
USER flutteruser
ENV HOME=/home/flutteruser
WORKDIR /home/flutteruser

# Install Flutter via git clone (proven method)
RUN git clone https://github.com/flutter/flutter.git $HOME/flutter
ENV PATH="$HOME/flutter/bin:$PATH"

# Set Flutter to stable channel and enable web
RUN flutter channel stable && \
    flutter upgrade && \
    flutter config --enable-web

# Install Rust for flutteruser (required for WASM builds)
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="$HOME/.cargo/bin:$PATH"

# Install wasm-pack
RUN cargo install wasm-pack

# Clone AppFlowy repository
RUN git clone --depth 1 https://github.com/AppFlowy-IO/appflowy.git $HOME/appflowy

WORKDIR $HOME/appflowy/frontend/appflowy_flutter

# Fix dependency conflict using Python script (run as flutteruser who owns the files)
RUN python3 << 'PYTHON_SCRIPT'
from ruamel.yaml import YAML
import sys

try:
    yaml = YAML()
    yaml.preserve_quotes = True
    yaml.width = 4096
    
    with open('pubspec.yaml', 'r') as f:
        data = yaml.load(f)
    
    # Remove leak_tracker from dependencies if it exists
    if 'dependencies' in data and 'leak_tracker' in data['dependencies']:
        del data['dependencies']['leak_tracker']
        print("Removed leak_tracker from dependencies")
    
    # Ensure dependency_overrides exists
    if 'dependency_overrides' not in data:
        data['dependency_overrides'] = {}
    
    # Add leak_tracker to dependency_overrides
    data['dependency_overrides']['leak_tracker'] = '^11.0.1'
    print("Added leak_tracker: ^11.0.1 to dependency_overrides")
    
    with open('pubspec.yaml', 'w') as f:
        yaml.dump(data, f)
    
    print("pubspec.yaml modified successfully")
    
except Exception as e:
    print(f"ERROR: {e}")
    sys.exit(1)
PYTHON_SCRIPT

# Get Flutter dependencies
RUN flutter pub get

# Build Flutter web app
RUN flutter build web --release

# Runtime stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /home/flutteruser/appflowy/frontend/appflowy_flutter/build/web /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
