# Simple Task Management Backend
# Rust API server for task management

FROM rust:1.82-slim as builder

# Install build dependencies
RUN apt-get update && apt-get install -y \
    pkg-config \
    libssl-dev \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Create a simple Rust backend
RUN cargo new --bin task_backend

WORKDIR /app/task_backend

# Create Cargo.toml with dependencies (using older stable versions)
RUN cat > Cargo.toml << 'EOF'
[package]
name = "task_backend"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1.28", features = ["full"] }
axum = "0.6"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tower = "0.4"
tower-http = { version = "0.4", features = ["cors"] }
EOF

# Create main.rs with proper Rust code (compatible with axum 0.6)
RUN cat > src/main.rs << 'EOF'
use axum::{
    routing::get,
    Router,
};
use tower_http::cors::CorsLayer;

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/health", get(health_handler))
        .route("/api/health", get(api_health_handler))
        .layer(CorsLayer::permissive());

    let addr = "0.0.0.0:8000".parse().unwrap();
    println!("Server running on http://{}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn health_handler() -> &'static str {
    "OK"
}

async fn api_health_handler() -> &'static str {
    r#"{"status":"ok"}"#
}
EOF

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
