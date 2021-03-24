# ya runtime wasi quickstart

[![](https://crates.io/crates/ya-runtime-wasi)](https://crates.io/crates/ya-runtime-wasi) [![](https://docs.rs/ya-runtime-wasi/)](https://docs.rs/ya-runtime-wasi/)

This crate allows you to embed [Yagna](https://github.com/golemfactory/yagna) WASI runtime inside your application.

## Quick start

The usage is pretty straightforward. In your `Cargo.toml`, put `ya-runtime-wasi` as your dependency

```text
# Cargo.toml
[dependencies]
ya-runtime-wasi = "0.2"
```

You can now embed the runtime in your app like so

```rust
use std::path::Path;
use ya_runtime_wasi::*;

// In this example, we assume that `package.zip` contains a WASI binary
// called `hello.wasm`, and maps input/output to `/workdir`
let workspace = Path::new("workspace");
let module_name = "hello.wasm";
let package = Path::new("package.zip");

// Deploy package
deploy(&workspace, &package).unwrap();

// Start the runtime
start(&workspace).unwrap();

// Execute the binary
run(
    &workspace,
    &module_name,
    vec![
        "/workdir/input".to_string(),
        "/workdir/output".to_string(),
    ],
).unwrap();
```

## Examples

A good example of using `ya-runtime-wasi` embedding API can be found in the [`gfaas`](https://github.com/golemfactory/gfaas) crate.

