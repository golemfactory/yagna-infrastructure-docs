# The Next Milestone (dubbed internally as YAGNA)

![CI](https://github.com/golemfactory/yagna/workflows/CI/badge.svg)

An open platform and marketplace for distributed computations.

## Project Layout

* [agent](agent) - basic agent applications based on core services. 
* [core](core) - core services for the open computation marketplace.
* [exe-unit](exe-unit) -  ExeUnit Supervisor.
* [service-bus](service-bus) - portable, rust-oriented service bus for IPC.
* [test-utils](test-utils) - some helpers for testing purposes
* [utils](utils) - trash bin for all other stuff ;)
* [docs](docs) - project documentation including analysis and specifications.

## Public API
Public API rust binding with data model is in 
[ya-client](https://github.com/golemfactory/ya-client) repo.

## High Level API
Public high-level API for Python is in 
[yapapi](https://github.com/golemfactory/yapapi) repo.

## Runtimes
We call our runtime **ExeUnit**. As for now we support
 * [Light VM](https://github.com/golemfactory/ya-runtime-vm) - [QEMU](https://www.qemu.org/)\-based ExeUnit.
 * and WASM in two flavours:
   * [wasmtime](https://github.com/golemfactory/ya-runtime-wasi) - [Wasmtime](https://github.com/bytecodealliance/wasmtime)\-based ExeUnit.
   * [emscripten](https://github.com/golemfactory/ya-runtime-emscripten) - [SpiderMonkey](https://github.com/servo/rust-mozjs)\-based ExeUnit.

Other ExeUnit types are to come (see below).

## MVP Requirements

* Clean and easy UX, most specifically during onboarding.
* NGNT-centric.
* Production-ready, modular and easy to maintain architecture and code base.  
_Modular_ means that all the building blocks can be easily replaceable.
* Documentation and SDK for developers.
* Small footprint binaries.

### Functional 

1. Distributed computations
    * [x] **Batching**
    * [ ] Services _(optional)_
1. Computational environment (aka ExeUnit)
   * [x] **Wasm computation**
   * [x] Light vm-s _(optional)_
   * [ ] Docker on Linux _(optional)_
   * [ ] SGX on Graphene _(optional)_
1. Payment platform
    * [x] **Payments with NGNT**
    * [ ] **Gasless transactions**
    * [x] **ERC20 token**
    * [ ] payment matching _(optional)_
1. Transaction system
    * [x] **Usage market**
    * [x] **Pay per task**
    * [ ] Pay for dev _(optional)_
1. Network
    * [ ] **P2P** (Hybrid P2P) 
    * [ ] **Ability to work behind NAT** (Relays)
1. Verification
    * [ ] **Verification by redundancy**
    * [x] **No verification**
    * [ ] Verification by humans _(optional)_
1. Back compatibility
    * [ ] Golem Brass/Clay interoperability _(optional)_

