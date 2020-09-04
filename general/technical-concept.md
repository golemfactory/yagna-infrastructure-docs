# Technical concept

## Overview

This document describes a Solution Concept of a Yagna corresponding to requirement scope as indicated here: [Lightweight Golem MVP](https://docs.google.com/document/d/1GZnZ725E_OIRkXzYJNlmafNGDDvR88LFaDpzAmio_nQ/edit).

The purpose of this document is to indicate components of the solution, their dependencies and provide all external references relevant for specification of the components \(eg. API specifications, standards, problem statements for consideration, etc.\)

## Software Development Plan

The SDP shall include following guidelines & recommendations:

* Control of project dependency upgrades \(ie. no spontaneous, non-consulted dependency updates\).
* Versioning approach \(how is versioning controlled, who assigns version numbers\)
  * Versioning matrix \(modules vs versions\)

The draft SDP articles can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/development-plan.md).

The source code repository is [here](https://github.com/golemfactory/yagna)

## Components

![](../.gitbook/assets/0%20%284%29.png)

### Yagna Daemon

Yagna Daemon is a binary package hosted in a separate OS process, preferably hosted as a “daemon” or “service” on a Yagna Node machine. It exposes a set of APIs \(as listed below\) and is the “gateway” into the Yagna Network for other components which host Provider/Requestor/UseCase-specific logic.

![](../.gitbook/assets/1%20%282%29.png)

#### Yagna Service Bus API Implementation

The Yagna Service Bus API is a “communication bus” of the Yagna Daemon. It shall provide capabilities for client processes to “register” as listeners to generic transport channels via Yagna Network. The purpose of this is to provide arbitrary communication channels between Yagna Nodes.

In the first instance this is to be implemented as a library \(eg. Rust crate\) to be embedded in Yagna Daemon application.

The Service Bus API concept description can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/service-bus/ysb-api-concept.md).

#### Yagna Net Implementations

Yagna Net is an abstraction over the bottom, “transport” layer of the Yagna Network. The Yagna Daemon requires at least one YagnaNet implementation module.

**GolemNet Mk1 - Centralized Hub**

The first, simplistic implementation of YagnaNet, based on a centralized communication “hub” which can be instantiated on the network \(eg. Internet\) so that it is easily accessible to connecting Yagna Nodes.

This is to serve as the first, PoC implementation of YagnaNet, which should be fairly straightforward to build and thus should allow for integration testing with other Yagna modules relatively quickly.

The REST API of the YagnaNet Hub is specified [here](https://github.com/golemfactory/yagna/blob/master/docs/net-api/net-mk1-hub-openapi.yaml) \(waiting to be merged\).

**YagnaNet Mk2 - Yagna Implementation**

This component is an implementation of P2P network “transport” layer.

#### Market API Implementations

Yagna Daemon must include at least one implementation of the [\[BB04\] Golem Market API](https://docs.google.com/document/d/1Zny_vfgWV-hcsKS7P-Kdr3Fb0dwfl-6T_cYKVQ9mkNg/edit?usp=sharing).

API specification can be found in [market-api.yaml](https://github.com/golemfactory/ya-client/blob/master/specs/market-api.yaml).

**Market API Mk0 - Centralized Market Exchange**

Simplistic temporary stage when Requestor/Provider Agent applications connect to a centralized implementation of the Market API using a central Market Exchange server, which supports the Demand/Offer matching protocol.

This is to serve as the first, PoC implementation of Market API, which should be fairly straightforward to build and enable integration testing of Requestor-Provider Market interaction patterns.

The Market API Mk0 concept description can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/market-api/market-api-mk0-central-exchange.md).

**Market API Mk1 - Simplistic Distributed Market Implementation**

Implementation dedicated for relatively small P2P networks, with distributed Demand/Offer matching protocol, based on a simplistic broadcast of Demand/Offer artifacts over the whole network.

The Market API Mk1 concept description can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/market-api/market-api-mk1-broadcast-exchange.md).

**Market API Mk2 - Distributed Market Implementation**

Implementation dedicated for P2P networks, with optimized distributed Demand/Offer matching protocol implementing:

* Localcast distribution of Demand/Offer artifacts
* Proof of work to prevent spamming with Demands&Offers

The Market API Mk2 concept description can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/market-api/market-api-mk2-decentralized-matching.md).

