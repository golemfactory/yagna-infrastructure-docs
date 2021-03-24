# Golem P2P Net

[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/golemfactory/ya-net-p2p/Build)](https://github.com/golemfactory/ya-net-p2p/actions?query=workflow%3ABuild)
![GitHub commit activity](https://img.shields.io/github/commit-activity/w/golemfactory/ya-net-p2p)
![GitHub](https://img.shields.io/github/license/golemfactory/ya-net-p2p)
![net p2p](https://img.shields.io/badge/module-net--p2p-yellowgreen)
[![Pending Specs](https://img.shields.io/github/issues-raw/golemfactory/ya-net-p2p/needs-spec?label=specs%20needed)](https://github.com/golemfactory/ya-net-p2p/labels/needs-spec)


## Description

**net-p2p** is a module that is responsible for network communication and discovery between Golem nodes.

## Key Features

 - ![in progress](https://img.shields.io/badge/impl%20status-in%20progress-darkgreen)
 Network address translation traversal
   - ![GitHub issue #6](https://img.shields.io/github/issues/detail/state/golemfactory/ya-net-p2p/6?style=flat-square)
   Traversal Using Relays around NAT
   - ![GitHub issue #8](https://img.shields.io/github/issues/detail/state/golemfactory/ya-net-p2p/8?style=flat-square)
   NAT hole punching
   - ![GitHub issue #7](https://img.shields.io/github/issues/detail/state/golemfactory/ya-net-p2p/7?style=flat-square)
    Internet Gateway Device Protocol
 - ![pending](https://img.shields.io/badge/impl%20status-pending-red)
 Local Message Boradcasing for distributed KV-Store implementation.
 - ![deferred](https://img.shields.io/badge/impl%20status-deferred-yellow)
 Web borwser compability
    - UDP NAT hole punching with HTTP/3
    - WebRTC compatibility
