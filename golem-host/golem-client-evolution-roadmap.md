# Golem Client evolution roadmap

The purpose of this article is to illustrate a proposed breakdown of tasks intended to establish the **golem-client** component of Golem so that it becomes a launchpad for implementation of the “new” golem daemon.

## Starting point

![](../.gitbook/assets/0%20%282%29.png)

## Interim situation

![](../.gitbook/assets/1%20%283%29.png)

## Phase 1

## Step 1 - golem-client

To create **golem-client** as a platform to add low-level APIs to prototype and illustrate the newly designed protocols, spread awareness of the generic golem model, practice the new concepts, etc.

1. Reimplement golemcli functionality in rust \(as per [https://docs.golem.network/\#/Products/Brass-Beta/Command-line-interface](https://docs.golem.network/#/Products/Brass-Beta/Command-line-interface) and python docs\)
2. Implement **golem-client** "daemon mode" - with ability to publish RESTful APIs
3. Implement Requestor MarketAPI facade which communicates with golemapp.py \(just as Golem Electron does\) to implement operations of the Market API \(see [this document](https://docs.google.com/document/d/1WFC-SNaD8LQfm2h_YgpH4zHFmyFAmwAr0dHurm3uGaE/edit?usp=sharing) as well\):
   1. POST /demands \(where request contains a Demand in json form\) This call gets translated into a Task which gets published in P2P network.
   2. GET /demands/{subscriptionId}/events As WantToComputeTask objects arrive, they get translated into Offer proposal artifacts and are available via “event collect” mechanism.
   3. PUT /demands/{subscriptionId}/proposal/{proposalId} \(for negotiation\) POST /agreement/{agreementId}/approve A submitted Demand Proposal gets mapped onto TaskToCompute, which includes subtask spec \(as expressed in Demand proposal properties\). The TaskToCompute includes extra data which can be used to transport the Demand proposal details \(including docker image data, rendering params like resolution, number of frames, etc - these would belong to standardized golem.svc.app.media.render namespace\).
   4. Note. In Golem Messages there is a possibility \(but not obligatory?\) to acknowledge the acceptance of a Task by Provider?
4. Cooperate with R&D and CGI teams and use Demand/Offer properties language to model the selected use cases \(transcoding?\)
5. Design standard docker image 'interface' on Provider side \(as the Simple Task API tries to\) - to define eg. benchmark call, input, output locations, etc.
6. Design a simple "mock-like" implementation on Provider side - to respond using the Offer artifacts \(but on old protocol, as "extradata" in golem messages\)

## Step 2 - Sample Client App

To illustrate the interaction with Golem APIs.

1. Implement simple UI to control Demand generation \(command line, eg. shell script with simple curl calls, json props, whatever\)
2. Implement simple Offer selection strategy \(first incoming offer below given fixed price threshold - accepted\)
3. Implement Agreement signing

## Step 3 - Define Activity API \(in parallel\)

So that it can be called by Sample Client App and so that it's first implementation is implemented on top of golem messages

The first iteration \(an MVP\) will probably have limited scope, but it will be iteratively enhanced.

## Phase 2

## Step 4 - Implement Golem-net

Implement Golem-net to the extent required by Distributed Market Protocol.

## Step 5 - Distributed Market Protocol Implementation

Implement the Distributed Market protocol on both Requestor & Provider so that it replaces the "Golem Messages implementation” from Step 1 - c\) above

## References:

Market API spec [https://docs.google.com/document/d/1Zny\_vfgWV-hcsKS7P-Kdr3Fb0dwfl-6T\_cYKVQ9mkNg/edit\#heading=h.90kan3ygeeum](https://docs.google.com/document/d/1Zny_vfgWV-hcsKS7P-Kdr3Fb0dwfl-6T_cYKVQ9mkNg/edit#heading=h.90kan3ygeeum)

Brass Gateway for Unlimited [https://github.com/golemfactory/golem-unlimited/wiki/Brass-Gateway](https://github.com/golemfactory/golem-unlimited/wiki/Brass-Gateway)

Market API swagger spec \(by reqc\) [https://github.com/golemfactory/golem-model/blob/market-api/market/spec/market-api.yaml](https://github.com/golemfactory/golem-model/blob/market-api/market/spec/market-api.yaml)

Golemrustcli \(polish by 2rec\) zajawka ;\) [https://docs.google.com/document/d/1euDJ0hMnAs6zgdygefOc6obcgQDUcGWsuI1fw8AS0L8/edit\#](https://docs.google.com/document/d/1euDJ0hMnAs6zgdygefOc6obcgQDUcGWsuI1fw8AS0L8/edit#)  
google translation \(by Gert-Jan\) \([https://docs.google.com/document/d/1gdvPaejBqSjfOhWtqeHdDkR2UcPZ0pJwxD-dsR3q-oo](https://docs.google.com/document/d/1gdvPaejBqSjfOhWtqeHdDkR2UcPZ0pJwxD-dsR3q-oo)\)

Rost-golemcli Phase 1 Work Breakdown Structure:

[https://docs.google.com/spreadsheets/d/1RGGj8ArXySmYVywczyXRAqiUrUlCA9VuhA81zGSUmiA/edit\#gid=0](https://docs.google.com/spreadsheets/d/1RGGj8ArXySmYVywczyXRAqiUrUlCA9VuhA81zGSUmiA/edit#gid=0)

Golem CLI documentation \(outdated; originally by badb + mf\)  
[https://docs.google.com/document/d/12EXpHCIBkqJ37nIPGCZEc-bgKn-ewUacXTyPmMkpeiw](https://docs.google.com/document/d/12EXpHCIBkqJ37nIPGCZEc-bgKn-ewUacXTyPmMkpeiw)

Golem RPC \(by mplebanski\)  
[https://github.com/golemfactory/golemrpc](https://github.com/golemfactory/golemrpc)

Golem CLI mockup rust impl \(by Gert-Jan\)  
[https://github.com/maaktweluit/golemcli](https://github.com/maaktweluit/golemcli)

