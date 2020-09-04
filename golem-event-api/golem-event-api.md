# Golem Event API

Golem Event API

Specification

Version 0.1

## Abstract

The purpose of this paper is to define a logical API for handling events observed in Golem Network node implementations. Event API is a technical interface which facilitates the flow of event notifications related to all Functional APIs. It specifies a uniform pattern of collecting event objects from a Daemon instance.

## Event API

The Event API is symmetric, ie. identical on both Requestor and Provider sides. Note that the Event Type sets received via Event API differ - the difference is indicated in the [Event Types]() section.

The events published to client applications are aggregated by Subscription Id. This means that all events generated as a result of a specific Demand/Offer subscription - Market, Activity or Payment related - are available via the Subscription Id. The client applications are expected to route the incoming event objects to relevant modules for further processing.

### CollectSubscriptionEvents

**Inputs:**

* SubscriptionId
* IncludeEventTypes
* ExcludeEventTypes
* Timeout
* AfterTimestamp \(optional\)
* MaxEvents

**Outputs:**

* Collection of event objects

**Description:**

This method performs long-polling on a queue of event objects arriving in the Daemon process as a result of activities performed on the Network.

**Note:** The Event Type filtering criteria can be used to specify event types to be included or excluded from scope. Asterisk \(\*\) can be used as a wildcard. Eg.

1\) include all Event Types

includeEventTypes: \["\*"\]

2\) include only specific Event Types

includeEventTypes: \["DemandProposalEvent", "PropertyQueryEvent"\]

3\) include all apart from some specific Event Types

includeEventTypes: \["\*"\]

excludeEventTypes: \["DemandProposalEvent", "PropertyQueryEvent"\]

4\) include all Event Types with Agreement in name

includeEventTypes: \["\*Agreement\*"\]

### PurgeSubscriptionEvents

**Inputs:**

* SubscriptionId -
* IncludeEventTypes
* ExcludeEventTypes
* BeforeTimestamp \(optional\) -

**Outputs:**

* Number of events purged

**Description:**

This method clears the event buffer on Daemon side, to release resources. It is expected for a client \(“agent”\) application to clear the queue after the events have been read and processed.

**Note:** the Daemon implementations may also implement “self-purging” regimes to cater for non-compliant client applications.

## Event Types

| **Event Type** | **API** | **Side** | **Description** |
| :--- | :--- | :--- | :--- |
| DemandProposalEvent | Market | Provider | Proposal received by Provider from the Market. |
| OfferProposalEvent | Market | Requestor | Proposal received by Requestor from the Market. |
| PropertyQueryEvent | Market | Both | Property Query received from the Market by Requestor/Provider during Proposal resolution. |
| AgreementProposalEvent | Market | Provider | Agreement proposal received by the Provider from the Requestor. |
| AgreementAcceptedEvent | Market | Requestor | Agreement acceptance notification. |
| AgreementRejectedEvent | Market | Requestor | Agreement rejection notification. |
| AgreementCancelledEvent | Market | Provider | Notification of Agreement cancellation by the Requestor. |
| AgreementTimeoutEvent | Market | Requestor | Notification of Agreement timeout during the approval process. |
| AgreementTerminatedEvent | Market | Both | Notification of Agreement termination. |
| CreateActivityEvent | Activity | Provider | Create Activity event. |
| DestroyActivityEvent | Activity | Provider | Destroy Activity event. |
| ExeScriptCommandFinishedEvent | Activity | Requestor | Result of an ExeScript command execution. |
| GetActivityStateEvent | Activity | Provider | Notification of GetActivityState request. |
| GetActivityUsageEvent | Activity | Provider | Notification of GetActivityUsage request. |
| ActivityStateChangedEvent | Activity | Requestor | Notification of Activity State change. |
| DebitNoteReceivedEvent | Payment | Requestor | Debit Note received by Requestor. |
| DebitNoteAcceptedEvent | Payment | Provider | Debit Note accepted by Requestor. |
| DebitNoteRejectedEvent | Payment | Provider | Debit Note rejected by Requestor. |
| DebitNoteFailedEvent | Payment | Both | Notification of Payment failure for Debit Note. |
| DebitNoteSettledEvent | Payment | Requestor | Notification of Payment reaching the Provider. |
| DebitNoteCancelledEvent | Payment | Requestor | Notification of Debit Note cancelled by Provider. |
| InvoiceReceivedEvent | Payment | Requestor | Invoice received by Requestor. |
| InvoiceAcceptedEvent | Payment | Provider | Invoice accepted by Requestor. |
| InvoiceRejectedEvent | Payment | Provider | Invoice rejected by Requestor. |
| InvoiceFailedEvent | Payment | Both | Notification of Payment failure for Invoice. |
| InvoiceSettledEvent | Payment | Requestor | Notification of Payment reaching the Provider. |
| InvoiceCancelledEvent | Payment | Requestor | Notification of Invoice cancelled by Provider. |
| PaymentReceivedEvent | Payment | Provider | Notification of new Payment received by Provider. |
|  |  |  |  |

