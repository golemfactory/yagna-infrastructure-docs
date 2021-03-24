# ya runtime wasi readme

![Continuous integration](https://github.com/golemfactory/ya-runtime-wasi/workflows/Continuous%20integration/badge.svg)

`ya-runtime-wasi` is a [`Yagna`](https://github.com/golemfactory/yagna) plugin that allows the provider to execute WASI modules in a safe, sandboxed way. Typically, you will use this crate as part of your Yagna provider installation. However, it is also possible to use the integration standalone to execute zipped WASI modules according to some included manifest file.

## Building

Building the plugin is very straightforward:

```text
cargo build
```

If you decide to make some tweaks to the [API](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/crates/api/README.md) and would like to test if everything still behaves as expected, you can trigger included end-to-end integration tests. Make sure you have `wasm32-wasi` target installed \(`rustup target add wasm32-wasi`\) and then run:

```text
cargo test --all --features ya-runtime-wasi/integration-tests
```

Note that running the end-to-end tests requires you to have `wasm32-wasi` target installed.

## Running

### As part of Yagna

This step is explained in [Yagna's general tutorial](https://github.com/golemfactory/yagna/tree/master/agent/provider).

### Standalone

Running standalone is pretty simple. For this, you'll want to use a Wasm module which performs some input from the host, does some computations, and outputs the results to a file on the host. To keep everything simple, we'll assume you use [`rust-wasi-tutorial`](https://github.com/kubkon/rust-wasi-tutorial). Clone the repo, and build the project:

```text
git clone https://github.com/kubkon/rust-wasi-tutorial.git
cd rust-wasi-tutorial
cargo build --release --target wasm32-wasi
```

This will automatically cross-compile your project to `wasm32-wasi` target.

Next, we'll need to create a Yagna package. Go ahead and create new dir called `package` somewhere in your home directory, and copy `rust-wasi-tutorial.wasm` module into it:

```text
mkdir package
cp rust-wasi-tutorial/target/wasm32-wasi/release/main.wasm package/rust-wasi-tutorial.wasm
```

Next, we'll need to create a manifest for the package called `manifest.json`:

```javascript
{
    "id": "rust-wasi-tutorial",
    "name": "rust-wasi-tutorial",
    "entry-points": [
        {
            "id": "rust-wasi-tutorial",
            "wasm-path": "rust-wasi-tutorial.wasm"
        }
    ],
    "mount-points": [
        { "rw": "input" },
        { "rw": "output" }
    ]
}
```

Here, of interest are `entry-points` and `mount-points` entries. The former tell the runtime what modules to load up when we specify some entrypoint \(e.g., `rust-wasi-tutorial` will load up the `rust-wasi-tutorial.wasm` module\), whereas the latter instruct the runtime which directories to preopen and map into our container so that we can make use of it. In this case, we'll map a relative dir `input` as `/input` inside the container and similarly `output` as `/output`.

OK, now we can create the package by zipping the `package` folder:

```text
zip -r rust-wasi-tutorial.zip package/
```

Finally, we'll create a `workspace` dir where we'll mount our package using the runtime:

```text
mkdir workspace
```

We're now ready to deploy the package:

```text
./target/debug/ya-runtime-wasi --task-package rust-wasi-tutorial.zip --workdir workspace deploy
```

Deployment created the mount points on the host for us inside `workspace`. Namely, you should find there `workspace/input` and `workspace/output` among other things. Next, go ahead and create some dummy text file `in` with `Hello WASI!` and put it in `workspace/input/in`. It will then automatically get mapped into `/input/in` for use by our Wasm module.

Next, we need to start the module:

```text
./target/debug/ya-runtime-wasi --task-package rust-wasi-tutorial.zip --workdir workspace start
```

Finally, we can run it:

```text
./target/debug/ya-runtime-wasi --task-package rust-wasi-tutorial.zip --workdir workspace run --entrypoint rust-wasi-tutorial /input/in /output/out
```

If everything went according to plan, you should now find `out` text file with `Hello WASI!` text in it inside `workspace/output/out`.

## License

Licensed under [GPLv3](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/LICENSE/README.md)

