# Golem Market API

Golem Market API

Specification

Version 1.5

## Abstract

The purpose of this paper is to define a logical API for market operations in Golem Network node implementations. It does not specify the actual implementation of any Golem market protocol, but rather indicates an interface which must be implemented by such implementations. All Golem-compliant Market mechanism implementations are expected to conform to the protocols and interaction sequences described in this article.

## Scope

The Market API is the entry to Golem’s market on which [Demands](https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.v1l0ybcz71rq) and [Offers](https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.jq90wu9f371b) circulate, and Requestors find optimal Providers for the computing tasks they want to achieve. The interaction with the Market then proceeds to direct interaction of the Requestor with potential Providers, and offer negotiation is possible. After a successful negotiation, an [Agreement](https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.rm34421t0322) approval takes place. Final artifact of the Market API is the Agreement approved and signed by both sides.

The Market API also enables the Golem Nodes to “observe” the market - scan the circulating Demands and Offers, which may give insight into market trends and influence the Node’s market strategy.

**Note:** The Observation capability only reveals the Demands and Offers which have been published to the market \(ie. “open” Demands & Offers\). The communication in Negotiation and Agreement phases \(see below\) is peer-to-peer and thus remains hidden from the public.

![](../.gitbook/assets/0%20%281%29.png)

## Overview

The Market API has two interfaces, one on Requestor side and one on Provider side. It supports following Requestor-Market-Provider interactions:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Phase</b>
      </th>
      <th style="text-align:left"><b>Requestor</b>
      </th>
      <th style="text-align:left"><b>Provider</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><a href>Observation</a>
      </td>
      <td style="text-align:left">Each Node is able to scan the market for circulating Demands and Offers
        meeting given filter criteria.</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href>Discovery</a>
      </td>
      <td style="text-align:left">Requestor publishes a Demand on the market, expecting to receive Proposals
        of potential <a href="https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.jzr5wr9i4uh5">matching</a> Offers.</td>
      <td
      style="text-align:left">Provider publishes Offers symmetrically.</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href>Negotiation</a>
      </td>
      <td style="text-align:left">From the initial set of received Offer <a href>Proposals</a>, the Requestor
        selects Provider candidates, and begins <b>direct</b> interaction with them
        by sending the Proposals, with Demand potentially adjusted to the receiving
        Provider&#x2019;s Offer (as, an Offer may eg. place some <a href="https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.8a2umgq9ob4s">constraints</a> on
        Requestor&#x2019;s <a href="https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.r6oufnsb6bu6">properties</a>,
        so the bespoke Demand should take this into account, etc.)</td>
      <td style="text-align:left">
        <p>Provider listens for Demand <a href>Proposals</a>, to which it may respond
          with adjusted Offer (e.g. it is possible to provide a better price for
          a known Requestor, etc.)</p>
        <p>The Proposal exchange may be iterative.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href>Agreement</a>
      </td>
      <td style="text-align:left">
        <p>In a successful negotiation scenario Requestor receives a Proposal with
          Offer which satisfies all the requirements and expectations.</p>
        <p>The Requestor promotes the Proposal into Agreement and sends it to Provider
          which issued the selected Offer.</p>
        <p>When the Provider responds with the Agreement approval, the Requestor
          approves it as well, and Agreement is considered in force.</p>
      </td>
      <td style="text-align:left">
        <p>At some point, the Provider receives the Agreement from the Requestor.
          If the Agreement is still acceptable for the Provider - an approval message
          is sent. At this point, the Provider may place an early &#x201C;lock&#x201D;
          on its resources to ring-fence them, but would not yet commit/trigger the
          activity.</p>
        <p>When the Provider approval is matched by the Requestor approval, the Agreement
          is considered approved and in force.</p>
      </td>
    </tr>
  </tbody>
</table>

#### Clock Synchronization

Market API Implementations must depend on synchronized clock mechanism. This is a fundamental assumption, as the artifacts used in API Implementations refer to absolute time source.

### Market Observation

#### Sequence Diagram

![](../.gitbook/assets/1%20%284%29.png)

#### Shallow Scan vs Deep Scan

