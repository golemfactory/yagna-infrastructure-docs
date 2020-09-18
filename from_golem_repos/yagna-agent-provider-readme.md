# Provider Agent

This is a reference Yagna Provider Agent implementation.
Provider Agent is a module which effectively controls the behaviour of
the Provider Node in the Yagna Network. It includes rules and logic for:
* Offer formulation
* Demand validation and evaluation
* Agreement negotiation
* Payment regimes
* Node Resources management
* Activity workflow control
* ExeUnit instantiation and control
* Invoice/Debit Note generation

### Offer formulation

It is rather straightforward and minimal: 
* at most two constrains:
  * requires `golem.srv.comp.expiration` to be set
  * if provided (via env or CLI) sets also `golem.node.debug.subnet`
*  properties:
  * linear pricing (see sample below: 0.01 NGNT/sec + 1.2 NGNT/CPUsec + 1.5 NGNT const)
  * hardware: memory and storage (sample below: 1 gib RAM and 10 gib disk)
  * node name set via env or CLI
  * runtime (sample below: wasmtime)
  
Provider subscribes to the network as many Offers as presets enumerated from CLI. 

#### Sample Offer 
```json
      "properties": {
        "golem": {
          "com": {
            "pricing": {
              "model": "linear",
              "model.linear": {
                "coeffs": [
                  0.01,
                  1.2,
                  1.5
                ]
              }
            },
            "scheme": "payu",
            "scheme.payu": {
              "interval_sec": 6.0
            },
            "usage": {
              "vector": [
                "golem.usage.duration_sec",
                "golem.usage.cpu_sec"
              ]
            }
          },
          "inf": {
            "mem": {
              "gib": 1.0
            },
            "storage": {
              "gib": 10.0
            }
          },
          "node": {
            "id": {
              "name": "__YOUR_NODE_NAME_GOES_HERE__"
            }
          },
          "runtime": {
            "name": "wasmtime",
            "version": "0.1.0",
            "wasm.wasi.version@v": "0.9.0"
          }
        }
      },
      "constraints": "(golem.srv.comp.expiration>0)"
    }
```

### Market Strategy
Current implementation has two naive market strategies:
 * accepting all proposals and agreements
 * accepting limited number of agreements; will reject proposals and agreements above the limit
 
Provider Agent uses (hardcode) the second with limit of 1 agreement.
It will accept all Proposals until first agreement approval.
 
Upon agreement termination (in case of failure, expiration or successful finish)
Provider Agent will start accepting Proposals again until agreement confirmation; and so on.


### Activity
Provider agent allow just one activity per agreement.
On activity finish Provider Agent will initiate Agreement termination.
This is workaround because `terminate_agreement` operation is not supported yet in Market API.

### Payments
Provider agent issues Debit Notes periodically (every `scheme.payu.interval_sec`; `6` in sample above).
It issues Invoice once, after activity (ie. subordinate ExeUnit) finish. 

## Configuration

