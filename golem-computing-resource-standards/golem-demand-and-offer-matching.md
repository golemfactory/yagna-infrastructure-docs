---
description: Problem Statement
---

# Golem Demand&Offer Matching

## Abstract

The purpose of this paper is to summarize analysis on possible approaches to matching Requestors with Providers, which is one of the fundamental capabilities of a Golem ecosystem, provided to Golem Node implementations in the Discovery area of Market API. With a potential multitude of Golem network nodes and variety of computation resources being traded, a pragmatic and efficient matching approach is crucial for seamless network operation. This paper presents a number of potentially feasible approaches and considers their attributes in order to enable informed design and implementation decision making.

## Problem Statement

This article refers to the concepts of Demand, Offer, Demand&Offer Matching relation as specified in [Golem Demand & Offer Specification Language](https://drive.google.com/a/golem.network/open?id=1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY).

### Scope

The Golem ecosystem is primarily a network of nodes functioning in two roles: **Requestors**, which formulate Demand messages and **Providers**, which publish Offer messages. The primary feature of Golem is to enable matching of Demands with Offers so that Agreements for computation resource sharing can be negotiated between specific Requestor-Provider pairs.

#### Golem Network Model

It seems a fair assumption that \(at least initially\) the Providers will outnumber the Requestors in Golem ecosystem. Therefore eg. the number of Offers generated will be larger than the number of Demands.

![](../.gitbook/assets/0%20%285%29.png)

From Requestor’s point of view, the Golem network is expected to **provide an Offer that responds to a Demand** published by this Requestor.

From Provider’s perspective, the Golem network needs to be a **source of relevant Demands** to which this Provider can respond, and thus make profit.

The most trivial approach to this requirement would be to broadcast all messages to all nodes \(ie. each public Demand gets broadcast to all Providers, each public Offer gets broadcast to all Requestors\), so that each node may analyze and respectively act upon an incoming message if it needs to. Unfortunately this is impractical, for following reasons:

* The number of messages circulated in the network grows quickly with the number of nodes in the network.
* For imbalanced networks, with few Requestors \(and Providers competing for work\), a Demand broadcast to all Providers is likely to result in a flood of responding Offers, overwhelming the Requestor in an pseudo-DDoS attack.

Therefore, more sophisticated mechanisms for Requestor-Provider pairing are required.

#### NonExclusive Offers

It must be assumed that Providers will be publishing Offers spanning more capacity than they actually have. The capacity is also dynamic, as in example:

* Provider has eg. 4 cores for “rent”
* Offers published are: 1x4 cores, 2x2 cores, 4x1 core
* If an Agreement is made to purchase 1x4 cores, all remaining Offers get invalidated, whereas Agreement for 1x1 core renders the 1x4 Offer invalid, but the Offer for 1x2 cores is still available.

#### Matching vs Strategic Decision Making

Golem Nodes are autonomous agents in the Golem network. Various strategies of Node’s behaviour on the marketplace are possible, and the decision making \(ie. how to select a potential service Provider? which Demand should be picked up for execution?\) needs to be located in a clearly “interfaced” module - the “brain” of a node. Moreover, strategies for eg. Provider selection may be tightly coupled with a specific Golem Application.

On the other hand, the mechanism for matching Demands with Offers should be as generic as possible. Hence the Requestor/Provider Metamodels propose **separation of strategic decision making from Demand/Offer Matching**:

![](../.gitbook/assets/1%20%285%29.png)

**This article summarizes approaches to matching mechanism**, as provided by Golem network, **which is separated from strategic decision making logic** implemented in Golem Nodes. The Matching serves as a “delivery man” \(hence a nickname: “Market API”\), so that the Demand/Offer selection and optimization decisions are autonomously taken by the Nodes. The “Strategy Control Unit” logic is not considered part of Matching and is **out of scope** of this article \(albeit the implementation of various strategies may have some implications on the required features of the Matching mechanism.\)

### Bidding scenarios

We can consider three main demand/offer exchange scenarios:

* “Tender” - Demand first, Offers second
* “Advertise” - Offers first, Demand second
* “Blind” - Demands and Offers are published independently in parallel

#### “Tender” Model

A Tender means there is a “request for service” broadcast to a set of providers who are invited to take part in bidding.

A general sequence in this model is as follows:

1. Requestor generates a Demand object and publishes it in Golem network.
2. Matching mechanism relays the Demand to Providers who may be concerned to pick up the Demand.
3. Providers decide to respond to the Demand, generate relevant Offers targeted at the Requestor and send them to the requesting party.

Note that in this model, the following features can be attributed:

* Demand - open \(non-targeted or semi-targeted\), potentially long living \(if no relevant Offers appear\)
* Offer - targeted \(at a specific Requestor\), related to specific Demand, short-lived

**Important:** it must be possible to target the Demand to a selected subset of known Providers.

#### “Advertise” Model

In the Advertise scenario, Providers are more active in their search for Demands by publishing open Offers. The hope is some Offers would reach a prospective Requestor, who will pick the Offer.

A general sequence in this model is as follows:

1. Provider generates an open Offer object and publishes it in the Golem network.
2. Matching mechanism allows the Requestor to “browse” relevant Offers.
3. The Requestor selects a preferred Offer published by a specific Provider and:
   1. Formulates a targeted Demand and sends it to that Provider

 In this model, following features can be attributed:

* Offer - open \(non-targeted\), potentially long living
* Demand - targeted \(at a specific Provider\), related to specific Offer, short-lived

Note that in this respect, the Advertise model is symmetric to the Auction model.

**Important:** it must be possible to target an open Offer to a selected subset of known Requestors, but only if they are still present on the market.

#### “Blind” Model

In the Blind model, both Demands and Offers are published to the network independently - in this model, Golem network is an open “marketplace” which attempts to match arriving Demands and Offers.

It may be tempting to consider this model by comparison to eg. a commodity exchange, where Order Book is kept for each traded asset, orders \(bid and ask\) are placed and the exchange mechanisms attempts to match the traders, usually maximizing volume of trade.

**Important:** actual Golem node behaviours may be a mixture of the “pure” models described above. Eg. it is possible for a Requestor to browse through open Offers, and based on that select a subset of Providers, and formulate a “Tender” Demand dispatched to this subset of Providers.

#### Generic Blind Market

The Golem ecosystem can be viewed as a “virtual marketplace” where Demands and Offers are being published and circulate with the purpose of finding “matches”. A ‘Blind’ market in a generic view can be expressed via a graph: All currently open Demands&Offers can be arranged in a graph where edges are defined by Matching relation \(ie. an edge in graph between a Demand D and Offer O exists if D and O are positively matching each other\).

In a classic graph theory view, “finding matching in a graph” is a problem, where one is required to identify a subset of edges in a graph, so that the edges do not have common vertices. Translated to Golem reality this implies that a Demand is assigned with up to one Offer, selected from among all existing Offers which fit the Demand’s requirements.

The classic “matching in graph” approach does not seem to address the practical requirements of a Golem ecosystem, where it may not be expected from the Matching mechanism to find the single best match for each Demand/Offer published into network. Instead, the final selection is made in the Node’s strategic control unit, which may require eg. n best matches for a published Demand, so that it can pick the optimal one according to arbitrary logic.

Therefore instead of exactly one matching Offer for a Demand, the matching mechanism may be required to present eg. n matches, for the node to choose from:

![](../.gitbook/assets/2%20%281%29.png)

**For consideration:** It may be useful to come up with algorithm which allows to identify a “candidate” set of edges in this graph, optimizing some arbitrarily defined metric. With such feature, the strategic control units may be able to execute their behavioral strategies by setting criteria on the matching mechanism - potentially making the execution more efficient. See [Matching Optimization]().

#### Applicability of Order Book concept

Order Book is a concept well known in financial and commodity markets. As summarized in a wikipedia article \([link](https://en.wikipedia.org/wiki/Order_book_%28trading%29)\):

An **order book** is the list of orders \(manual or electronic\) that a **trading venue** \(in particular stock exchanges\) uses to record the interest of buyers and sellers in a **particular financial instrument**. A **matching engine** uses the book to determine which orders can be fulfilled i.e. what trades can be made.

It is debatable whether Order Book concept is strictly applicable to Golem. The main concern is: While it is possible to maintain a single Order Book for trades on a single well defined financial instrument or commodity, the computation services/resources traded by Golem are hardly standardized or well defined. A Golem service is potentially a multidimensional entity, and in that respect it’s hard to imagine “clustering” Demands and Offers into manageable number of Order Books.

#### Potential to introduce Standardized Services

While the [Golem Demand&Offer Specification Language](https://drive.google.com/a/golem.network/open?id=1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY) is broadly generic, it does not by itself provide prescriptive guidelines for definition of specific computation resources. To make Golem ecosystem more understandable and practical, further structuring restrictions can be imposed by a set of recommended, standardized service/computation resource categories.

There are a number of benefits brought by a concept of standardized services:

* Implementing and Order Book-based marketplace becomes possible, as each standard service category may be perceived as a commodity, and all Offers/Demands on this service category can be clustered in one virtual Order Book for matching.
* Querying the market for Offers & Demands in a standard service category simplifies benchmarking the market and observing trends, thus allowing for more sophisticated market strategies.

There are drawbacks as well:

* Proposing and maintaining a standard requires a standardization body, and initially would be responsibility of Golem Factory. It is difficult to imagine a decentralized mechanism for maintenance of the standard service “library”.
* As the space of possible computing resources is very broad, so would be a usable set of standardized services.

A proper Service Standard would need to define the following:

* A set of mandatory properties required to describe a given service category.
  * In that sense, the standard is open, ie. Providers may add/implement additional properties
* The pricing models \(including allowed forms of pricing function\) - thus making Offer comparison potentially easier and possible to automate.
* The Service Standards would need to be multi-dimensional, so that independent, orthogonal aspects of a service can be standardized. Sample dimensions may include:
  * Service class: Batch, Open-ended/interactive
  * Execution environment: VM host, Docker host, etc.
  * Charging/payment model: Time/linear, CPU quants/linear, etc.
  * Execution env. specific categories: eg. VM classes

Golem Service Standard Framework is a potential Building Block which would supplement the Demand&Offer Specification Language in defining the meta-space of computing resources available on Golem network. A draft Framework specification can be found [here](https://drive.google.com/a/golem.network/open?id=1CPGNZBPjNydbFFc-Vul9HeSVYwPTJJ3i8i9-VqUxwDQ).

### Functional Requirements

#### Functional Interface

The interface of the Market API module as part of Requestor/Provider Node implementations is specified [here](https://docs.google.com/document/d/19MtGP6mQrS8BzOfE2jqVHSBPwYJtd96OvOw5PxuG61M/edit#heading=h.eh79a7udofnz).

#### Matching Optimization

The ultimate purpose of the matching mechanism is to enable the selection of optimal Offer for a given Demand. The Requestor/Provider Metamodels assume that final selection is made by the “strategic unit” of a node, and the Market API \(and thus the Matching Mechanism\) is there to provide candidates for selection.

The selection made from amongst randomly selected is not optimal in terms of efficiency, however such model is considered simple and “good enough”.

In subsequent versions of Market API an “optimization function” feature can be considered: to enable the caller to specify the cost function in Market API query, so that Market API optimizes the results according to specifications.

#### Anti-Spam Mechanisms

The Matching mechanism must have some mechanisms which would prevent or discourage nodes from generating excessive traffic \(“spam”\) of Demand/Offer messages.

Cost on node participants needs to be incurred, either “HashCash”, GNT cost or at least a deposit.

How to prevent flooding - \(as in many Requestors flooding one Provider with Demands/Agreements\)?

* Is there potential need to use some sort of “distributed throttling” mechanism for publishing and relaying Demands, Offers and Agreements?

#### Race Conditions

Race Conditions are possible in Golem Network. A Race can be observed eg. when a large number of Requestors are observing the market, on which an extremely attractive offer appears. This attractive offer is likely to trigger a burst of demands, of which most will be turned away.

The Race Conditions are considered a minor problem, and with expected market imbalance \(more Providers than Requestors\) it seems that no dedicated countermeasure is required to stabilize the market. Such a race-prevention mechanism may however be required in the future, as Golem ecosystem evolves.

### Required Characteristics \(Non-Functional Requirements\)

#### Response time

 - _what is the maximum acceptable latency incurred by matching mechanism \(excluding the time spent by nodes on processing incoming objects\)?_

#### Efficiency

- How much network traffic does the solution generate?

- The matching mechanism must be “selective” so that it is capable of picking most appropriate matches from the network without the need for a requesting node to process responses from all potential counterparties.

#### Robustness, resilience to attacks

 - This includes resistance to D/DoS attacks \(eg. prevent a situation where one Requestor’s Demand causes a flood of Providers’ Offers, see ‘Efficiency’\)

#### Transparency

 - The mechanism must provide ability to browse the “marketplace”, see the currently open Offers & Demands

* This includes also ability to view some history of Offers & Demands
* The browse feature must also be ‘selective’ ie. must provide ability to query the “marketplace” using criteria/filters

## 

## Use Case Scenarios

The Matching Mechanism expected behaviour can be described \(and verified\) using a defined set of behaviour scenarios. The scenarios need to cover representative space of situations, where nuances of participating node behaviours can be considered.

### Simple Batch Job Order - “Brass Blender”

This scenario describes a simple batch computation activity, using a standard “docker” hosting mechanism, with linear pricing model and Requestor asking for price estimate based on estimated usage \(equivalent of “task timeout” in Golem Brass\).

1. Providers subscribe to the Market with Offers \(note how the Offers leverage the draft Computing Resource Standard as proposed [here](https://drive.google.com/a/golem.network/open?id=1CPGNZBPjNydbFFc-Vul9HeSVYwPTJJ3i8i9-VqUxwDQ)\):

A\) Offer OA \(with list price presented\):

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Element</b>
      </th>
      <th style="text-align:left"><b>Value</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Service Properties</td>
      <td style="text-align:left">
        <p>golem.std.category=docker</p>
        <p>golem.svc.docker.image:List=[&quot;golemfactory/blender:1.5&quot;] //
          ImageId of Brass Blender</p>
        <p>golem.svc.docker.benchmark{*}</p>
        <p>golem.svc.docker.benchmark{golemfactory/blender:1.5}=5573 // Declared
          benchmark for Brass Blender</p>
        <p>golem.svc.docker.timeout // Support timeout prop</p>
        <p>golem.inf.cores=8</p>
        <p>golem.inf.ram.gb=16</p>
        <p>golem.inf.cpu.platform=x86</p>
        <p>golem.inf.cpu.bit=64</p>
        <p>golem.inf.disk.gb=10</p>
        <p>golem.std.pricemodel=linear</p>
        <p>golem.svc.usagevector:List=[&quot;golem.usage.time.sec&quot;]</p>
        <p>golem.est.price{*} // supports price estimation</p>
        <p>golem.svc.price:List=[123, 0] // list price (prop. to time)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Constraints</td>
      <td style="text-align:left">None</td>
    </tr>
  </tbody>
</table>

B\) Offer OB with price declared as dynamic \(“ask me for price”\)

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Element</b>
      </th>
      <th style="text-align:left"><b>Value</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Service Properties</td>
      <td style="text-align:left">
        <p>golem.std.category=docker</p>
        <p>golem.svc.docker.image:List=[golemfactory/blender:1.5] // ImageId of Brass
          Blender</p>
        <p>golem.svc.docker.benchmark{*}</p>
        <p>golem.svc.docker.benchmark{golemfactory/blender:1.5}=5573 // Declared
          benchmark for Brass Blender</p>
        <p>golem.svc.docker.timeout // Support timeout prop</p>
        <p>golem.inf.cores=8</p>
        <p>golem.inf.ram.gb=16</p>
        <p>golem.inf.cpu.platform=x86</p>
        <p>golem.inf.cpu.bit=64</p>
        <p>golem.inf.disk.gb=10</p>
        <p>golem.std.pricemodel=linear</p>
        <p>golem.svc.usagevector=[golem.usage.time.sec]</p>
        <p>golem.est.price{*} // supports price estimation</p>
        <p>golem.svc.price // price not published</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Constraints</td>
      <td style="text-align:left">None</td>
    </tr>
  </tbody>
</table>

2\) The Requestor subscribes to the Market with following Demand:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Element</b>
      </th>
      <th style="text-align:left"><b>Value</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Requestor Properties</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left">Constraints</td>
      <td style="text-align:left">
        <p>(&amp;</p>
        <p>(golem.std.category=docker)</p>
        <p>(golem.svc.docker.image= golemfactory/blender:1.5)</p>
        <p>(golem.std.pricemodel=linear)</p>
        <p>(golem.svc.usagevector= [golem.usage.time.sec])</p>
        <p>(golem.est.price{[120]}&lt;=125) // calculate amount for estimated time</p>
        <p>(golem.svc.docker.timeout.secs=120) // estimated &#x201C;timeout&#x201D;</p>
        <p>(golem.svc.docker.benchmark {golemfactory/blender:1.5}&gt;=1000)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

3\) In DIscovery phase, the Market API Matching implementation attempts to match Demand-Offer pairs in the network. The matching Offer Proposals start being received on Requestor side \(as results of Collect\(\) calls\).

* Some arriving proposed Offers would have list price indicated - these can be immediately put in order based on their \(linear\) price specification \(estimating the total price can be easily performed by the Requestor’s logic\)
* Some arriving proposed Offers may have some significant properties \(price\) declared but not “valued” - these can be considered for negotiation phase.
* Based on selected strategy, the Requestor node aggregates the arriving Proposals, selecting the ones which seem most promising for negotiation. Note that this would happen “on the fly”, ie. as the new Proposals arrive. The Requestor does not need to wait for all relevant Proposals to start negotiating.

4\) As the set of candidate Proposals grows, the Requestor initiates Negotiation threads with selected candidates. Each negotiation thread begins by issuing a dedicated Demand Proposal to issuers of candidate Offers.

* Note how Demand sample above includes a “request for estimate”, so that the responding Provider calculates the estimated price from submitted estimated usage.
* For Offers where price hasn’t been explicitly specified - their issuers which receive a dedicated Proposal are now expected to respond with dedicated Offer Proposals which include specific price function, and also estimated total price \(as requested via golem.est.usagevector property\)
* The estimated total amount responses \(expressed as golem.est.price property\) can be used by Requestor to validate their “understanding” of the pricing function declared in the Offer.
* The estimated total amount can be used for ordering of the received Offers \(so eg. the cheapest one is selected\)

The Requestor logic must decide at which point it wants to select the “best-so-far” Proposal. Note the Requestor is naturally encouraged to take decision without too much delay, as the best Providers are more likely to become unavailable. Therefore a successful Requestor logic is bound to process the incoming Proposals as they arrive \(as opposed to eg. waiting for fixed amount of time for Proposals before negotiating\).

5\) For the best Provider Proposal received in Negotiation phase, the Requestor selects the one to convert into Agreement. It calls CreateAgreement\(\) and subsequently - ApproveAgreement\(\) waiting for Provider’s response. A positive ApproveAgreement\(\) response from Provider indicates, that the contract has been made, and the Requestor may now issue a call to start the Activity.

6\) The propagation of job input and results happens directly between Requestor and Provider \(note that it can happen using Golem’s network API or other means\).

7\) As soon as the batch run completes on the Provider host, the docker container shuts down, and the Provider node calculates the total usage vector “burned”. It then sends the usage vector and calculated amount due in an Invoice directly to the Requestor node.

**Variants**

* If the Market implementation includes [dynamic property resolution](https://docs.google.com/document/d/1Zny_vfgWV-hcsKS7P-Kdr3Fb0dwfl-6T_cYKVQ9mkNg/edit#heading=h.4nnq84r1f1p7) in Discovery phase, in step 3\) above the golem.svc.price property can be resolved by the Market mechanism before the Proposals are routed to Requestor waiting on Collect\(\) call. The relevant Providers would receive price property value queries, and would be able to declare their price based on Requestor’s identity. The received responses can then be integrated in the Proposal by the Market implementation and returned to Market API client.

### Bootstrapping a Blockchain Network

TODO

* Setup bootstrap nodes
* Instantiate working nodes, giving them bootstrap IP as parameter

### Distributed Database Setup - “Inception”

TODO

* Instantiate “index” nodes \(somewhere in some datacenter???\)
* Instantiate “storage” nodes \(anywhere\)
* Self-sustaining mechanism - “monitor” nodes which are capable of Requesting new nodes when existing ones disappear

### Sophisticated Offer Selection - “Luxury Broker”

This scenario illustrates following circumstances:

* Requestor defines ezoteric, non-trivial computation resource demands which in large part are implemented in the Requestor’s Strategy Control Unit \(we illustrate execution of complex Offer selection optimization logic leveraging a rudimentary Market API\)
* The Offers obtained in low-level matching are non-exclusive \(see [NonExclusive Offers]()\) which means that the optimization of the Offer choice needs to be made in a “quasi-atomic” manner - there is a need to implement some sort of algorithm to actively work around the fact of “non-exclusivity” - in fact the broker needs to become a distributed transaction coordinator.

**Definition:**

The Consumer requires a set of VMs instantiated on Golem Network. Each VM must conform to one of three “classes” \(A, B, C\), each class having different specs. To each class, the Consumer assigns a certain number of “points”:

A - 1 core, 2 GB RAM - 1 point

B - 2 cores, 4 GB RAM - 2 points

C - 4 cores, 8 GB RAM - 5 points

The Consumer expects to receive a set of VMs, so that the total number of “points” attributed to the obtained VMs is equal to 20.

**Notes:**

* Providers with large amount of resources may publish offers for all 3 VMs classes, eg. an 8 core, 12 GB host may publish 4 class A offers, 2 class B offers and 1 class C offer. These offers are non-exclusive, ie. if the single class C offer is picked, all other offers are no longer valid.
* There is no single optimal choice of VMs possible for this criteria. 20 x class A VMs is as good as 4 x class C VMs.

**Sample Implementation 1:**

_\(Should this be described here, or together with respective Approaches?\)_

* The solution assumes a 3rd-party broker implementing the sophisticated offer selection logic. The broker is the Golem Requestor in this scenario.
* The broker is contacted by Consumer outside of the Golem Node implementation - via bespoke protocol, web API, etc.
* The broker executes the demand publishing and processes the offers to meet the criteria defined by the Consumer.
* The broker must use pre-booking to secure the resources and re-validate the offer status - the selection process most likely must be iterative as each round of pre-bookings must be followed by re-sending demands to re-confirm the resource availability.
* Algorithm sketch: \(TODO\)

TBC

## Summary of Possible Approaches

So far following ideas for Matching mechanisms have been briefly considered:

1. Golem Brass-Style Task Relaying - based on Task-scheduling protocol as implemented in Golem Brass. In Golem Brass, the issuing Requestor node iteratively forwards Task requests to subsequent “circles” of Provider nodes, hoping to eventually find relevant candidates, so that it can select the best one \(considering eg. previous Task history, reputation, etc.\)
2. Decentralized, Blockchain-based - a smart-contract is published to which Demands and Offers are sent. The contract performs the Matching relation evaluation and dispatches the matches to respective Golem network nodes.
3. Quasi-decentralized Exchanges - a protocol for Golem Exchange is proposed for third-party vendors. A Golem Exchange would serve as a marketplace/advertising board to which Demands and Offers would be published \(most likely for a fee\). The Exchange may either:
   1. Serve as a plain advert board and Golem nodes would perform the browse and selection actively, or
   2. Perform the matching and dispatch the matched objects to respective Golem nodes.
4. Dedicated distributed protocol - a peer-2-peer network protocol designed to perform efficient matching of Demands and Offers.

## Approach 1 - Golem Brass-Style

### Concept Description

In Golem Brass network, the Tasks/TaskHeaders \(which are similar to the concept of a Demand\) get distributed over the p2p network as an effect of active “pull” from Providers. The Providers actively query their neighbours for known TaskHeaders and from the responses they receive they select jobs they would like to pick up. Once a potential TaskHeader is selected, a MessageWantToComputeTask message is sent directly to the Task’s issuer \(Requestor node\) - this can be perceived as equivalent of a targeted Offer. After sending the “WantToCompute” the willing Provider awaits for a Task confirmation \(equivalent of Agreement proposal\).

This concept rewritten into Demand&Offer model can be expressed as follows:

1. A Demand formulated by Requestor strategic logic is published into Market API and remains stored in Market API module’s Demand list.
2. The Market API module labels the Demands with a “netmask” which specifies the projected “range” of the Demand in the network. The “netmask” is determined based on the estimated network size.
3. A Provider’s strategic logic calls the Market API to subscribe to Demands meeting specific criteria.
4. A Provider node’s Market API module maintains a list of closest network peers \(or rather - expects the Net API module to maintain this list\).
5. A Provider’s Market API module periodically queries closest peers for new Demands they had received. The received Demands are stored in a Demand list.
6. A Provider’s Market API module filters the Demand list to fulfill subscriptions raised by strategic logic. If a new Demand meeting subscription’s criteria is received - it is sent to the upper layers for processing.

**Note: the Demand is selected for processing only if Provider’s address fits within the Demand’s netmask.**

1. When the provider’s Strategy Control Unit decides to bid for the Demand, it formulates a targeted Offer and sends it via Market API to the Demand’s Requestor, hoping for Agreement proposal.
2. The Requestor receives a direct Offer from the Provider and evaluates it. If the Offer is selected for negotiation, the Requestor node initiates the Agreement handshake protocol by formulating an Agreement proposal and sending it to Provider.
3. In case no Offers are received by issuing Requestor node for a certain amount of time, the Requestor decides to “extend range” of the Demand. It re-labels the Demand on its publish list with a broader netmask \(2 x broader range\). The re-labeled Demand is treated as a new entity by querying Providers, and as it propagates through the network, it can be picked up by a larger set of interested Providers.

### Concept Diagram

The key concept of this mechanism is the fact that Demands are propagated through the network an a “pull” basis. Observe a chunk of network as illustrated below:

![](../.gitbook/assets/3%20%283%29.png)

Notice the three highlighted nodes. A Requestor “announces” its Demands to the network - they will be returned when peer nodes call to get new Demands. A nearest Provider peer fetches the new Demands and stores them in its own Demand list. Another Provider further down the network is then able to fetch this Demand which thus gets propagated further. A Provider which receives a Demand which matches its “appetite” will issue a direct targeted Offer to the issuer of the interesting Demand, inviting to initiate Agreement handshake.

### Requirements fulfillment

| Response time |  |
| :--- | :--- |
| Efficiency |  |
| Robustness |  |
| Transparency |  |

## Approach 2 - Decentralised, Blockchain-based

Concept Description

Concept Diagram

Fulfills Requirements

## Approach 3 - Quasi-decentralized Exchanges

Concept Description

Concept Diagram

Fulfills Requirements

## Approach 4 - Dedicated distributed protocol

A hybrid approach consists of parts of Advertise and Tender models.

### Concept Description

We assume the existence of a distributed \(p2p\) storage where all Offers reside. Storage supports at least three operations:

* adding a new Offer
* removing an existing one
* retrieving all Offers \(or a subset based on some criteria\)

1. Providers advertise their Offers through p2p network adding them to Storage.
2. Requestor retrieves all Offers and filters them \(or retrieves with filter\) to find those matching his criteria.
3. Requestor sends his Demand to N Providers which published top Offers \(offer ordering/scoring is up to Requestor\).
4. Providers having resources available respond with an Offer tailored to this specific Demand.
5. Requestor waits for Offers for an arbitrary amount of time. When there is a offer\(s\) that satisfy his needs, makes the final selection and sends Agreement to chosen Providers. Otherwise he need to repeat step 4 with next N Providers.
6. Providers confirm or reject the Agreement.

Notes:

* It is only a good practice to remove Offer if Provider goes offline or decides to permanently discontinue work within Golem Network. We cannot enforce offer removal in such cases. Therefore distributed Offers Storage should have “garbage collector” mechanism.
* Ad 3. Offers ranking can be based on [match strength metric](https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.16i911cb8fa2) or can be any arbitrary mechanism compliant with Requestor policy or business goals.
* Ad 3. Number N must be much greater than actual Requestor computing demand size, because some of the preselected Providers, being offline or busy computing other Demand, may not respond.

### Concept Diagram

![](../.gitbook/assets/4.png)

### Requirements fulfillment

<table>
  <thead>
    <tr>
      <th style="text-align:left">Response time</th>
      <th style="text-align:left">Offer list is immediate. Response from Providers takes an arbitrary amount
        of time - Requestor specifies Demand validity time and waits for Offers
        till it elapses. Provider also specifies Offer validty time and waits for
        the Aggreement till it elapses.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Efficiency</td>
      <td style="text-align:left">
        <p>Offer retrieval (p. 2.) can be weighty, but filtering can lower this cost.
          Simple optimization: retrieving only the new Offers (those which was added/removed
          since the last retrieval).</p>
        <p>Other steps are efficient, there are O(N) messages, where N is number
          of Providers preselected by a Requestor</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Robustness</td>
      <td style="text-align:left">
        <p>Distributed Offer store can be protected from flooding requesting some
          deposits, proof of work and/or introducing a limit of Offers per Provider
          (such limit can be related with reputation score)</p>
        <p>An attacker can target a single Provider sending lots of Demands. This
          requires lots of Requestors being controlled by the attacker.</p>
        <p>Other communication is 1-on-1 so message signing should avoid DDoS.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Transparency</td>
      <td style="text-align:left">
        <p>Offer store is fully transparent, while Demands are private (only preselected
          Providers can see them).</p>
        <p>This can be mitigated by introducing Distributed Demand store.</p>
      </td>
    </tr>
  </tbody>
</table>

### Example Scenarios

#### Dynamic properties - VM with specific network property

**Scenario summary:**

* The Requestor puts constraints on dynamic/parametric properties - to request confirmation of network accessibility from the Provider node to specific URLs.

Scenario sequence:

* Provider publishes an open Offer with following properties:

golem.inf.cores=8  
[golem.inf.ram.gb](http://golem.inf.ram.gb/)=4  
golem.inf.cpu.platform='x86'  
golem.inf.cpu.bit  
cpu.features=\['sse4\_1', 'sse4\_2','aes','avx','avx2'\]  
golem.env.vm.platform=\['HyperV'\]  
golem.contract.payment.scheme=\['pay-as-you-go'\]  
**golem.inet.canConnectTo.tcp\***

**golem.inet.canConnectTo.udp\***

**Note** how the wildcards in property names ensure that any constraint filter referring to these properties with any parameter value will resolve properly \(so that matching relation can be calculated\).

* The Requestor publishes Demand with following requirements:

\(&\(golem.inf.ram.gb&gt;=1\)

 \(golem.inf.cpu.platform=x86\)

 \(vm.platform=\*\)

 \(**golem.inet.canConnectTo.tcp{www.onet.pl:443}=true**\)

\)

* The Market API module scans the market for Offers which match the constraints.
* The filter expression in the Demand is successively reduced by a resolver, so that all expression terms are resolved. After this static resolution, the only remaining terms are those which refer to dynamic/parametric properties.
* The expression terms which refer to parametric property are then forwarded for resolution to the Offer issuer node. The results of dynamic resolution are substituted in the expression.
* The “reductions” continue until no expression left to reduce. If during resolution some dynamic terms are resolved as not relevant, the pending dynamic resolution call to the node is aborted/cancelled.
* The Offers which resolve to ‘true’ \(after all relevant dynamic terms are resolved\) - are forwarded to the Requestor for further processing.

Notes:

* Offers which are more descriptive \(ie. require less dynamic property resolution calls\) will resolve sooner - and therefore will be returned more quickly. This will organically promote efficient Offer compositions.
* For expressions where a number of dynamic parameters require resolution, and some resolution response renders some other term irrelevant - the pending resolution calls are aborted/cancelled.
* Dynamic property resolution is a potential opportunity for frauds \(eg. a Provider can be tricked to do port scanning…\)

#### Hidden properties on both sides

**Scenario summary:**

* Both Requestor and Provider have some properties which they do not want to reveal in public, but which are constrained by other part.

Let’s name those properties for the scenario be more illustrative:

* Requestor do not want Providers from Russia, but Provider is not publishing its country of location
* Provider serves only Requestors with _High_ rating at certain reputation provider, but Requestor doesn’t disclose his reputation score.

**Scenario sequence:**

1. Provider P1 advertise his Offer Of1 \(w/o country\)
2. Requestor R1 retrieves all Offers and finds Of1 weakly matching his criteria
3. R1 sends Demand Dm1 \(with explicit need of country property\) to P1
4. P1 checks all Dm1 criteria and finds he met them all, but Dm1 does not specify R1 rep rating. He prepares a new tailored Offer Of1t that contains country=’PL’ and explicitly states need rep rating at certain provider, and send it to R1
5. R1 checks Of1t once more to confirm compliance. Upon rep rating need R1 prepares tailored Demand Dm1t containing rep=’High’, wraps it within Agreement Ag1 together with Of1t and sends to P1
6. P1 checks Dm1t embedded in Ag1 and confirms the agreement.

## References

* Theoretical aspects of graph matching \([brilliant.org](https://brilliant.org/wiki/matching/)\)
* Order Book \([wikipedia](https://en.wikipedia.org/wiki/Order_book_%28trading%29)\)
* Order Matching Systems \([wikipedia](https://en.wikipedia.org/wiki/Order_matching_system)\)

## Appendix

### Golem Brass Task Protocol Description

1. Once a Task is created in Requestor’s UI, the Task record is stored in TaskManager. Network size is estimated for the purpose of netmasking, and the Task is wrapped in a TaskHeader with a mask limiting the number of included providers.
2. Providers periodically query their direct neighbour nodes for known Tasks and receive their lists of TaskHeaders \(there are limits on list size and number of requests from a single Requestor\). The nodes transmit TaskHeaders they had received directly, as well as TaskHeaders received from neighbours for equivalent query. Tasks have limited time of life.
3. Providers which have computing capacity available, browse their lists of Tasks and attempt to select a Job to execute from the Tasks they support.
4. Once a Task is selected for execution, the Provider node attempts to connect directly with the Requestor and send a MessageWantToComputeTask. The Provider awaits for a certain time period. If no confirmation of Task request is received, or a rejection is received, the tasks is removed from their list of TaskHeaders, placing an additional temporary lock on that Task so that it doesn’t get immediately added again.

