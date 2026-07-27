# supported versions here: https://hub.docker.com/_/rust
ARG ALPINE_VERSION=3.22

FROM rust:alpine${ALPINE_VERSION} AS builder

ARG REDLIB_SRC=https://github.com/redlib-org/redlib
ARG REDLIB_COMMIT=a4d36e954cf1bd64f209cd8868c5a29edc81b374

RUN apk add --no-cache build-base clang20-libclang cmake git perl

ENV RUSTFLAGS="-C target-feature=-crt-static"

WORKDIR /redlib

RUN git init . \
 && git remote add origin "${REDLIB_SRC}" \
 && git fetch --depth 1 origin "${REDLIB_COMMIT}" \
 && git checkout --detach "${REDLIB_COMMIT}"

RUN cargo build --release --locked --bin redlib
RUN echo "finished building redlib from ${REDLIB_SRC}@${REDLIB_COMMIT}!"

FROM alpine:${ALPINE_VERSION} AS release

RUN apk add --no-cache ca-certificates libgcc

COPY --from=builder /redlib/target/release/redlib /usr/local/bin/redlib

RUN adduser --home /nonexistent --no-create-home --disabled-password redlib
USER redlib

EXPOSE 8080

HEALTHCHECK --interval=1m --timeout=3s CMD wget --spider -q http://localhost:8080/settings || exit 1

CMD ["redlib"]