Provider agent can be used with `.env` file. [Here](https://github.com/golemfactory/yagna/wiki/DotEnv-Configuration) is list of additional environment variables that can be set.

Create separate working dir for the Provider Agent (please create `ya-prov` in the main yagna source code directory), and create `.env` file there by copying
[`.env-template`](https://github.com/golemfactory/yagna/blob/master/.env-template) from yagna repo main directory:
```bash
mkdir ya-prov && cd ya-prov && cp ../.env-template .env
```
and change `NODE_NAME` there.

### Command line parameters

This can be displayed using `cargo run -p ya-provider run --help`

| Parameter      | Description | Env var | 
| -------------- | ----------- | ------- |
| app-key        | Authorization token. |`YAGNA_APPKEY`|
| market-url     | Market api address. |`YAGNA_MARKET_URL`|
| activity-url   | Activity api address. |`YAGNA_ACTIVITY_URL`|
| payment-url    | Payment api address. |`YAGNA_PAYMENT_URL`|
| data-dir       | Path to a directory where configuration files are stored. |`DATA_DIR`| 
| node-name      | Node name to use in agreements. |`NODE_NAME`| 
| subnet         | You can set this value to filter nodes with other identifiers than selected. Useful for test purposes. |`SUBNET`| 
| exe-unit-path  | Path to JSON descriptor file for ExeUnits. |`EXE_UNIT_PATH`|

### Creating app-key authentication token

To obtain `YAGNA_APPKEY` we need to be in this newly created workdir `cd ya-prov`:

1. Run [yagna service](https://github.com/golemfactory/yagna/blob/master/core/serv/README.md):
    ```bash
    cargo run service run
    ```
    If you want to set `debug` log level or higher its good to filter out core crates to `info`:
    ```bash
    RUST_LOG=debug,tokio_core=info,tokio_reactor=info,hyper=info cargo run service run
    ```

2. Create app-key token

    In another console, go to the same directory and run:\
    (it will change your `.env` file with newly created app-key)
    ```bash
    APP_KEY=`cargo run app-key create 'provider-agent'`
    sed -e "s/__GENERATED_APP_KEY__/$APP_KEY/" -i.bckp .env
    ```


## ExeUnits

## WASI (wasmtime)

This is the first ExeUnit we've prepared for you.
You need to clone its repository and build.
In following sections we assume you've cloned it to the same directory where `yagna` is cloned.
```
cd ../..  # assuming you are in ./yagna/ya-prov
git clone git@github.com:golemfactory/ya-runtime-wasi.git
cd ya-runtime-wasi
cargo build
cd ../yagna/ya-prov
```

You also need to build ExeUnit supervisor.
```bash
cargo build -p ya-exe-unit
```

You can list available ExeUnits with command:

```bash
$ cargo run -p ya-provider exe-unit list

Available ExeUnits:

Name:          wasmtime
Version:       0.1.0
Supervisor:    /Users/tworec/git/yagna/target/debug/exe-unit
Runtime:       /Users/tworec/git/ya-runtime-wasi/target/debug/ya-runtime-wasi
Description:   This is just a sample descriptor for wasmtime exeunit used by ya-provider
Properties:
    wasm.wasi.version@v           "0.9.0"
```

## Presets

Provider uses presets to create market offers. On the first run, the Provider Agent will create 
a `default` preset. Presets are saved in a `presets.json` file, located in application's data directory.

You can list presets by running command:
`cargo run -p ya-provider preset list`

The result will be something like this:
```
Available Presets:

Name:               default
ExeUnit:            wasmtime
Pricing model:      linear
Coefficients:
    Duration        0.1 NGNT
    CPU             0.2 NGNT
    Init price        1 NGNT

```

Coefficients describe unit price of ExeUnit metrics:

* Duration - `golem.usage.duration_sec`
* CPU - `golem.usage.cpu_sec`
* Init price - constant price per created activity 

In order to publish an offer based on a preset, that preset needs to be activated first.

### Active presets

To list all active presets, type:

```bash
cargo run -p ya-provider preset active
```

### Creating presets

You can create preset in the interactive mode:

```bash
cargo run -p ya-provider preset create
```

...and non-interactively also:

```bash
cargo run -p ya-provider preset create \
    --no-interactive \
    --preset-name new-preset \
    --exe-unit wasmtime \
    --pricing linear \
    --price Duration=1.2 CPU=3.4 "Init price"=0.2
```

If you don't specify any of price values, it will be defaulted to `0.0`.  


### Updating presets

Note: updating a preset will cancel (unsubscribe) all related offer subscriptions. 

Updating in interactive mode:

```bash
cargo run -p ya-provider preset update new-preset
```

or using command line parameters:

```bash
cargo run -p ya-provider preset update new-preset \
    --no-interactive \
    --exe-unit wasmtime \
    --pricing linear \
    --price Duration=1.3 CPU=3.5 "Init price"=0.3
```

You can omit some parameters and the will be filled with previous values.

### Removing presets

Note: removing a preset will cancel (unsubscribe) all related offer subscriptions.

```bash
cargo run -p ya-provider preset remove new-preset
```

### Activating and deactivating presets

When you activate a preset, a new offer will be published (subscribed) to the marketplace.

```bash
cargo run -p ya-provider preset activate new-preset
```

Note: deactivating a preset will cancel all related offer subscriptions.

```bash
cargo run -p ya-provider preset deactivate new-preset
```

### Listing metrics

You can list available metrics with command:

```bash
$ cargo run -p ya-provider preset list-metrics

Duration       golem.usage.duration_sec
CPU            golem.usage.cpu_sec
```
Left column is name of preset that should be used in commands. On the right side
you can see agreement property, that will be set in usage vector.

## Hardware profiles

Hardware profiles control the maximum amount of hardware resources assigned to computations.
Provider Agent **does not allow you to allocate all of your system resources**. The remaining 
logical CPU cores, memory and storage will be utilized by the background applications,
the operating system and Provider Agent itself.

Note: updating or activating another hardware profile will cancel (unsubscribe) all current offer subscriptions.

The available sub-commands for `profile` are:

```
list        List available profiles
active      Show the name of an active profile
create      Create a new profile
update      Update a profile
remove      Remove an existing profile
activate    Activate a profile
```

### Listing profiles

```bash
cargo run -p ya-provider profile list
```

will print an output similar to:

```
{
  "default": {
    "cpu_threads": 3,
    "mem_gib": 10.9375,
    "storage_gib": 73.57168884277344
  }
}
```

### Display the active profile

```bash
cargo run -p ya-provider profile active
```

will print:

```
"default"
```

### Creating a profile

Usage:
```bash
ya-provider profile create \
    <name> \
    --cpu-threads <cpu-threads> \
    --mem-gib <mem-gib> \
    --storage-gib <storage-gib>
```

E.g.:
```bash
ya-provider profile create half --cpu-threads 2  --mem-gib 8. --storage-gib 256.
```

### Updating a profile

Note: updating a profile will cancel all current offer subscriptions. 

Usage is similar to profile creation.

E.g.:
```bash
ya-provider profile update half --cpu-threads 3  --mem-gib 5. --storage-gib 128.
```

### Removing a profile

Note: removing an active profile will cancel all current offer subscriptions. 

E.g.:
```bash
ya-provider profile remove half
```

### Activating a profile

Note: activating a different profile will cancel all current offer subscriptions.

E.g.:
```bash
ya-provider profile activate some_other_profile
```

## Running the Provider Agent

While the yagna service is still running (and you are in the `ya-prov` directory)
you can now start Provider Agent.

At first, you need to register provider's payment account:
```bash
cargo run payment init ngnt -p
```

Then start the Provider Agent:
```bash
cargo run -p ya-provider run
```

## Mock requestor

Run `ya-requestor` app to mock negotiations, activity and payments.
There is also `gwasm-runner` being prepared as a fully fledged Requestor Agent.
See [PR#47](https://github.com/golemfactory/gwasm-runner/pull/47)

#### 0. Configure Requestor

You need to run a separate yagna service with a different identity,
if you want to run requestor on the same machine. The best way is to create
a separate directory (say `ya-req` in the main yagna
source directory) with a new `.env` copied from `.env-template`. 
In this `.env` file you must change port numbers not to interfere with the provider
and `NODE_NAME` of your choice. Use below script

```bash
mkdir ya-req && cd ya-req && cp ../.env-template .env
sed \
  -e "s|#GSB_URL=tcp://127.0.0.1:7464|GSB_URL=tcp://127.0.0.1:7474|" \
  -e "s|#YAGNA_API_URL=http://127.0.0.1:7465|YAGNA_API_URL=http://127.0.0.1:7475|" \
  -i.bckp .env
```

#### 1. Run yagna service
Start requestor-side yagna service
```
cargo run service run
```

#### 2. Create app-key
1. In a new console but in the same directory (`<repo>/ya-req`) run:
```bash
APP_KEY=`cargo run app-key create 'requestor-agent'`
sed -e "s/__GENERATED_APP_KEY__/$APP_KEY/" -i.bckp .env
```

#### 3. Get some ETH and NGNT
1. We need to acquire funds from faucet on testnet (rinkeby).
This can last a little bit long. Retry if not succeed at first.
```bash
cargo run payment init ngnt -r
```
2. Check if you got credit on your account:
```bash
cargo run payment status
```
Or go to the Rinkeby's etherscan: https://rinkeby.etherscan.io/address/0xdeadbeef00000000000000000000000000000000
(Replace the address with the generated node id for the requestor agent -- a result of `cargo run id show`)

#### 4. Run Requestor Agent
You need `commands.json` file which contains commands to be executed on the provider:

```
cargo run -p ya-requestor -- --exe-script ../exe-unit/examples/commands.json --one-agreement
```

## Central setup
We have centrally deployed (ip: `3.249.139.167`) three independent standalone modules/apps:
 - [net Mk1](https://github.com/golemfactory/yagna/blob/master/docs/net-api/net-mk1-hub.md) @ 3.249.139.167:7464 \
   (can be run locally with `cargo run --release -p ya-sb-router --example ya_sb_router`)
 - [market Mk0](https://github.com/golemfactory/yagna/blob/master/docs/market-api/market-api-mk0-central-exchange.md) @ http://3.249.139.167:8080/market-api/v1/ \
   (can be run locally with `dotnet run --urls "http://0.0.0.0:5001" -p GolemClientMockAPI`)
 - simple "wasm store" @ 3.249.139.167:8000 \
   this is a http server that has two purposes: to serve binary `.zip`/`.yimg` packages (GET) and receive computation results (PUT)
   (can be run locally with `cargo run --release -p ya-exe-unit --example http-get-put -- --root-dir <DIR-WITH-WASM-BINARY-IMAGES>`)

### Yagna binary image
   TODO: describe how to build and pack yagna wasm binary image

   Example can be found in gwasm-runner: [here](https://github.com/golemfactory/gwasm-runner/pull/47/files#diff-bb439e3905abce87b1ff2f3d832f6f0cR83) and [here](https://github.com/golemfactory/gwasm-runner/pull/47/files#diff-bb439e3905abce87b1ff2f3d832f6f0cR130).