The Observation functionality of the Market API is provided by the **Scan** family of operations, which is meant to allow the Golem Node logic to perform queries on the Demands & Offers currently circulating on the market. The [**BeginScan**]() operation initiates a scanning activity by accepting filtering criteria and returning Scan Id. The results are aggregated by the Market API implementation and can be collected by calling [**CollectScanResults**](). Finally, a scanning activity is closed by a call to [**EndScan**]().

Depending on the implementation of the Market mechanisms, the **Scan** operations may require a variable amount of effort/resources depending on the scan “range”. Therefore two grades of **Scan** implementation can be considered:

* A “shallow”/”passive” scan is a more efficient/lightweight implementation of the market observation feature. This comes at the cost of results being potentially incomplete. For example, in a market implementation where Demands/Offers are distributed in a decentralized network and maintained in a DHT, the shallow scan may require only viewing the Node’s own DHT Table, or at most - the closest neighbours. This form of market observation is expected to be “cheap”. **Note:** This grade is part of Market API [Capability Level]() 2.
* A “deep”/”active” scan implementation requires dedicated search/filter capability that is expected to allow querying the whole network for results. Naturally, for some market implementations, this may be time and effort consuming and would require a dedicated protocol. The benefit of the deep scan is - the results are complete, i.e. regardless of how small a set of matching results is, and how “far” it is in the network - the deep scan is expected to locate and return this set. **Note:** This grade is part of Market API [Capability Level]() 3.

### Market Interaction

#### Sequence Diagram

![](../.gitbook/assets/2%20%282%29.png)

#### Discovery Phase - Demand&Offer Matching

The purpose of the Discovery Phase is to perform the actual “matching” of Offers with Demands as they are circulated on the market. Parties subscribe to the network with their Demands and Offers respectively. The outcome of the Discovery Phase is a collection of Proposals with matching Demand-Offer pairs.

