# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    # Without this flag, we run into an LLVM bug: https://github.com/rust-lang/rust/issues/99886
    export RUSTFLAGS="-Cllvm-args=-experimental-debug-variable-locations=0" && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/fuzz_multipart_bytes /fuzz_multipart_bytes && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv fuzz/target/release/fuzz_multipart_bytes /fuzz_multipart_bytes_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /fuzz_multipart_bytes /fuzz_multipart_bytes_no_inst /
