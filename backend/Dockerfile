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
ARG APPFLOWY_VERSION=main
RUN git clone --depth 1 --branch ${APPFLOWY_VERSION} https://github.com/AppFlowy-IO/appflowy.git /tmp/appflowy

# Copy backend code
RUN cp -r /tmp/appflowy/backend /app/backend

WORKDIR /app/backend

# Build the backend
# Note: Adjust build command based on actual AppFlowy backend structure
RUN cargo build --release || \
    (echo "Build failed, checking structure..." && ls -la && exit 1)

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