While many implementations of [matching protocol](https://drive.google.com/a/golem.network/open?id=1yTupuRsN9DKVrK1TPhM6dBxKCAPk0wCB8KxRf57ZkV4) are possible, the interaction with them shall follow the sequence and operation specifications defined in this paper.

The Discovery Phase is the only one which involves indirect \(eg. via P2P network\) communication. Further phases are direct \(one-to-one\).

**Proposal**

Proposal is a logical “vehicle” which binds matching Demand&Offer for the purposes of communication between Market and Nodes. From the Market API point of view it is a Demand-Offer pair, with Id and a status.

An actual Market API implementation should avoid sending both Demand and Offers artifact content back and forth. It is probably more efficient for a Requestor node to receive Proposal messages containing an Offer and a reference to their Demand \(a Demand Id\), as it is expected from the Requestor to know the Demands it had issued. Likewise, a Proposal message sent to Provider should include the content of Demand, and a reference to Offer. Signatures of the CounterProposal operation suggests what should be sent.

#### Negotiation Phase - Dynamic Property Resolution

The Specification Language for Demands & Offers allows for special scenarios of property specification \(as described [here](https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.rk3omg1xebi3)\). The bottom-line is: the Requestor may receive Offers from the market, which don’t have all the requested properties explicitly stated \(price being a prominent example\). The resolution of “value-less”/”dynamic” properties, as well as “pseudo-functions” requires direct interaction between the Requestor and Provider, in order to fulfill all the constraints specified in each side’s Demand/Offer.

This interaction is primarily expected to happen during the exchange of Proposals in the Negotiation Phase:

* On the Requestor side, the Strategy Unit processes the received Proposal \(Offer and demand Id\). Knowing constraints declared in the Offer it is able to supplement the Demand with the requested properties \(unless these properties depend on “dynamic” properties of the Offer which haven’t yet been indicated by the Provider\). Therefore a “bespoke” Demand is formulated by the Requestor Strategy Unit and submitted **directly** to the Provider as a Proposal.
* On the Provider side, a Proposal \(Demand and offer Id\) reception triggers a negotiation process, in which Demand’s declared constraints can be satisfied by supplementing the Offer with requested property values \(same as above\). The Provider’s Strategy Unit formulates the “counter”-Proposal \(“bespoke” Offer and demand Id\) and responds **directly** to the Requestor.

At least one iteration of Proposal exchange is required. The purpose of this is the following:

* The Provider has opportunity to adjust the price to the Requestor \(even if the price originally expressed in the “Open” Offer was different\)
* The Requestor has an opportunity to validate the understanding of received price specification \(by submitting a Proposal with estimated usage counters and expecting the calculated cost to be returned by Provider - this can be then compared with Requestor’s own calculation and thus the pricing specification is validated\).

The negotiation may require multiple iterations, however, each of the iterations is expected to “converge” towards an explicit Proposal \(Demand-Offer pair\). The negotiating nodes may implement autonomous rules of negotiation. For example, if a subsequent iteration returns Demand or Offer which is identical to the one received in the previous iteration **the receiving node may decide to break the negotiation phase**. However, it is expected that sensible negotiation strategy implementations shall include “stop” criteria, as it seems counter-productive to repeat the Proposal exchange for too long.

**Proposal Lifecycle**

![](../.gitbook/assets/3%20%281%29.png)

**Dynamic Property Resolution in Discovery Phase**

This is part of [Capability level]() 2. Some market implementations may allow for a more efficient resolution of dynamic Demand/Offer properties ー this may happen already in the Discovery phase.

The Matching mechanism, when resolving the match relation for the specific Demand-Offer pair, is to detect the “dynamic” properties required \(via constraints\) by the other side. At this point, it is able to query the issuing node for those properties and submit the other side’s requested properties as the **context** of the query. For example:

* A Demand D\(props:\[p2=\*, p3=x\], constr:\[p1&gt;0\]\) is tested against an Offer O\(props:\[p1=\*\], constr:\[p3=x\]\). Demand D has a constraint on a specific property p1, while Offer O only declares this property as supported \(p1=\*\), but does not specify its value. The Offer O specifies constraints on Demand’s property p3, which is in turn declared openly in D.
* At this point the matching mechanism would query the issuer of the Offer O for the value of O.p1, submitting the value of D.p3 as query context.

**Important:** Note that the Requestor must be able to present the query context \(D.p3 in this case\) based on public properties of the Offer. If that is not possible \(ie. Requestor will not trust the Offer issuer based on public properties only\) - the match will not succeed.

* The Provider of O is able to respond to this query with a specific value of O.p1, which has been calculated taking into account the D.p3 received as a query **context**.
* The matching mechanism would supplement the Offer O with resolved dynamic properties and \(if matching constraints are met\) send supplemented Offer \(in the form of a Proposal\) to the Requestor.

With such a mechanism, the Requestor should receive an Offer which has been resolved for that particular Demand. Therefore the subsequent Negotiation phase could be reduced to a single Proposal exchange.

![](../.gitbook/assets/4%20%282%29.png)

In the Market API, the property value queries are routed by the Market implementation to the relevant node and are delivered to the client of the Market API via Collect\(\) method. A call to Collect\(\) may return PropertyQuery messages, which include:

* Query Id
* Id of the potential counter-party node
* Query context \(and relevant properties of that party’s Demand/Offer\)
* List of requested property names

Implementation of “dynamic” property resolution in the Discovery phase increases the complexity of Market API implementation, therefore, this mechanism is classified as covered by Market API [Capability Level]() 2.

**Property Aspects**

As indicated in the [Demand&Offer Specification Language](https://docs.google.com/document/d/1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY/edit#heading=h.xljbllnqnd9i), the constraint filters may refer to property aspects. These are additional metadata of the properties, which however are not directly sourced from the incoming Demand/Offer, but rather are derived by the processing engine \(eg. based on Demand/Offer issuer’s identity, message attachments, etc.\)

The Property Aspect handling must be implemented via a scalable pattern of plugins, ie. new aspects can be implemented by providing aspect plugins, which can be added to Market API implementations.

This mechanism is classified as Market API [Capability Level]() 2.

#### Agreement Phase - The “Handshake”

The agreement “handshake” **begins with Requestor** deciding to accept a Proposal with a selected Offer matching their Demand. The outcome of this phase is double signed Agreement persisted on both sides. The process is as follows:

* The Requestor creates an Agreement from the negotiated Proposal and signs it with Requestor key \(“signR”\), then sends the signed Agreement to Provider.
* The Provider validates the received Agreement and makes a final decision to “pick the job”. When approved, the Provider node signs the Agreement with their key \(“signP”\) and sends it back to the Requestor, awaiting for a delivery acknowledgement.
  * The Provider may decide to break the Agreement handshake by sending a RejectAgreement message instead of an approval. This “resets” the Agreement Phase. The Requestor may decide to start negotiating again by sending a Proposal \(back to the Negotiation Phase\).
  * While the Provider is awaiting the acknowledgement of ApproveAgreement message, the Requestor may still send CancelAgreement message. This also “resets” the Agreement Phase.
* The Requestor’s acknowledgement is added to the Agreement artifact and returned to Provider.
* The successful conclusion of the Agreement approval process is signaled to both Requestor and Provider by a return from WaitForApproval/ApproveAgreement call with Ok response.
  * **The Market API Implementation must ensure that Ok on the Provider side is returned before corresponding Ok confirmation is returned on the Requestor side.** This is to make sure the Provider is ready to start an activity before Requestor is able to request it \(a subject of separate Activity API spec\).
* After WaitForApproval/ApproveAgreement has been successfully delivered to both sides, both the Provider and the Requestor consider the Agreement Approved and “in force”.

**Agreement persistence**

Once an Agreement is created \(on Requestor side\) or received \(on Provider side\) the Market API implementation is expected to reliably persist its content \(latest known state\). At any point, the Market API client code should be able to extract its body with signatures based on Agreement Id.

**Agreement lifecycle**

![](../.gitbook/assets/5%20%282%29.png)

### Market API Capability Levels

Elements of Market API functionality can be perceived as blocks. The implementation of all blocks for every implementation may not be pragmatic or feasible, however, it needs to be clear to the consumers what functionality blocks a given Market API implementation provides. The functional blocks are therefore aggregated in Capability Levels, specified as follows:

A given Market API Implementation is expected to indicate the Capability Level it conforms with. The Market API consumer is, therefore, able to leverage this information and adjust its strategy to the Capabilities provided by the Market.

The Capability Level structure may also serve as implementation roadmap guidelines so that implementation development work is phased, and subsequent Capability Levels are delivered in subsequent development phases.

## Provider Node Operations

This section describes operations available in the Market API on the Provider side.

### Subscribe

Publish Provider’s service capabilities on the market to declare an interest in Demands meeting specified criteria.

**Inputs:**

* Offer \(mandatory\)

**Output \(synchronous\):**

* Subscription Id

**Description:**

Provider capabilities are declared via an Offer object which can be considered an “open” or public Offer, as it is not directed at a specific Requestor, but rather is sent to the market so that the matching mechanism implementation can associate relevant Demands.

**Note:** Subscribe\(\) is an “atomic” operation, ie. as soon as Subscription is placed, the Offer is published on the market.

### Unsubscribe

Stop subscription by invalidating a previously published Offer.

**Inputs:**

* Subscription Id

**Output:**

* Ok or Error

**Description:**

Provider effectively expects to stop receiving Proposals. It is up to matching mechanism implementation whether the Unsubscribe\(\) call should actively invalidate previously published Offers \(eg. by sending a “cancel” message around the network\), or simply start ignoring responses appearing for the canceled subscription.

**Note:** Unsubscribe\(\) call will terminate all pending Collect\(\) calls on this subscription. This implies that client code should not Unsubscribe\(\) before it has received all expected/useful inputs from Collect\(\).

### Collect

This is an “iterator” function - which reads a collection of Events which have arrived from the market in response to Offers declared by the Provider.

**Inputs:**

* Subscription Id and/or collection of Proposal Ids
* Max count of results to read
* Timeout

**Outputs:**

* Collection of Event objects or Timeout. The returned Events may belong to the following classes:
  * Proposal \(with status indicating the current stage of Negotiation\) - this means that a Proposal can either be:
    * a “negotiation proposal” to which Provider is expected to respond with a CounterProposal\(\) or
    * An “agreement draft” to which Provider should respond via either ApproveAgreement\(\) or RejectAgreement\(\) \(see below\)
  * PropertyQuery \(for [Dynamic Property Resolution in Discovery Phase]()\)
  * Agreement proposal - indicating that the Requestor is intending to strike a contract based on specific Demand and Offer objects.

**Description:**

The collection of received Demand Proposals from the Market API module happens on a “pull” basis.

The caller may specify a Subscription Id to collect all events related to a specific Subscription Id, or may specify a list of specific Proposal Ids to listen for messages related only to specific Proposals.

This is a **blocking operation**, ie. it will not return until there is a non-zero count of messages in the arrival queue. As soon as messages arrive, they are returned \(up to max\_count\).

**Note:** Collect\(\) called on non-existent subscription Id results in “Subscription does not exist” error.

**Note:** When Collect\(\) is waiting, simultaneous call to Unsubscribe\(\) on the same subscription Id should result in “Subscription does not exist” error returned from Collect\(\).

**Note for implementers:**  
Clients are able to indirectly indicate their interest by not calling Collect\(\). Implementation can stop processing and pushing new events to the queue when there are already unreceived ones \(with arbitrarily predefined threshold\).

### CounterProposal

Allows sending a bespoke Offer in response to specific Demand. This created Proposal is routed **directly** to the issuing Requestor Node.

**Inputs:**

* Demand Id
* Offer

**Notes:**

This method allows participating in the Negotiation phase of interaction with a Requestor. The Provider is expected to actively adjust the Offer to constraints expressed in the Demand \(by resolving all possible missing properties\)

The call to CounterProposal\(\) generates a new Proposal Id. The purpose of this Id is to enable the Market API client to call Collect\(Proposal Id\) to listen for the response from the issuing Provider node.

### RejectProposal

Allows rejecting a bespoke Offer. This is routed **directly** to the issuing Requestor Node.

**Inputs:**

* Proposal Id

**Notes:**

Call to RejectProposal\(\) effectively ends a Negotiation chain - it explicitly indicates that the sender will not create another counter-Proposal.

### ApproveAgreement

Confirms the Agreement received from the Requestor. Mutually exclusive with RejectAgreement.

**Input:**

* Agreement Id

**Output:**

* Ok or Cancelled \(see Description below\)

**Description:**

Similarly to Collect\(\) this is a **blocking** operation. It returns one of the following options:

* Ok - Indicates that the approved Agreement has been successfully delivered to the Requestor and acknowledged.
  * The Requestor side has been notified about the Provider’s commitment to the Agreement.
  * The Provider is now ready to accept a request to start an Activity as described in the negotiated agreement.
  * The Requestor’s corresponding WaitForApproval call returns Ok **after** the one on the Provider side.
* Cancelled - Indicates that before delivering the approved Agreement the Requestor has sent a CancelAgreement message, thus invalidating the Agreement. The Provider may attempt to return to the Negotiation phase by sending a new Proposal.

**Note:** It is expected from the Provider node implementation to “ring-fence” the resources required to fulfill the Agreement before the ApproveAgreement is sent. However, the resources should not be fully committed until Ok response is received from the ApproveAgreement\(\) call.

### RejectAgreement

Rejects the Agreement received from the Requestor. Mutually exclusive with ApproveAgreement.

**Input:**

* Agreement Id

**Output:**

* Ok or Error

**Description:**

* The Requestor side is notified about the Provider’s decision to reject a negotiated agreement.
* This effectively stops the Agreement handshake.

### TerminateAgreement

Method to finish the Agreement while in Approved state.

**Input:**

* Agreement Id

**Output:**

* Ok or Error

**Effect:**

The Requestor gets notified about the Provider’s decision to terminate a “running” agreement.

## Requestor Node Operations

This section describes operations available in the Market API on the Requestor side.

### Subscribe

Publish Requestor’s service Demand on the market to declare an interest in Offers meeting specified criteria.

**Inputs:**

* Demand \(mandatory\)

**Output \(synchronous\):**

* Subscription Id

**Description:**

Raise a query on the market, so that matching Offers can be routed to the Requestor. Note that the requirements are declared via a Demand object - which can be considered an “open” or public Demand, as it is not directed at a specific Provider, but rather is sent to the market so that the matching mechanism implementation can associate relevant Offers.

**Note:** Subscribe\(\) is an “atomic” operation, ie. as soon as the Subscription is created, the Demand is published on the market.

### Unsubscribe

Stop subscription by invalidating a previously published Demand.

**Inputs:**

* Subscription Id

**Output:**

* Ok or Error

**Description:**

Requestor effectively expects to stop the stream of Offers incoming as an effect of a raised subscription. It is up to matching mechanism implementation whether the Unsubscribe\(\) call should actively invalidate previously published Demands \(eg. by sending a “cancel” message around the network\), or simply start ignoring responses appearing for the canceled subscription.

**Note:** Unsubscribe\(\) call will terminate all pending Collect\(\) calls on this subscription. This implies, that client code should not Unsubscribe\(\) before it has received all expected/useful inputs from Collect\(\).

### Collect

This is an “iterator” function - which reads a collection of Events which have arrived from the market in response to a Demand declared by the Requestor in Subscribe\(\) call.

**Inputs:**

* Filter Subscription Id
* Max count of messages to read
* Timeout

**Outputs:**

* Collection of Event objects or Timeout. The returned objects may belong to following classes:
  * Proposal \(with status indicating the current stage of Negotiation\)
  * PropertyQuery \(for [Dynamic Property Resolution in Discovery Phase]()\)

**Description:**

The collection of received Events from the Market API module happens on a “pull” basis.

The caller may specify a Subscription Id to collect all events related to a specific Subscription Id, or may specify a list of specific Proposal Ids to listen for messages related only to specific Proposals.

This is a **blocking operation**, ie. it will not return until expected messages arrive \(or the timeout expires\).

**Note:** Collect\(\) called on non-existent subscription Id results in “Subscription does not exist” error.

**Note:** When Collect\(\) is waiting, simultaneous call to Unsubscribe\(\) on the same subscription Id should result in “Subscription does not exist” error returned from Collect\(\).

Note for implementers:  
Clients are able to indirectly indicate their interest by not calling Collect\(\). Implementation can stop processing and pushing new events to the queue when there are already unreceived ones \(with arbitrarily predefined threshold\).

### CounterProposal

Allows sending a bespoke Demand in response to specific Offer. This created Proposal is routed directly to the issuing Provider Node.

**Inputs:**

* Offer Id
* Demand

**Outputs:**

* Proposal Id

**Notes:**

This method allows participating in the Negotiation phase of interaction with Provider. The Requestor is expected to actively adjust its Demand to constraints expressed in the received Offer \(by resolving all possible missing properties\)

The call to CounterProposal\(\) generates a new Proposal Id. The purpose of this Id is to enable the Market API client to call Collect\(Proposal Id\) to listen for the response from the issuing Provider node.

### RejectProposal

Allows rejecting a bespoke Offer. This is routed **directly** to the issuing Provider Node.

**Inputs:**

* Proposal Id

**Notes:**

Call to RejectProposal\(\) effectively ends a Negotiation chain - it explicitly indicates that the sender will not create another counter-Proposal.

### CreateAgreement

Method to initiate the Agreement handshake phase.

**Input:**

* Proposal Id
* Agreement Approval Expiry Date

**Output:**

* Agreement Id

**Description:**

* The Market API implementation formulates an Agreement artifact from the Proposal indicated by the received Proposal Id.

The Approval Expiry Date is added to Agreement artifact and implies the effective timeout on the whole Agreement confirmation sequence.

* A successful call to CreateAgreement shall immediately be followed by ConfirmAgreement and WaitForApproval call in order to listen for responses from the Provider.

### ConfirmAgreement

Sign and send a proposed Agreement message to the Provider.

**Input:**

* Agreement Id

### WaitForApproval

Wait for the response from Provider after an Agreement proposal has been sent, expecting corresponding ApproveAgreement message.

**Output:**

* Response from Provider \(see Description\) or Timeout.

**Description:**

Similarly to Collect\(\) this is a **blocking** operation.

The call may also be aborted by Requestor caller code. After the call is aborted, another WaitForApproval\(\) call can be raised on the same Agreement Id.

It returns one of the following options:

* Ok - Indicates that the Agreement has been approved by the Provider.
  * The Provider is now ready to accept a request to start an Activity as described in the negotiated agreement.
  * The Requestor’s corresponding WaitForApproval call returns Ok **after** this on the Provider side.
* Rejected - Indicates that the Provider has sent a RejectAgreement message, which effectively stops the Agreement handshake. The Requestor may attempt to return to the Negotiation phase by sending a new Proposal.
* Cancelled - Indicates that the Requestor himself has sent a CancelAgreement message, which effectively stops the Agreement handshake.

**Note:** The WaitForApproval\(\) has been separated from the ConfirmAgreement\(\) - the reason for that is: ConfirmAgreement\(\) is only called once, while WaitForApproval\(\) can be called multiple times in a sequence \(eg. in case of timeouts\).

### CancelAgreement

Method to cancel the Agreement while still in the Proposed state.

**Input:**

* Agreement Id

**Output:**

* Ok or Error

**Description:**

The CancelAgreement message delivered to Provider should cause the awaiting WaitForApproval call to return with “Cancelled” response.

### TerminateAgreement

Method to finish/close the Agreement while in Approved state.

**Input:**

* Agreement Id

**Output:**

* Ok or Error

**Effect:**

The Provider gets notified about the Requestor’s decision to terminate a “running” agreement.

## Common Operations

Following operations are common to Market API on both Requestor and Provider Nodes.

### BeginScan

A method which allows querying the market for Demands/Offers currently published. This serves the purpose of “market observation”.

**Input:**

* Observed object type \(Demand/Offer\)
* Filter constraints.

**Output:**

* Scan Id

**Effect:**

* The query is dispatched to the market and “scanning” is started. The results of the scan \(objects matching the specified criteria\) are aggregated by Market API module and can be received by calling CollectScanResults.

**Note:** This method must be implemented for Market API [Capability Level]() 2.

### CollectScanResults

Receive results of previously started scanning query.

**Input:**

* Scan Id
* Max count of results to read
* Timeout

**Output:**

* Collection of scan results or Timeout
  * The “results” is a collection of messages/objects found on the market during the scan. The scan criteria \(as submitted in BeginScan\(\) call\) indicate the result object type \(Demand/Offer\).

**Effect:**

* The collection of query results from the Market API module happens on a “pull” basis.

This is a **blocking operation**, ie. it will not return until expected messages arrive \(or the timeout expires\).

**Note:** This method must be implemented for Market API [Capability Level]() 2.

### EndScan

Ends a previously started market scanning action.

**Input:**

* Scan Id

**Output:**

* Ok or Error

**Effect:**

* The running query is finished.

**Note:** This method must be implemented for Market API [Capability Level]() 2.

### RespondPropertyQuery

A method which allows the node which had received a dynamic property query to respond with bespoke property values \(see [Dynamic Property Resolution in Discovery Phase]()\).

**Input:**

* Query Id
* Property value set as requested in the Query, adjusted for the context of received query.
* List of properties which are still pending resolution \(see Effect\)

**Output:**

* Ok or Error

**Effect:**

Delivers the property values back to the Market implementation so that they can be used for Demand/Offer matching.

**Note:** The property query responses may be submitted in “chunks”, ie. the responder may choose to resolve ‘quick’/lightweight’ properties faster and provide response sooner, while still working on more time-consuming properties in the background. Therefore the response contains both the resolved properties, as well as list of properties which responder knows still require resolution.

This method must be implemented for Market API [Capability Level]() 2.

### GetAgreementContent

\(Offline operation\) A method which allows extracting the agreement artifact from the Market API module \(for the purposes of persistence outside of Golem Node, escalation eg. to external arbitration body, etc.\)

**Input:**

* Agreement Id

**Output:**

* Agreement body \(byte stream\)

**Effect:**

* Gets the body of Agreement as persisted by the Golem Node \(the latest known state is returned\)

### ValidateAgreementContent

\(Optional, potentially offline operation\) Method for validation of agreement submitted in the form of byte stream.

**Input:**

* Agreement body \(byte stream\)

**Output:**

* Validation result

## Implementation notes

* Persistence - it is recommended that following artifacts are recorded for auditing and dispute resolution purposes:
  * Agreement

## Implementation Notes

#### Distributed Market Protocol vs GNT

The Golem Network “native” implementation of Market API must include some forms of anti-Sybil prevention. Where applicable this shall be based on GNT costs, which should be incurred by:

* Demand/Offer Subscriptions - both Requestors and Providers should be charged for Demands/Offers they publish in the network.
* Volume of Proposals received by Subscriptions from the network - as the main cost of running the Golem Network is carried by the nodes which relay the Demands/Offers, the nodes may be rewarded for participation in the network by earning some share of the GNT cost paid by the Demand/Offer publishers.

## Related documents

[Golem Demand & Offer Specification Language](https://drive.google.com/a/golem.network/open?id=1tzMrhdBr9wiUXtSn1JO18MmIiP31dkMakdjStnF3eZY)

[Golem Demand & Offer Matching - Problem statement](https://drive.google.com/a/golem.network/open?id=1yTupuRsN9DKVrK1TPhM6dBxKCAPk0wCB8KxRf57ZkV4)

