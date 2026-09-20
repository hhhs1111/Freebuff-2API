# Freebuff2API Rust 版 — 多阶段构建
# 修复: 原版骨架缓存层会产出 fn main(){} 的空壳二进制(329KB),
#        第二次 cargo build 因 fingerprint 未变而跳过,导致镜像跑起来即静默退出。
#        改为: 只预下载依赖(cargo fetch),真源码就位后 touch 强制重编。
FROM rust:1.95-bookworm AS builder

WORKDIR /build
COPY Cargo.toml Cargo.lock ./
RUN mkdir -p src && echo 'fn main(){}' > src/main.rs && echo '' > src/lib.rs && \
    cargo fetch

COPY src/ src/
RUN touch src/main.rs src/lib.rs && \
    cargo build --release

# 防呆: 空壳(<<2MB)直接构建失败,避免再次产出跑不动的镜像
RUN ls -l target/release/freebuff2api && \
    SIZE=$(stat -c%s target/release/freebuff2api) && \
    echo "binary size = $SIZE bytes" && \
    if [ "$SIZE" -lt 2000000 ]; then echo "ERROR: binary too small, stale build cache"; exit 1; fi

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates curl && rm -rf /var/lib/apt/lists/*
# v0.10.2：容器健康检查（gateway /healthz）
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 CMD curl -sf http://127.0.0.1:47821/healthz >/dev/null || exit 1
COPY --from=builder /build/target/release/freebuff2api /usr/local/bin/freebuff2api
ENV PORT=47821
VOLUME ["/data"]
EXPOSE 47821
ENTRYPOINT ["freebuff2api", "--config", "/data/config.json"]
