# Using gftp transfer binary

## Publishing files

Start yagna service:
```bash
cargo run service run
```

Publish a chosen file (blocking).
```bash
cargo run -p gftp -- publish {file name}
```

Example output:
```json
{"result": [{"file": "Cargo.toml", "url": "gftp://0xf2f32374dde7326be2461b4e16a34adb0afe018f/39dc05a25ea97a1c90166658d93786f3302a51b8e31eb9b26001b615dea7e773"}]}
```

or with `--verbose` (`-v`)
```bash
cargo run -p gftp -- publish {file name} -v
```

```json
{"jsonrpc": "2.0", "id": null, "result": [{"file": "Cargo.toml", "url": "gftp://0xf2f32374dde7326be2461b4e16a34adb0afe018f/39dc05a25ea97a1c90166658d93786f3302a51b8e31eb9b26001b615dea7e773"}]}
```

## Downloading a file

```
cargo run -p gftp -- download \
    gftp://0x06bf342e4d1633aac5db38817c2e938e9d6ab7f3/edb0016d9f8bafb54540da34f05a8d510de8114488f23916276bdead05509a53 \
    -o workdir/gftp/download.txt
```

## Uploading a file

Publish file for upload (blocking):

```
cargo run -p gftp -- receive workdir/gftp-upload/License
```

Upload file on provider side:
```
cargo run -p gftp -- upload LICENSE gftp://0x06bf342e4d1633aac5db38817c2e938e9d6ab7f3/z2IeDvgs1Q1hZ6seR0iSEsKW8kxdxQCK0eoz6DsYVznqJIl5K18NqwJPdLgesY9yR
```

## JSON-RPC 2.0 server

To start the application in JSON RPC server mode, type:

```
cargo run -p gftp -- server
```

JSON RPC messages can be sent to application's stdin. **Each JSON object needs to be terminated with a new line**  (`\n`).

### Publish

```json
{"jsonrpc": "2.0", "id": "1", "method": "publish", "params": {"files": ["Cargo.toml"]}}
```

### Download
```json
{"jsonrpc": "2.0", "id": 2, "method": "download", "params": {"url": "gftp://0xf2f32374dde7326be2461b4e16a34adb0afe018f/1d040d4ea83249ec6b8264305365acf3068e095245ea3981de1c4b16782253cc", "output_file": "/home/me/download.bin"}}
```

### AwaitUpload
```json
{"jsonrpc": "2.0", "id": "3", "method": "receive", "params": {"output_file": "/home/me/upload.bin"}}
```

### Upload
```json
{"jsonrpc": "2.0", "id": 4, "method": "upload", "params": {"url": "gftp://0xf2f32374dde7326be2461b4e16a34adb0afe018f/1d040d4ea83249ec6b8264305365acf3068e095245ea3981de1c4b16782253cc", "file": "/etc/passwd"}}
```

## Flags

- `-v`, `--verbose`
    
    Increases output verbosity to match the one in JSON RPC server mode. 
