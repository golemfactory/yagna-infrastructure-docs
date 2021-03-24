# Yagna readme

![CI](https://github.com/golemfactory/yagna/workflows/CI/badge.svg)

An open platform and marketplace for distributed computations.

## Project Layout

* [agent](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/agent/README.md) - basic agent applications based on core services. 
* [core](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/core/README.md) - core services for the open computation marketplace.
* [exe-unit](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/exe-unit/README.md) -  ExeUnit Supervisor.
* [service-bus](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/service-bus/README.md) - portable, rust-oriented service bus for IPC.
* [test-utils](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/test-utils/README.md) - some helpers for testing purposes
* [utils](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/utils/README.md) - trash bin for all other stuff ;\)
* [docs](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/from_golem_repos/docs/README.md) - project documentation including analysis and specifications.

## Public API

Public API rust binding with data model is in [ya-client](https://github.com/golemfactory/ya-client) repo.

## High Level API

Public high-level API for Python is in [yapapi](https://github.com/golemfactory/yapapi) repo.

## Runtimes

We call our runtime **ExeUnit**. As for now we support

* [Light VM](https://github.com/golemfactory/ya-runtime-vm) - [QEMU](https://www.qemu.org/)-based ExeUnit.
* and WASM in two flavours:
  * [wasmtime](https://github.com/golemfactory/ya-runtime-wasi) - [Wasmtime](https://github.com/bytecodealliance/wasmtime)-based ExeUnit.
  * [emscripten](https://github.com/golemfactory/ya-runtime-emscripten) - [SpiderMonkey](https://github.com/servo/rust-mozjs)-based ExeUnit.

Other ExeUnit types are to come \(see below\).

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
   * [ ] Services _\(optional\)_
2. Computational environment \(aka ExeUnit\)
   * [x] **Wasm computation**
   * [x] Light vm-s _\(optional\)_
   * [ ] Docker on Linux _\(optional\)_
   * [ ] SGX on Graphene _\(optional\)_
3. Payment platform
   * [x] **Payments with NGNT**
   * [ ] **Gasless transactions**
   * [x] **ERC20 token**
   * [ ] payment matching _\(optional\)_
4. Transaction system
   * [x] **Usage market**
   * [x] **Pay per task**
   * [ ] Pay for dev _\(optional\)_
5. Network
   * [ ] **P2P** \(Hybrid P2P\) 
   * [ ] **Ability to work behind NAT** \(Relays\)
6. Verification
   * [ ] **Verification by redundancy**
   * [x] **No verification**
   * [ ] Verification by humans _\(optional\)_
7. Back compatibility
   * [ ] Golem Brass/Clay interoperability _\(optional\)_

