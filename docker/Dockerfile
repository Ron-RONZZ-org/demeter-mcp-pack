# syntax=docker/dockerfile:1
# Build both French legal MCP servers for linux/arm64 (OCI instance).
# One image, two binaries; the compose service selects the binary via command.
FROM rust:1.95-bookworm AS build
WORKDIR /src
COPY sources/mcp-legifrance-rs ./sources/mcp-legifrance-rs
COPY sources/mcp-judilibre-rs ./sources/mcp-judilibre-rs
RUN cargo build --release --manifest-path sources/mcp-legifrance-rs/Cargo.toml \
 && cargo build --release --manifest-path sources/mcp-judilibre-rs/Cargo.toml

FROM debian:bookworm-slim
RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates \
 && rm -rf /var/lib/apt/lists/*
COPY --from=build /src/sources/mcp-legifrance-rs/target/release/mcp-legifrance /usr/local/bin/mcp-legifrance
COPY --from=build /src/sources/mcp-judilibre-rs/target/release/mcp-judilibre /usr/local/bin/mcp-judilibre
USER 1000:1000
EXPOSE 6501 6503
ENTRYPOINT ["/usr/local/bin/mcp-legifrance"]
