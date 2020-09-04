# Golem Architecture Principles

## Preamble

All efforts around Golem platform architecture should derive from fundamental principles agreed by Golem stakeholders. Principles are general rules and guidelines, intended to be enduring and seldom amended, that inform and support the way in which an organization sets about fulfilling its mission.

## Business Principles

### Principle B0 - User is in control

**Statement:**

User is able to make decisions regarding properties of the resources.

**Implications:**

* The ecosystem allows the user to specify the resources they require. The language allowing to specify needs to cover possibly broad class of resources.
* Golem Network provides mechanisms to “execute” the specified resource requirements \(possibly more than one, see below\).
* User’s freedom of choice applies to:
  * Semantics of resource description - ie. users/participants must be able to add to the resource class set \(eg. by defining new resource types and their properties\)
    * Eg. users/participants may implement trust/reputation mechanisms, expressed via resource description language
  * Ways of “executing” the requirements \(ie. different resource selection/filtering/matching mechanisms must be possible\)
* ...with one exception: payments in the Golem Network are based on GNT.

### Principle B1 - Heterogeneity

**Statement:**

Golem is a platform for sharing of heterogeneous computation resources.

**Implications:**

Implementations of Golem Platform must take into account multiple hardware/OS platforms.

### Principle B2 - Multi-economy

**Statement:**

There can be multiple economies on the Golem platform \(eg. for different price units and different pricing methods of shared resources\).

**Implications:**

There is no practical single universal price unit, which could be a basis for any economy.

### Principle B3 - GNT-centric

**Statement:**

GNT is essential for economy mechanisms published by Golem Factory.

**Implications:**

Reference implementations provided by Golem Factory assume all payments for computation is done in GNT.

### Principle B4 - User-focus

**Statement:**

Golem Platform architecture optimizes for User Experience of participants in Golem ecosystem. The participant roles are:

* Requestor
* Provider
* Integrator
* Customer

**Implications:**

Each participant role requires dedicated focus from UX point of view:

* Requestor: Onboarding, Daemon setup, Application discovery and setup, different Requestor Application models, seamless payments.
* Provider: Onboarding, Daemon setup and configuration, new plugin installation.
* Integrator: Compelling Golem App development model, comprehensive dev & support documentation
* Customer: Relevant Application design patterns

### Principle B5 - Self-sustainable

**Statement:**

In the long-run the Golem Platform does not require Golem Factory to function and evolve.

**Implications:**

The objective of Golem Factory is to design the platform so that:

* It allows self-sustainability via decentralization.
* It provides incentive for all users to participate without “external stimulators” eg. subsidies funded by Golem Factory.
* It has low barrier of entry for the participants \(-&gt; UX for all participant roles\)
* It is compelling for the Integrators to build Golem-based solutions which generate Consumer-Requestor-Provider traffic.
  * The Integrators require tools and guidance \(eg. in a form of “best practices” and “design patterns”\) in order to build the Golem-based solutions.
  * Golem Factory must prepare reference implementation that illustrate a complete Golem Network ecosystem.

## Functional Principles

### Principle F1 - Atomic Nodes

**Statement:**

The smallest functional unit is a node, which must have resources necessary to support the Golem network logic.

### Principle F2 - Meta-language

**Statement:**

The fundamental concepts of matching Requestors and Providers on the Golem Platform can be expressed in a universal, technology-agnostic meta-language.

**Implications:**

It is important to identify the sample resource description criteria in existing, well-defined markets, and understand the matching rules. This is fundamental in order to define the meta-criteria language for Golem platform.

### Principle F3 - Think decentralized

**Statement:**

Golem Platform is a decentralized ecosystem.

**Implications:**

* Where possible, target for decentralized implementations over centralized implementations \(eg. for Service Registry\).
* Assuming an ideal, scalable and inexpensive decentralized "smart contract" system is available, the "marketplace" layer \(at least the "matching", excluding the economy logic\) should be possible to implement using such system.

## Technical Principles

### Principle T1 - Infrastructure stack

**Statement:**

Golem infrastructure is a stack, with network as the bottom layer, and upper layers providing framework for building applications.

**Implications:**

The bottom layer is the network layer. It does not distinguish node roles. It does not interpret data being transported.

### Principle T2 - Versatility

**Statement:**

The Golem core layers are infrastructural components, on top of which different economies may be built, utilizing various "matching" algorithms and leveraging external "trust" sources.

**Implications:**

Golem infrastructure layers \(on top of the network layer\) should allow for implementation of different economies, "trust" models/mechanisms, broad class of applications and "matching" mechanisms.

The resource sharing logic is a layer, on top of which the economic layer is built, and which may function without economy mechanism implementations.

### Principle T3 - Interchangeability

**Statement:**

Individual layer implementations should be interchangeable \(eg. different implementations of network layer.\)

**Implications:**

The node layers must first be specified via interfaces and behaviours, and then implemented as modules.

### Principle T4 - Golem-App boundary

**Statement:**

Golem Factory reference implementation covers the core part of the Node logic. It is up to Integrators to develop applications leveraging the Golem Platform.

**Implications:**

It is important to define which part of the stack belongs to Golem platform itself, and which remains the responsibility of application builders, where is the boundary between Golem components and application components.

## References

* The Open Group Architecture Framework - Architecture Principles \([http://pubs.opengroup.org/architecture/togaf8-doc/arch/chap29.html](http://pubs.opengroup.org/architecture/togaf8-doc/arch/chap29.html)\)

