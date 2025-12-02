# AppFlowy Backend Dockerfile
# This builds AppFlowy backend from source
# Alternative: Use official image if available: appflowyio/appflowy-backend:latest

FROM rust:1.75-slim as builder

# Install build dependencies
RUN apt-get update && apt-get install -y \
    pkg-config \
    libssl-dev \
    libpq-dev \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

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
    echo "=== Looking for backend/server/Cargo.toml ===" && \
    find /tmp/appflowy -maxdepth 3 -name "Cargo.toml" -o -name "backend" -type d | head -10

# Try to find and copy backend code (handle different possible structures)
# AppFlowy might have backend in different locations
RUN if [ -d "/tmp/appflowy/backend" ] && [ -f "/tmp/appflowy/backend/Cargo.toml" ]; then \
        echo "Found backend directory with Cargo.toml" && \
        cp -r /tmp/appflowy/backend /app/backend; \
    elif [ -d "/tmp/appflowy/appflowy-backend" ] && [ -f "/tmp/appflowy/appflowy-backend/Cargo.toml" ]; then \
        echo "Found appflowy-backend directory" && \
        cp -r /tmp/appflowy/appflowy-backend /app/backend; \
    elif [ -d "/tmp/appflowy/server" ] && [ -f "/tmp/appflowy/server/Cargo.toml" ]; then \
        echo "Found server directory" && \
        cp -r /tmp/appflowy/server /app/backend; \
    elif [ -f "/tmp/appflowy/Cargo.toml" ]; then \
        echo "Found Cargo.toml at root, using entire repository as backend" && \
        cp -r /tmp/appflowy /app/backend; \
    else \
        echo "ERROR: Could not find backend with Cargo.toml. Full structure:" && \
        find /tmp/appflowy -type f -name "Cargo.toml" && \
        find /tmp/appflowy -maxdepth 2 -type d && \
        exit 1; \
    fi

WORKDIR /app/backend

# Check if Cargo.toml exists and show structure
RUN echo "=== Backend structure ===" && \
    ls -la && \
    if [ -f "Cargo.toml" ]; then \
        echo "Found Cargo.toml, building..." && \
        cargo build --release; \
    else \
        echo "ERROR: Cargo.toml not found. Contents:" && \
        find . -name "Cargo.toml" && \
        ls -la && \
        exit 1; \
    fi

# Runtime stage
FROM debian:bookworm-slim

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    ca-certificates \
    libssl3 \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy binary from builder
# Note: Binary name may vary - adjust based on actual build output
COPY --from=builder /app/backend/target/release/* /app/

# Create storage directory
RUN mkdir -p /app/storage && chmod 755 /app/storage

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Run the backend
# Note: Adjust command based on actual binary name
CMD ["./appflowy_backend"]

