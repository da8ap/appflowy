# Simple Task Management Backend
# Since AppFlowy doesn't have a separate backend, we'll create a simple REST API
# This can be extended later for full task management features

FROM rust:1.75-slim as builder

# Install build dependencies
RUN apt-get update && apt-get install -y \
    pkg-config \
    libssl-dev \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Create a simple Rust backend for task management
RUN cargo new --bin task_backend && \
    cd task_backend && \
    echo '[package]' > Cargo.toml && \
    echo 'name = "task_backend"' >> Cargo.toml && \
    echo 'version = "0.1.0"' >> Cargo.toml && \
    echo 'edition = "2021"' >> Cargo.toml && \
    echo '' >> Cargo.toml && \
    echo '[dependencies]' >> Cargo.toml && \
    echo 'tokio = { version = "1", features = ["full"] }' >> Cargo.toml && \
    echo 'axum = "0.7"' >> Cargo.toml && \
    echo 'serde = { version = "1.0", features = ["derive"] }' >> Cargo.toml && \
    echo 'serde_json = "1.0"' >> Cargo.toml && \
    echo 'sqlx = { version = "0.7", features = ["runtime-tokio-native-tls", "postgres"] }' >> Cargo.toml && \
    echo 'tower = "0.4"' >> Cargo.toml && \
    echo 'tower-http = { version = "0.5", features = ["cors"] }' >> Cargo.toml

# Create a simple health check server
RUN echo 'use axum::{routing::get, Router};' > task_backend/src/main.rs && \
    echo 'use tower_http::cors::CorsLayer;' >> task_backend/src/main.rs && \
    echo '' >> task_backend/src/main.rs && \
    echo '#[tokio::main]' >> task_backend/src/main.rs && \
    echo 'async fn main() {' >> task_backend/src/main.rs && \
    echo '    let app = Router::new()' >> task_backend/src/main.rs && \
    echo '        .route("/health", get(|| async { "OK" }))' >> task_backend/src/main.rs && \
    echo '        .route("/api/health", get(|| async { r#"{"status":"ok"}"# }))' >> task_backend/src/main.rs && \
    echo '        .layer(CorsLayer::permissive());' >> task_backend/src/main.rs && \
    echo '' >> task_backend/src/main.rs && \
    echo '    let listener = tokio::net::TcpListener::bind("0.0.0.0:8000").await.unwrap();' >> task_backend/src/main.rs && \
    echo '    println!("Server running on http://0.0.0.0:8000");' >> task_backend/src/main.rs && \
    echo '    axum::serve(listener, app).await.unwrap();' >> task_backend/src/main.rs && \
    echo '}' >> task_backend/src/main.rs

WORKDIR /app/task_backend

# Build the backend
RUN cargo build --release

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
COPY --from=builder /app/task_backend/target/release/task_backend /app/task_backend

# Create storage directory
RUN mkdir -p /app/storage && chmod 755 /app/storage

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Run the backend
CMD ["/app/task_backend"]
