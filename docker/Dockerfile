# syntax=docker/dockerfile:1
# Build both French legal MCP servers for linux/arm64 (OCI instance).
# Cross-compiles on the runner's native platform (fast), then copies the
# aarch64 binaries into an arm64 runtime image.
FROM --platform=$BUILDPLATFORM rust:1.95-bookworm AS build
RUN apt-get update \
 && apt-get install -y --no-install-recommends gcc-aarch64-linux-gnu \
 && rm -rf /var/lib/apt/lists/*
RUN rustup target add aarch64-unknown-linux-gnu
ENV CC_aarch64_unknown_linux_gnu=aarch64-linux-gnu-gcc \
    AR_aarch64_unknown_linux_gnu=aarch64-linux-gnu-ar \
    CARGO_TARGET_AARCH64_UNKNOWN_LINUX_GNU_LINKER=aarch64-linux-gnu-gcc
WORKDIR /src
COPY sources/mcp-legifrance-rs ./sources/mcp-legifrance-rs
COPY sources/mcp-judilibre-rs ./sources/mcp-judilibre-rs
RUN cargo build --release --target aarch64-unknown-linux-gnu \
      --manifest-path sources/mcp-legifrance-rs/Cargo.toml \
 && cargo build --release --target aarch64-unknown-linux-gnu \
      --manifest-path sources/mcp-judilibre-rs/Cargo.toml

FROM --platform=linux/arm64 debian:bookworm-slim
RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates \
 && rm -rf /var/lib/apt/lists/*
COPY --from=build /src/sources/mcp-legifrance-rs/target/aarch64-unknown-linux-gnu/release/mcp-legifrance /usr/local/bin/mcp-legifrance
COPY --from=build /src/sources/mcp-judilibre-rs/target/aarch64-unknown-linux-gnu/release/mcp-judilibre /usr/local/bin/mcp-judilibre
USER 1000:1000
EXPOSE 6501 6503
ENTRYPOINT ["/usr/local/bin/mcp-legifrance"]