#### Activity API Implementations

The Yagna Daemon must include at least on implementation of the [\[BB05\] Yagna Activity API](https://docs.google.com/document/d/1BXaN32ediXdBHljEApmznSfbuudTU8TmvOmHKl0gmQM/edit?usp=sharing) to provide control capability over the Activities executed by ExeUnits on Provider Node.

API specification can be found in [activity-api.yaml](https://github.com/golemfactory/ya-client/blob/master/specs/activity-api.yaml) \(draft\).

**Activity API - “Straight-thru” Implementation**

Default Activity API Implementation based on open-text transmission of messages between Requestor Client Application, Yagna Provider Agent and ExeUnit.

#### ExeUnit Implementations

Yagna Provider software depends on implementations of ExeUnits. An ExeUnit is an abstraction over an execution environment which can be integrated with a Yagna Provider Agent. A description of an ExeUnit responsibilities is provided in the [\[BB05\] Golem Activity API](https://docs.google.com/document/d/1BXaN32ediXdBHljEApmznSfbuudTU8TmvOmHKl0gmQM/edit?usp=sharing).

The ExeUnit Design concept description can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/activity-api/exe-unit/exe-unit-design.md).

**Dummy ExeUnit**

The initial implementation of an ExeUnit for Yagna shall be a mockup, which is expected to emulate the interaction between Provider Agent, Yagna Daemon and a generic ExeUnit. The purpose of this is to quickly build a sample for integration testing of the Provider infrastructure.

The Dummy ExeUnit concept description can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/activity-api/exe-unit/exe-unit-dummy-impl.md).

**WASM ExeUnit**

The first “real” ExeUnit to be developed for Yagna shall be an execution environment capable of running arbitrary WebAssembly modules.

The WASM ExeUnit concept description can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/activity-api/exe-unit/exe-unit-wasm.md).

#### Payment API Implementations

The Yagna Daemon must include at least one implementation of the [\[BB08\] Golem Payment API](https://docs.google.com/document/d/1GsbMGEduO1toTXW4oxitgeTCSfAI-ZL0fklGBr3LnT4/edit?usp=sharing)

API specification can be found in [payment-api.yaml](https://github.com/golemfactory/ya-client/blob/master/specs/payment-api.yaml).

**Payment API - Dummy Implementation**

A dummy, initial implementation of the Payment API, designed to enable early integration testing of Yagna Daemon. It shall not provide any “real” payment capabilities.

**Payment API - Simple Ethereum Implementation**

Implementation of the Payment API dedicated for the Yagna, based on “plain old” Ethereum platform, and including features indicated as required for the MVP.

#### Identity API Implementations

The Identity API Implementation shall include features required to perform Yagna Node onboarding, authorization, node identity & key management, etc.

### Yagna CLI

The control over the Daemon configuration is available via a Command-Line Interface \(CLI\).

### Yagna Provider Agent

A reference Yagna Provider Agent is a module/process which effectively controls the behaviour of the Provider Node in the Yagna Network. It includes rules and logic for:

* Offer formulation
* Demand validation and evaluation
* Agreement negotiation
* Payment regimes
* Node Resources management
* Activity workflow control
* ExeUnit instantiation and control
* Invoice/Debit Note generation

Concept Description can be found [here](https://github.com/golemfactory/yagna/blob/master/docs/provider-agent/provider-agent.md) \(scratchpad\)

### Requestor Applications

Yagna shall include Requestor-side applications to illustrate Yagna API Interaction patterns, as well as highlight the responsibilities of the Requestor-side client.

The Requestor-side responsibilities shall be modelled using abstractions, and a number of demonstrative implementations shall be provided. It is intended to subsequently add-on to the set of implementations, in order to provide a reusable set of components which can be assembled to form a powerful Yagna App.

#### Responsibility

The Requestor applications must consider and address following responsibility areas:

* Commercial Model & Parameters
  * Default commercial parameters \(eg. how much GNT do we want to charge for generic resource?\)
* Identity Management
  * Onboarding
  * Authorisation
* Demand formulation
* Offer rating
  * Provider reputation
* Agreement negotiation
  * Handle different Payment Regimes \(pay one off/”per task”, pay-as-you-go/”usage”\)
  * Handle different Pricing Functions
* Activity control
  * Create/Destroy
  * Start/Stop
  * Transfer In/Out
* Invoicing & Payment
  * Invoice acceptance strategy

#### Requestor Agent App

The most basic, yet functional Requestor-side Application. As we are focusing on WebAssembly module execution in the first instance, the “Requestor Agent App” may simply provide capability of running a single WebAssembly module on Yagna Network, based on trivial Demand formulation and negotiation logic, and following a single, basic Payment regime.

#### Simple General UI App

TODO discuss and provide description

### Golem Unlimited Integration

#### Golem Unlimited Provider Agent

TODO: Describe this as a gateway/adapter allowing to publish GU resources into the Yagna Network

#### Golem Unlimited Requestor Client

TODO: Describe this as a GU plugin which enables dispatch of GU tasks into Yagna Network \(on commercial basis\)

### Yagna API TestBed

In order to enable early integration testing of all the API protocols, a centralized test server emulates Market Protocol. It offers full functionality of Market API \(including Demand & Offer matching\) and can be used until a proper Distributed Market Protocol is implemented.

## Requirements Traceability Matrix

A Requirements Traceability Matrix is maintained [here](https://docs.google.com/spreadsheets/d/1gVHIjgedvOIvBdzAqRgSz5H3pFYqJ-TH5JSIV3of4bU/edit?usp=sharing) to ensure proper coverage of requirements by the solution components is achieved.

## Security Aspects

### Assumptions

A number of assumptions are made as foundations of Yagna security model.

#### REST APIs

* It is supported by Yagna Daemon to allow REST API clients calling from different network locations \(ie. not from localhost\)
* The management of encryption keys is embedded in Yagna Daemon, behind REST APIs. No access to key management features via REST API.
* Access to REST APIs must be authorized to bind the calling application with Golem identity it is using.

#### GSB

* It is assumed that GSB is contacted only from the same host machine \(ie. this version of Yagna does not support GSB clients connecting from different network locations, only from localhost\)
* Any process registering in GSB \(a GSB client\) must have full permissions on machine \(ie. can debug other processes, and other GSB clients\)
* To prevent the non-permissioned processes from contacting GSB - the GSB implementation in Yagna Linux is to be implemented using filesystem mechanisms \(with ACLs\)
  * For other OS platforms - equivalent communication mechanisms would be used.

### Threat Modeling

**Task:** List threats on each level of the solution \(building blocks, aggregates, modules\).

### Threat Mitigation

Select Threats which create highest risk and provide methods of mitigation to be implemented in Yagna.

### Design & Implementation Patterns

## Implementation Task Breakdown

A breakdown of implementation tasks and their dependencies is required to efficiently allocate and schedule the build effort.

This task breakdown describes the scope of **1st edition of Yagna**.

\(Effort: Low - 1-3 days; Mid - 4-10 days; High - 11+ days\)

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Task</b>
      </th>
      <th style="text-align:left"><b>Description</b>
      </th>
      <th style="text-align:left"><b>Effort</b>
      </th>
      <th style="text-align:left"><b>Comments</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">LTG0.1</td>
      <td style="text-align:left">
        <p>Agree general development guidelines (&#x201C;SDP&#x201D;):</p>
        <ul>
          <li>Toolsets, libraries</li>
          <li>Conventions?</li>
          <li>Dev Lifecycle - code reviews? Unit testing?</li>
          <li>Quality Assurance approach?</li>
          <li>GitLab migration???</li>
          <li>Continuous Integration framework</li>
          <li>Continuous Delivery framework</li>
        </ul>
      </td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG0.2</td>
      <td style="text-align:left">
        <p>Establish development environment foundations:</p>
        <ul>
          <li>Code repos</li>
          <li>AutoBuild infrastructure</li>
        </ul>
      </td>
      <td style="text-align:left">Low/Mid</td>
      <td style="text-align:left">Mid if migration to GitLab is decided</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">QA Strategy</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">Yagna Daemon Integration Round 1</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG1.1</td>
      <td style="text-align:left">Design the GSB implementation</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG1.2</td>
      <td style="text-align:left">Develop first iteration of GSB Implementation</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG2.1</td>
      <td style="text-align:left">Finish design of Yagna Net API</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG2.2</td>
      <td style="text-align:left">Design concept of YagnaNet mk1 - based on centralized routing hub, can
        be published in internet, etc.</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG2.3</td>
      <td style="text-align:left">Implement YagnaNet mk1</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG2.4</td>
      <td style="text-align:left">Design concept of YagnaNet mk2 - full P2P, consider 3rd party libs, etc.</td>
      <td
      style="text-align:left">High</td>
        <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG2.5</td>
      <td style="text-align:left">Implement YagnaNet mk2</td>
      <td style="text-align:left">High</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG3.1</td>
      <td style="text-align:left">
        <p>Design the ExeUnit:</p>
        <p>- &#x201C;interface&#x201D; with other parties (define messages, specify
          their relationship with Activity API)</p>
      </td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG3.2</td>
      <td style="text-align:left">Develop a dummy ExeUnit to provide first ExeUnit interface implementation
        for integration testing</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG3.3</td>
      <td style="text-align:left">
        <p>Design WASM ExeUnit</p>
        <ul>
          <li>Decide runtime for first impl</li>
          <li>Design impl of ExeScript Commands</li>
          <li>Choose the protocols to be supported for TRANSFER</li>
        </ul>
      </td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG3.4</td>
      <td style="text-align:left">Develop WASM ExeUnit Impl, along the concept</td>
      <td style="text-align:left">Mid/High</td>
      <td style="text-align:left">Depends on WASM runtime chosen, or protocols supported by TRANSFER</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG4.1</td>
      <td style="text-align:left">Design Market API Impl mk0 - centralized marketplace as a service, eg.
        available in internet</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left">This must include stabilization of market-api.yaml</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG4.2</td>
      <td style="text-align:left">Implement Market API Impl mk0 as module/crate</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG4.3</td>
      <td style="text-align:left">Implement the centralized Market &#x201C;exchange&#x201D; module as a
        webservice</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left">...not necessarily in Rust ;)</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG4.4</td>
      <td style="text-align:left">Design Market API Impl mk1 - decentralized naive marketplace</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG4.5</td>
      <td style="text-align:left">Implement Market API Impl mk1 as module/crate</td>
      <td style="text-align:left">High</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG5.1</td>
      <td style="text-align:left">Design Activity API basic implementation</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left">This must include stabilization of activity-api.yaml</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG5.2</td>
      <td style="text-align:left">Implement Activity API basic implementation</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left">Depends on ExeUnit interface and dummy ExeUnit, SRAPI Impl</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG6.1</td>
      <td style="text-align:left">Finish design of Payment API specs</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left">Must include payment-api.yaml</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG6.2</td>
      <td style="text-align:left">Implement Dummy Payment API for integration testing</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG6.3</td>
      <td style="text-align:left">Design Payment API Impl on basic Ethereum</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG6.4</td>
      <td style="text-align:left">Implement Payment API on basic Ethereum</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG7.0</td>
      <td style="text-align:left">Develop Dummy Provider App</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left">Create a dummy Provider application to emulate basic scenario for integration
        testing</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG7.1</td>
      <td style="text-align:left">Design Provider Agent mk1</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left">Only identify fundamental segments and build simplistic implementations
        of eg. Offer formulation, etc</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG7.2</td>
      <td style="text-align:left">Develop Provider Agent mk1</td>
      <td style="text-align:left">High</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">LTG8.0</td>
      <td style="text-align:left">Develop Dummy Requestor App</td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left">Create a dummy Requestor application to emulate basic scenario for integration
        testing</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG8.1</td>
      <td style="text-align:left">Design Simpl<b>er</b> Requestor App</td>
      <td style="text-align:left">Mid</td>
      <td style="text-align:left">Almost like a fatter gwasm-runner, with MarketAPI and PaymentAPI handling</td>
    </tr>
    <tr>
      <td style="text-align:left">LTG8.2</td>
      <td style="text-align:left">Develop Simpl<b>er</b> Requestor App</td>
      <td style="text-align:left">High?</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>



