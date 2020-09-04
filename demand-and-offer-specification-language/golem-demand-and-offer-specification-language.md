# Golem Demand & Offer Specification Language

## Abstract

The purpose of this article is to present a proposed model of description of Demand and Offer artifacts which are fundamental entities in Golem ecosystem. The Demand and Offer specifications are tightly coupled with a general model of a Service description.

## Intro

The fundamental feature of Golem network & ecosystem is to enable Providers to offer their computing resources for trade, and correspondingly enable Requestors to discover those Providers and their service offers. The key element of such an ecosystem is a generic specification language which allows to express Demand and Offer artifacts which are fundamental entities in Golem. The Demand and Offer specifications are tightly coupled with a general model of a Service description and are a foundation of an Agreement artifact which binds Requestor and Provider.

The proposed “language” needs to meet a fairly broad set of requirements:

* **General** - the language must be applicable for specification of a broad set of imaginable Services/Applications traded via Golem.
* **Versatile** - the language must allow to describe reasonably extensive set of conceivable Demand & Offer specifications \(eg. trading conditions, terms of business, etc.\)
* **Scalable** - the language should be openly extensible in intuitive way, ie. it should allow for parties in Golem ecosystem to easily add to the language in systematic way.
* **Constrained** - the language must not allow for abuse \(eg. specifying resource condition which would result in endless resolution must not be possible\).
* **Open** - the language should be abstract, but the possible applications of the language must be highlighted via a set of examples and pattern repositories.

## General Service Description Model

This paper attempts to capture and specify a general model that could describe a sufficiently broad class of computing services tradable on Golem Network.

A general service description seems to include:

* **Resource vector R**: describes “infrastructural/environmental” qualitative and quantitative parameters of the Service which is the object of the trade.
* **Usage vector B**: specifies the individual factors contributing to the measure of Service consumption \(ie. how much “resources” has been “burned” by the service Requestor\). The factors are expressed in arbitrary “counters” \(note that “uptime” can also be regarded as a counter.\)
* **Context vector C**: includes relevant dynamic factors influencing the price \(eg. “current date/time” may influence the rate at which a second of VM uptime is charged\)
* **Pricing function:** indication of how service parameters and cumulative usage contribute to the cost of service \(to be paid by the Requestor\). The cost is a function of \(at least\) Resource vector R, Usage vector B and additional “Context” vector C:

P = f\(R, B, C\)

## Specification of Resource Vector

The concept of describing the entities in Golem using generic system of properties is derived from **OSGi Requirements and Capabilities Model** \(a graceful intro to OSGi Reqs & Caps can be found [here](https://blog.osgi.org/2015/12/using-requirements-and-capabilities.html)\)

A Resource vector is a tuple of components/dimensions, each of which describe a property of the service being offered. A set of imaginable properties describing all conceivable services is quasi-infinite, and requires a scalable naming system.

A Resource vector can be expressed as an array of properties, which need to be given as key-value pairs in the format:

&lt;name&gt;\(”=”&lt;value&gt;\)?

where the value information is optional.

### Property naming

A property name may include any character apart from ‘special’ characters \(\[, \], =, \*, $\).

Note that there may be various naming conventions proposed to bring structure and readability to property name space.

* A **hierarchic namespace system** can be defined to categorize property names into “topic areas”. An illustration of such structure \(derived from programming languages like Java or C\#, with namespace names concatenated by ‘.’ characters\) is presented further down this document.
* A naming convention can be proposed to model and express **“parametric/dynamic” properties**, which are in fact pseudo-functions - may resolve to different values depending on “parameter” included in the property name. Such “parameter” may be specified in a constraint/filtering expression, while a resolver engine would include modules that would parse the property names to “extract” parameters and resolve the value accordingly. So following rules are in force:
  * The property value resolution can only be performed by the node which declared the property \(ie. if a dynamic property is declared as supported by an issuer of an Offer, the value of this property for a parameter included in Demand’s constraint expression can only be resolved by the issuer of the Offer.\)
  * A Demand/Offer matching resolver implementations and all mechanisms leveraging property resolution need to be aware of the rule above, so that the resolution of dynamic properties is delegated to the right nodes.

 Consider following constraint:

 \(golem.inf.net.ipv4.tcp\_visible{www.onet.pl:443}=true\)

 The semantics of such sample property would be defined so that the literal specified between {} brackets is interpreted as a “url” “parameter” to a function which verifies network visibility of this url via TCP protocol.

### Property syntax

#### Property types

The properties are declared to be of a specific type, which is important as it has impact on how comparison operators work with properties of different types. The type of property is inferred from the literal used to specify the value.

Following property types are supported:

* String - any value declared in quotes, eg: “sample text”
* Bool - any of following literals: true, True, TRUE, false, False, FALSE
* Number - any value which can be successfully parsed to a numeric constant, eg. 12, 34.56, 12e-02
* Decimal - any value which can be successfully parsed to a decimal constant, eg. 12, 34.56 \(&lt;prop\_name&gt;@d for JSON form and property references\)
* DateTime - a date/time string in quotes, prefixed by character t, eg. t”1985-04-12T23:20:50.52Z” \(&lt;prop\_name&gt;@t for JSON form and property references\)

**Note** DateTime timestamps must be expressed in [RFC 3339](https://www.ietf.org/rfc/rfc3339.txt) format.

* Version - a version number string in quotes, prefixed by character v, eg. v”1.3.0”. The version number is expected to follow [semantic versioning](https://semver.org/) arithmetic. \(&lt;prop\_name&gt;@v for JSON form and property references\)
* List - composite type indicated by syntax: “\[“&lt;item&gt;\(“,”&lt;item&gt;\)\*”\]”, where &lt;item&gt; is a literal expressing a value of one of types mentioned earlier.

The first item in the list declaration indicates the type of all elements in the list. If a List declaration contains literals indicating different types - a syntax error must be signalled by the parser.

**Note** that the only operator applicable to List is ‘=’ \(which is equivalent to “contains”\), in 2 variants:

* ‘=’ with a scalar value is resolved as “contains” \(does a list contain one particular element\),
* ‘=’ with a list verifies if the property contains a list identical to the one specified in constraint expression.

#### Property aspects

The properties flowing through Golem implementation may carry additional metadata called “aspects”. Note that aspects cannot be expressed via Demand/Offer property language expressions - they can only be “injected” by host implementation.

* An aspect is a syntactic construct which is meant to assign additional attributes to properties themselves. When no aspect is specified, the literal refers to the value of the property itself.
* The aspects would likely be used by host software \(ie. constraint parsers/resolvers\) to indicate “context” information related to properties. The aspects would most likely be assigned “from outside”, ie. by resolution engine rather than by Demand or Offer designer.
* The aspects can be referred to by constraint expressions \(see below in [Property Referencing]()\)

One use would be to define some semantics to express external attestation of property value against an entity being described. Eg. a service provider declares itself as having a reputation score granted to him by an external party. This score is “attested” by eg. a signed token issued by external provider for the provider, which can be shared with a requestor. The “aspects” would allow to refer to this signed token information from the Demand/Offer description language.

**Example 1**

Imagine an AWS EC2 virtual machine, described by a Resource vector:

\[Region; OS; vCPU; RAM; ECU; Local storage\] = \[“EU \(Frankfurt\)”; “Windows”; 2; 8 GB; 13; 50 GB\]

This Resource Vector expressed in Golem property notation would look as follows:

golem.inf.cpu.cores=2;

golem.inf.mem.gib=8;

golem.inf.storage.gib=50;

golem.node.geo.country\_code=”de”; // note this property does not exist above, but is implied by AWS’ \(Frankfurt\) region

golem.inf.os.name=”Win”;

aws.ec2.region=”EU \(Frankfurt\)”;

aws.ec2.ecu=13;

The example above includes _properties_ \(eg. “cores”\) and their _types_ \(inferred from value literals\). The _properties_ are grouped in _namespaces_ \(eg. “golem.inf”\) and have _values_ \(eg. “golem.inf.mem.gib=8”\).

**Note:** observe how namespaces can be used to segment the properties into semantic groups, including also service/application specific properties/attributes.

**Example 2**

Imagine Golem Brass Blender computation service offered by a Provider as a Golem-format Docker image hosted in a generic Docker execution environment, described as:

\[Num cores; RAM size; Storage size; Performance index; Concent status\] = \[2; 16 GB; 10 GB; 124; “Required”\]

This service can be described in Golem properties as follows:

golem.svc.docker.image=\[”golemfactory/blender”\]

golem.inf.cpu.cores=2;

golem.inf.mem.gib=8;

golem.inf.storage.gib=50;

golem.com.concent.required=true;

### Property vector flat form

The property set can be expressed as a flat array of text lines, as in examples listed above.

### Property vector JSON form

In some applications, the property vector can also be expressed in JSON form, which may be more efficient/applicable in some situations. For example, the equivalent of Example 2 above expressed in JSON would be as follows:

```text
{
   "golem":{
      "svc":{
         "name":"Golem Blender",
         "docker":{
            "image":[
               "golemfactory/blender"
            ]
         }
      },
      "inf":{
         "cpu":{
            "cores":2
         },
         "mem":{
            "gib":8
         },
         "storage":{
            "gib":50
         }
      },
      "com":{
         "concent":{
            "required":true
         }
      }
   }
}
ABOUT
The JSON Format
```

### Property special scenarios

**“Value-less” property**

In Demand/Offer negotiation scenarios it may required to indicate that a property is supported by a node, but specifying its value is not possible/practical, eg.:

* A Provider wishes to reveal a property value only to a specific Requestor/Demand, but in a public market it wants to indicate that the property is supported.
* A property is “dynamic”, ie. its value depends on external factors, like Requestor’s identity, current network configuration, Requestor’s specific constraints, etc. So an open Offer would only indicate that a property is supported, and the actual value would be returned on specific request, eg. to a specific targeted Demand.

The value-less property would be indicated in property set by:

* In flat-form: A mention of property name only, with no ‘=’ operator and no value
* In JSON-form: A property field initialized to null value.

**Value name wildcards**

In scenarios where property mechanism is leveraged to implement a “pseudo-function” \(see [Property Naming]()\), the property-issuing side needs to indicate that a “pseudo-function” is supported. It does this by declaring a value-less property with ‘\*’ wildcard. For example indicating a following property in Offer property set:

golem.inf.net.ipv4.tcp\_visible{\*}

Indicates, that the Provider node supports a pseudo-function capable of verifying network visibility of any URL specified in Demand’s constraint filter expression, eg.:

 \(golem.inf.net.ipv4.tcp\_visible{www.onet.pl:443}=true\)

### Property and aspect space expressed in Golem notation

While the property space is generally unbounded, and any participant of Golem ecosystem is able to extend property set \(ie. introduce and start using new property names\) it is practical to provide guidelines for property semantics and naming conventions.

Golem Factory maintains a set of Computing Resource Standards to specify property namespaces describing common services and computing resources. The standards library can be found here: \([draft](https://github.com/golemfactory/golem-architecture/tree/draft/standards)\).

## Specification of Resource Constraints

With Resources or Services described in Golem properties, it is required to be able to specify constraints against the properties. This is necessary to eg. filter service offers in the open market

Again, inspired by OSGi Requirements and Capabilities model, the Golem ecosystem utilizes constraint language derived from LDAP filter expressions.

### Property referencing

In the Constraint expressions, the properties are referenced using following grammar:

&lt;name&gt;\(“@”&lt;typecode&gt;\)?\(“\[“&lt;aspect&gt;”\]”\)?&lt;operator&gt;&lt;value&gt;?

Specifying no aspect means we are referencing the property value.

@&lt;typecode&gt; is optional in property reference and implies a specific type of constraint value \(this determines the behaviour of operators\). If a type code is not specified, the type of property as declared in Demand/Offer determines the operator behaviour. Type codes are indicated in [Property types]() section.

**Example 1**

So, a sample AWS VM capability defined earlier could be selected by a resource requirement defined as follows:

filter=\(&

\(golem.inf.cpu.cores=2\)

\(golem.inf.mem.gib=8\)

\)

**Example 2**

If an AWS-specific region constraint is expected:

filter=\(&

\(golem.inf.cpu.cores=2\)

\(golem.inf.mem.gib=8\)

\(aws.ec2.region=EU \(Frankfurt\)\)

\)

Note how a different namespace is specified for one of query components.

**Example 3**

Furthermore, if we want to refer to some arbitrarily selected reputation score:

filter=\(&

\(golem.inf.cpu.cores=2\)

\(golem.inf.mem.gib=8\)

\(aws.ec2.region=EU \(Frankfurt\)\)

\(svc.reliablerepu.score=High\)

\(svc.reliablerepu.score\[golem.attester.thumbprint\]=02faf3e...1868\)

\)

Note how an aspect of reputation score is referenced to accept only scores which are properly signed by a specific “attestation party”.

We propose using the RFC 4515 \(LDAP Search Filter Strings, see [References]()\) with few enhancements as the specification for the filter syntax. It seems powerful enough to indicate a sufficiently broad class of service requirements, but limited enough to reduce potential security threats.

The proposed subset LDAP Search Filter notation must include following features:

* AND, OR, NOT logical operators
* Straight comparison of property values
* Presence operator \(“=\*”\) - check if a property is defined
* Comparison of property values \(“&gt;=”, “&lt;=” operators\)
* Wildcard-based matching of property values

Following enhancements are required:

* Lexical support for property “aspects”

## Specification of Usage vector

As the Usage basis vector includes a series of counter values, which indicate the elapsed resource consumption, the components of the Usage vector can also be defined via pseudo-OSGi notation \(namespaces/counter names\).

**Example 1**

A sample Azure Function deployed in cloud may yield following Usage vector:

\[No of requests; Aggregated RAM consumption\] = \[150034; 22.456 GBs\]

Such vector represented in counters \(flat notation\) would look like:

golem.usage.transactions=150034;

golem.usage.ram.gb\_sec=22.456;

**Example 2**

The Usage vector from previous example may also be expressed in JSON form:

```text
{
   "golem":{
      "usage":{
         "transactions":150034,
         "ram":{
            "gb_sec":22.456
         }
      }
   }
}
```

Possible Usage Counter space can be specified as follows:

| **Name space** | **Property** | **Type** | **Description** | **Comments** |
| :--- | :--- | :--- | :--- | :--- |
| golem.usage | duration\_sec | Numeric | Duration of activity \(in seconds\) | Up-time is usually difficult to verify without a monitoring service… |
| cpu.usage | Numeric | Integral of CPU consumption \(in %\) over time | Is this verifiable? |  |
| input.gb | Numeric | Input data volume \(as transferred over Golem Network, in GB\) | Verifiable |  |
| output.gb | Numeric | Output data volume \(as transferred over Golem network, in GB\) | Verifiable |  |
| net.upload.gb | Numeric | Network upload volume \(outside Golem Network\) \(elapsed, in GBs\) |  |  |
| net.download.gb | Numeric | Network download volume \(outside Golem Network\) \(elapsed, in GBs\) |  |  |
| transactions | Numeric | Number of transactions, elapsed |  |  |
| mem.gib\_sec | Numeric | Aggregated RAM consumption \(GB \* sec\) |  |  |
| ... |  |  |  |  |
|  |  |  |  |  |
| svc.azure.billing.vm | storage.transactions | Numeric | Azure VM Storage Transactions count |  |
| disk.read.ops | Numeric | Managed Disk Read operations count |  |  |
| disk.write.ops | Numeric | Managed Disk Write operations count |  |  |
| disk.read.volume.gb | Numeric | Managed Disk Read volume \(GBs\) |  |  |
| disk.write.volume.gb | Numeric | Managed Disk Write volume \(GBs\) |  |  |
|  |  |  |  |  |
| &lt;service provider n namespace&gt; |  |  |  |  |
|  |  |  |  |  |
| &lt;application 1&gt; namespace |  |  |  |  |

## Specification of Pricing Function

The pricing tables of cloud providers are described as sets of articles and multi-dimensional matrices, where actual price per counter depends upon a multitude of conditions defined against Resource vector components as well as Usage vector components.

For the purposes of Golem ecosystem, we should consider a requirement to define the pricing function as a “script” expressed in a general-purpose, widely known programming language, eg. JavaScript.

A properly defined pricing function has the following properties:

* It is a function of Resource vector and Usage vector
* It yields a positive \(&gt;0\), scalar result \(which indicates cost of service in GNT\)
* It yields correct results for any combination of positive \(&gt;0\) values of Usage counters

**Example 1**

A Golem Brass Blender rendering service has a simple pricing model, which is linearly dependent upon processing time. It can be expressed as a simple expression-based pricing function:

golem.usage.duration\_sec\*0.00013

**Example 2**

An AWS Lambda pricing function is fairly simple and can be expressed in JavaScript as follows:

function price\(resources, usage, context\)

{

 var endPrice = 0;

 var transactions = usage\["golem.usage.transactions"\];

 var gbSecs = usage\["golem.usage.mem.gib\_sec"\];

 if\(transactions &gt; 1000000\) {

 endPrice += \(transactions - 1000000\) \* 20 / 1000000;

 }

 if\(gbSecs &gt; 400000\) {

 endPrice += \(gbSecs - 400000\) \* 1667 / 100000000;

 }

 return endPrice / 100; // price in dollars

}

**Note:** In the example above, the “free requests per month” aspect is not considered. To include it, some “context” parameters would need to be used as well.

**Note 2:** The context information \(eg. “current date time”\) may also be required to implement rates variable depending on eg. time of day.

### Possible Pricing Function Specification Models

It is possible to express a pricing function in a multitude of ways. There should be a distinction between pricing function description “schemes” or models.

Potential pricing function schemes:

* Text - human-readable text description.
* HTML - human-readable HTML description.
* Expression - algebraic expression defining the pricing function using counters, constants and arithmetic operators
* JavaScript - an arbitrary JavaScript function \(as in the example indicated above\)
* GPE - “Golem Pricing Expressions”, see below.

#### Golem Pricing Expressions

There is potential application for a dedicated Domain Specific Language designed to express Pricing Functions. Such a DSL would allow to control the “power” and complexity of pricing functions, at the same time possibly enabling automated price comparison algorithms.

Definition of GPE language is currently out-of-scope - we leave it as an option of another “module” that can be added to the Demand & Offer Specification Language implementations.

## Specification of Demand

The Demand object/artifact specifies the service required/sought by the Requestor. The purpose of Demand is to publish the requirements in Golem network, so that matching services can be found, and potential Providers located.

The Demand consists of the following elements:

* DemandId \( string\(256\) \)
* Demand Creation Timestamp
* Demand Expiration Date/Time
* Issuing NodeId
* Service Constraints
* Issuing node \(requestor\) properties \(relevant for the requested service\)
* Attachments
  * Each attachment is a triplet of label, MIME type and binary content.
  * Attachments carry additional information which eg. may be used to enhance reliability of published properties.

### Service price validation

In order to enable the Requestor to compare Offer prices in an efficient way, the Demand may include an estimate request. The estimate request can be expressed via a “pseudo-function” property, eg. expressed via standardized properties as follows:

Price estimate request in Demand constraints \(assuming that the pricing usage vector includes one time counter\):

**\(golem.com.pricing.est{\[600\]}=\*\)**

The response Offer shall include the value of the requested “pseudo-function” property:

**golem.com.pricing.est{\[600\]}=2.4**

Thus the Requestor may present a “service usage profile” over time, so that the Provider may calculate the price which would be charged for this usage according to the pricing function.

The proposed mechanism is designed to prevent abuse of pricing functions - as it is the Provider who designs the pricing function, the Provider will also be requested to execute it. Possibly this should also prevent Providers from designing overcomplicated pricing functions, as otherwise the estimation would become computationally overwhelming in case of large volume of demands.

**Example 1 \(“targeted service demand”\)**

* Service Constraints:

filter=\(&

\(golem.svc.name=Golem Blender\)

\(golem.svc.version=1.1.3\)

 \(golem.inf.cpu.cores&gt;2\)

 \(golem.inf.mem.gib&gt;32\)

 \(golem.inf.gpu.vendor=NVidia\)

 \(golem.node.geo.country\_code=pl\)

\)

* Issuing node \(requestor\) properties:

golem.identity.org.name=”TheGreat3DStudio, Inc.”;

svc.reliablerepu.score=\[”High”\];

golem.contract.deposit=0.005;

* Service price estimate request:

None

* Additional information:

None.

**Note:** It may be desired to specify an up-front “deposit” in the Demand \(as in the example above\). By “deposit” the Requestor indicates they are placing x GNT aside for the total usage of the service. \(This can be leveraged by Provider to cut-off a running service when it exceeds the indicated deposit.\)

**Note:** the properties used in above example are indicative only.

**Example 2 \(“broad/descriptive service demand”\)**

* Service Constraints:

filter=\(&

\(golem.svc.name=Golem Blender\)

\(golem.svc.version=1.1.3\)

\(golem.svc.payload.type=docker\)

\(golem.svc.docker.image=golemfactory/blender\)

\(golem.usage.vector=\[“golem.usage.duration\_sec”\]\)

\(golem.com.pricing.est{\[600\]}=\*\) // Service price estimate request

\)

* Issuing node \(requestor\) properties:

golem.identity.org.name=”TheGreat3DStudio, Inc.”;

svc.reliablerepu.score=”High”;

golem.contract.deposit=0.005;

* Service price estimate request - as specified in constraint filter
* Additional information:

&lt;An attestation token issued by ReliableRepu to confirm the “High” score granted to the issuing node&gt;

**Note:**

* This Demand example gives more info about the service, so it allows the Provider to decide if it is able to support this service \(ie a Docker app from a specific docker image\) and eg. download the Svc/App package, etc. in order to place the offer.
* Note the price estimate request content: this is a case where the Requestor requests a price for 5 mins of continuous computation \(ie. after 600 secs since launch, the service will have burnt 600 secs of run time\)
* This Demand includes additional attestation data which confirms the validity of reputation score declared by the Requestor in this case.
* It is likely that some Svc/App Registries/Catalogs will evolve, which will allow for publishing and browsing through Svc/App descriptors, defined via sets of properties. For example it must be possible to specify constraints implied by Svc/App, and we need some way to include a description \(human-readable\).

## Specification of Offer

The Offer object/artifact specifies the service published by the Provider. The purpose of Offer is to indicate the service description and conditions in Golem network, either as response to a specific Demand or as a way to “advertise” a Service, in hope to trigger Demands for that service.

The Offer consists of following elements:

* OfferId \( string\(256\) \)
* Issuing NodeId
* Offer Creation Timestamp
* Offer Expiration Date/Time
* \(Optional\) DemandId of Demand this Offer responds to
* Service Properties \(matches Service Constraints from Demand\)
  * The properties may include the specification of Pricing Function \(Note that pricing function is optional, eg. for “open” Offers\)
* Requestor Constraints \(matches Requestor Properties from Demand\)
* Attachments
  * Each attachment is a triplet of label, MIME type and binary content.
  * Attachments carry additional information which eg. may be used to enhance reliability of published properties.

**Important:** Note that pricing function is optional on Offer description - no pricing function makes sense only for “open Offers” used by Providers to announce the supported Services on the Market.

**Example 1**

Resource vector describing the service:

* At least all properties included in Demand constraints, plus the ones which may be relevant to the Requestor

golem.svc.name=”Golem Blender”;

golem.svc.version=v”1.1.3”;

golem.usage.vector=\[”golem.usage.duration\_sec”\];

golem.inf.cpu.cores=4;

golem.inf.mem.gib=64;

golem.inf.gpu.vendor=”NVidia”;

golem.node.geo.country\_code=”pl”

golem.node.id.name=”UberDatacenter, LLC”

svc.reliablerepu.score=”Mid”

golem.com.pricing.est{\[600\]}=2.4

Requestor constraints:

filter=\(&

\(golem.env.country.iso&lt;&gt;”ru”\)

\(\|

\(svc.reliablerepu.score=Mid\)

\(svc.reliablerepu.score=High\)

\)

\(svc.reliablerepu.score\[golem.attester.thumbprint\]=02faf3e...1868\)

\)

Pricing function:

Function scheme: “Expression”

Function body: golem.usage.time.sec\*0.004

Price estimate - expressed in golem.est.price{\[600\]} property.

**Notes:**

* Observe how the Provider references the golem.attester.thumbprint aspect. thus requiring the reputation of a Requestor to be properly attested by a specific signer certificate thumbprint.

## Demand-Offer Matching

With Demand and Offer specified above, it is possible to define “matching” relation between them. It is possible to have a constraint on property that isn’t included in the evaluated property set \(being ‘undefined’\). This can be either “disqualifying” or “acceptable”. Therefore we can consider a “Strong Match” and a “Weak Match”.

### Strong Match

MS\(D; O\) = true/false

We specify a Demand D and Offer O as _strongly matching_ when:

* All service constraints specified in D are explicitly fulfilled by service properties specified in O
* All requestor constraints specified in O are explicitly fulfilled by requestor properties specified in D

If any property listed in constraints is ‘undefined’, the whole constraint expression evaluates to _**false**_.

### Weak Match

MW\(D; O\) = true/false

We specify a Demand D and Offer O as _weakly matching_ when:

* Service constraints specified in D are either explicitly fulfilled by service properties specified in O, or ignored if a property is ‘undefined’
* Requestor constraints specified in O are either explicitly fulfilled by requestor properties specified in D, or ignored if a property is ‘undefined’

The ‘undefined’ properties are ignored, ie. any constraints referring to ‘undefined’ properties should be evaluated as ‘undefined’. Logical expressions referring to ‘undefined’ shall be evaluated according to following table:

| **Expression** | **Resolution** |
| :--- | :--- |
| p AND undefined | p AND true \(= p\) |
| p OR undefined | p OR false \(= p\) |
| NOT undefined | undefined |

Both Matching relations have following properties:

* Symmetric:

M\(D; O\) = M\(O; D\)

* Non-transitive!

Assume M\(D1; O1\), M\(O1; D2\), M\(D2; O2\)

This does **not** imply that M\(D1, O2\)

## Match Strength Metrics

For the purpose of optimizations of Demand&Offer matching it may be useful to define metric to express how exactly a given Demand-Offer pair are fulfilling each other’s requirements/constraints. The strength metric:

* Is orthogonal to the true/false Matching relation.
* Should measure how accurately the constraints expressed by one side are addressed by the other side’s properties. In other words, it should **promote well-defined and specific Demands & Offers**, while those which are **vague and under-specified** \(ie. include only few generic properties\) **should score low**.

One possible implementation of strength metric is defined below:

* Take constraint expression from Demand and extract all properties mentioned.
* For each property from Demand constraints, check if it is explicitly listed in Offer. Add 1 to score for each property found.
* Take constraint expression from Offer and extract all properties mentioned.
* For each property from Offer constraints, check if it is explicitly listed in Demand. Add 1 to score for each property found.

Example:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Demand</b>
      </th>
      <th style="text-align:left"><b>Offer</b>
      </th>
      <th style="text-align:left"><b>Matching Strength Score</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>Service constraints:</p>
        <p>filter=(&amp;</p>
        <p>(<b>golem.svc.name</b>=Golem Blender)</p>
        <p>(<b>golem.svc.version</b>=1.1.3)</p>
        <p>(<b>golem.inf.cpu.cores</b>&gt;2)</p>
        <p>(<b>golem.inf.mem.gib</b>&gt;32)</p>
        <p>(<b>golem.inf.gpu.vendor</b>=NVidia)</p>
        <p>(<b>golem.node.geo.country_code</b>=PL)</p>
        <p>)</p>
        <p>Requestor properties:</p>
        <p>golem.identity.org.name=&#x201D;TheGreat3DStudio, Inc.&#x201D;;</p>
        <p><b>svc.reliablerepu.score</b>=[&#x201D;High&#x201D;];</p>
        <p>golem.contract.deposit=0.005;</p>
      </td>
      <td style="text-align:left">
        <p>Service properties:</p>
        <p><b>golem.svc.name</b>=&#x201D;Golem Blender&#x201D;;</p>
        <p><b>golem.svc.version</b>=v&#x201D;1.1.3&#x201D;;</p>
        <p>golem.usage.vector=[&#x201D;golem.usage.time.sec&#x201D;];</p>
        <p><b>golem.inf.cpu.cores</b>=4;</p>
        <p><b>golem.inf.mem.gib</b>=64;</p>
        <p><b>golem.inf.gpu.vendor</b>=&#x201D;NVidia&#x201D;;</p>
        <p><b>golem.node.geo.country_code</b>=&#x201D;PL&#x201D;</p>
        <p>golem.node.id.name=&#x201D;UberDatacenter, LLC&#x201D;</p>
        <p>svc.reliablerepu.score=&#x201D;Mid&#x201D;</p>
        <p>Requestor constraints:</p>
        <p>filter=(&amp;</p>
        <p>(golem.env.country.iso&lt;&gt;ru)</p>
        <p>(|</p>
        <p>(<b>svc.reliablerepu.score</b>=Mid)</p>
        <p>(<b>svc.reliablerepu.score</b>=High)</p>
        <p>)</p>
        <p>(<b>svc.reliablerepu.score</b>[golem.attester.thumbprint]=02faf3e...1868)</p>
        <p>)</p>
      </td>
      <td style="text-align:left">7</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>Service constraints:</p>
        <p>filter=(&amp;</p>
        <p>(<b>golem.svc.name</b>=Golem Blender)</p>
        <p>(<b>golem.svc.version</b>=1.1.3)</p>
        <p>(golem.svc.docker.image=golemfactory/blender)</p>
        <p>)</p>
        <p>Requestor properties:</p>
        <p>golem.node.id.name=&#x201D;TheGreat3DStudio, Inc.&#x201D;;</p>
        <p><b>svc.reliablerepu.score</b>=&#x201D;High&#x201D;;</p>
        <p>golem.contract.deposit=0.005;</p>
      </td>
      <td style="text-align:left">
        <p>Service properties:</p>
        <p><b>golem.svc.name</b>=&#x201D;Golem Blender&#x201D;;</p>
        <p><b>golem.svc.version</b>=v&#x201D;1.1.3&#x201D;;</p>
        <p>golem.usage.vector=[&#x201C;golem.usage.duration_sec&#x201D;];</p>
        <p>golem.inf.cpu.cores=4;</p>
        <p>golem.inf.mem.gib=64;</p>
        <p>golem.inf.gpu.vendor=&#x201D;NVidia&#x201D;;</p>
        <p>golem.node.geo.country_code=&#x201D;pl&#x201D;</p>
        <p>golem.node.id.name=&#x201D;UberDatacenter, LLC&#x201D;</p>
        <p>svc.reliablerepu.score=&#x201D;Mid&#x201D;</p>
        <p>Requestor constraints:</p>
        <p>filter=(&amp;</p>
        <p>(golem.node.geo.country_code&lt;&gt;&#x201D;ru&#x201D;)</p>
        <p>(|</p>
        <p>(<b>svc.reliablerepu.score</b>=Mid)</p>
        <p>(<b>svc.reliablerepu.score</b>=High)</p>
        <p>)</p>
        <p>(<b>svc.reliablerepu.score</b>[golem.attester.thumbprint]=02faf3e...1868)</p>
        <p>)</p>
      </td>
      <td style="text-align:left">3</td>
    </tr>
  </tbody>
</table>

## Specification of Agreement

The main purpose of Demand and Offer specifications is to enable matching of Requestors with Providers. Once a Requestor finds a Provider for their Demand, a “deal” is struck, and the terms of the deal need to be expressed in an “Agreement”.

The Agreement shall be an artifact including following elements:

*  Timestamp
*  Agreement Id
*  Content of Demand
*  Content of Offer
*  Attachments
*  Requestor Signature of the above \(includes signature timestamp\)
* Provider Signature of the above \(includes signature timestamp\)
* Confirmation of receipt by Requestor side \(signed, includes timestamp\)

Implementation-wise, the Agreement shall be signed by both Provider and by Requestor.

The Agreement remains confidential by default, however its content may be disclosed by one of the parties, eg. for the purposes of reconciliation, dispute resolution, etc.

### Agreement Id

Important: Agreement Id must be globally unique. Implementations must consider possible Id replay attacks, etc. Therefore it must be possible to validate the origin of Agreement Id \(eg. by deriving it from issuer’s node Id\)

## Specification of Invoice

After an Agreement is committed, Services are expected to be provided in a form of Activity/ies. The Activities incur liability on Requestor. The Provider issues Invoice artifacts to specify the amount owed by the Requestor as a result of Activity execution.

The Invoice shall be an artifact including following elements:

* Timestamp
* InvoiceId
* AgreementId
* **ActivityId???**
* Usage vector value \(value of billing counters\)
* Amount

Implementation-wise, the Invoice shall be signed by Provider. The Requestor’s signature isn’t really required, as their liability is underwritten by the Invoice.

The Invoice remains confidential by default, however its content may be disclosed by one of the parties, eg. for the purposes of reconciliation, dispute resolution, etc.

## Implementation Aspects

Implementation of aspects described in this paper is an effort requiring multiple, interdependent tasks. This section aggregates all defined and potential tasks in this area.

### Topics pending

* Properties, verification & non-repudiation of props, lifetime \(who generates, where are they visible\)?
  * Who generates the properties? Who “populates” the properties? “Caching” of properties? Property registry? Follow-up -&gt; Application/Service registry
* Application/Service registry?
  * Description \(Svc Descriptor\)
    * Expressed in Golem properties
    * Indicates Infrastructure reqs
    * Indicates Usage vector definition
    * Indicates attributes specific to service type \(eg. “Package definitions” - eg Docker image IDs\)

## References

* OSGi Requirements and Capabilities Model Overview \([link](https://blog.osgi.org/2015/12/using-requirements-and-capabilities.html)\)
* RFC-0112 OSGi Bundle Repository \([download](https://raw.githubusercontent.com/wiki/Rud5G/barchart-service/OBR-RFC-112.pdf)\)
* Overview of LDAP Filters \([link](https://ldap.com/ldap-filters/)\)
* RFC 4515 LDAP String Representation of Search Filters \([link](https://tools.ietf.org/search/rfc4515)\) \(note this requires “Common ABNF Productions” specified in RFC 4512 - [link](https://tools.ietf.org/search/rfc4512#section-1.4)\)
* RFC 4515 Parser implementation using Rust & Nom \([link](https://github.com/dequbed/rfc4515)\)

## Appendix A - Study of Cloud Provider offerings

An extract of selected popular cloud provider service descriptions is listed below:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Category</b>
      </th>
      <th style="text-align:left"><b>Offering</b>
      </th>
      <th style="text-align:left"><b>Resource vector</b>
      </th>
      <th style="text-align:left"><b>Usage vector</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Generic VMs</td>
      <td style="text-align:left">AWS EC2</td>
      <td style="text-align:left">
        <p>[Region;</p>
        <p>OS;</p>
        <p>vCPU;</p>
        <p>RAM;</p>
        <p>ECU;</p>
        <p>Local storage]</p>
      </td>
      <td style="text-align:left">
        <p>[Up-time;</p>
        <p>Data Transfer In;</p>
        <p>Data Transfer Out]</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Azure Virtual Machines</td>
      <td style="text-align:left">
        <p>[Region;</p>
        <p>OS;</p>
        <p>vCPU;</p>
        <p>RAM;</p>
        <p>Temp storage,</p>
        <p>Managed Disk]</p>
      </td>
      <td style="text-align:left">
        <p>[Up-time;</p>
        <p>Storage transactions]</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Google Compute Engine</td>
      <td style="text-align:left">
        <p>[Region;</p>
        <p>OS;</p>
        <p>vCPU;</p>
        <p>RAM]</p>
      </td>
      <td style="text-align:left">
        <p>[Up-time;</p>
        <p>Data Transfer Out]</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Storage</td>
      <td style="text-align:left">AWS EBS</td>
      <td style="text-align:left">
        <p>[Region;</p>
        <p>Storage type;</p>
        <p>Capacity Provisioned;</p>
        <p>Data Volume Stored]</p>
      </td>
      <td style="text-align:left">[Up-time]</td>
    </tr>
    <tr>
      <td style="text-align:left">Azure Managed Disks</td>
      <td style="text-align:left">
        <p>[Region;</p>
        <p>Redundancy class;</p>
        <p>Storage type;</p>
        <p>Capacity Provisioned]</p>
      </td>
      <td style="text-align:left">
        <p>[Up-time;</p>
        <p>Write Ops;</p>
        <p>Container Ops;</p>
        <p>Read ops;</p>
        <p>Read volume;</p>
        <p>Write volume]</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Google Cloud Storage</td>
      <td style="text-align:left">
        <p>[Region;</p>
        <p>Storage type;</p>
        <p>Capacity Provisioned]</p>
      </td>
      <td style="text-align:left">
        <p>[Up-time;</p>
        <p>Data Transfer Out;</p>
        <p>Class A Operations;</p>
        <p>Class B Operations]</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Serverless</td>
      <td style="text-align:left">AWS Lambda</td>
      <td style="text-align:left">[Region???]</td>
      <td style="text-align:left">
        <p>[No of requests;</p>
        <p>Aggregated RAM consumption]</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Azure Functions</td>
      <td style="text-align:left">[Region]</td>
      <td style="text-align:left">
        <p>[No of requests;</p>
        <p>Aggregated RAM consumption]</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Google Cloud Functions</td>
      <td style="text-align:left">
        <p>[Region???;</p>
        <p>CPU clock;</p>
        <p>RAM provisioned]</p>
      </td>
      <td style="text-align:left">
        <p>[No of requests;</p>
        <p>Aggregated RAM consumption;</p>
        <p>Data Transfer Out]</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Application containers</td>
      <td style="text-align:left">Azure App Service</td>
      <td style="text-align:left">
        <p>[Region;</p>
        <p>OS;</p>
        <p>vCPU;</p>
        <p>RAM;</p>
        <p>Storage Capacity]</p>
      </td>
      <td style="text-align:left">[Up-time]</td>
    </tr>
    <tr>
      <td style="text-align:left">Batch</td>
      <td style="text-align:left">Azure Batch</td>
      <td style="text-align:left">
        <p>[Region;</p>
        <p>OS;</p>
        <p>vCPU;</p>
        <p>RAM;</p>
        <p>Temp storage,</p>
        <p>Rendering Application]</p>
      </td>
      <td style="text-align:left">[Up-time]</td>
    </tr>
    <tr>
      <td style="text-align:left">Golem Brass Blender</td>
      <td style="text-align:left">
        <p>[vCPU;</p>
        <p>Max RAM;</p>
        <p>Max Storage;</p>
        <p>Performance Index;</p>
        <p>Concent Status]</p>
      </td>
      <td style="text-align:left">[Task Timeout]*</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

### Notes

Cloud pricing study

**Amazon Web Services**

Generic VMs \(EC2\)

Resource vector: \[Region; OS; vCPU; RAM; ECU; Local storage\]

Billing basis vector: \[Time; Data Transfer In; Data Transfer Out\]

Storage \(EBS\)

Resource vector: \[Region; Storage type; Capacity Provisioned; Data Volume Stored\]

Billing basis vector: \[Time\]

Serverless computing \(AWS Lambda\)

Resource vector: \[Region???\]

Billing basis vector: \[No of requests; Aggregated RAM consumption\]

**Azure**

Generic VMs \(Virtual Machines\)

Resource vector: \[Region; OS; vCPU; RAM; Temp storage, Managed Disk\],

Billing basis vector: \[Up-time; Managed Disk time; Storage transactions\]

Storage \(Managed Disks\)

Resource vector: \[Region; Redundancy class; Storage type; Capacity\],

Price basis vector: \[Write ops x time; Container Ops; Read ops; Read volume; Write volume\]

Application containers \(App Service\)

Resource vector: \[Region; OS; vCPU; RAM; Storage Capacity\]

Price basis vector: \[Up-time\]

Serverless computing \(Azure Functions\)

Resource vector: \[Region\]

Billing basis vector: \[No of requests; Aggregated RAM consumption\]

**Google cloud**

Generic VMs \(Compute Engine\)

Resource vector: \[Region; OS; vCPU; RAM\]

Billing basis vector:

Serverless computing \(Google Cloud Functions\)

Resource vector: \[Region???; CPU clock; RAM provisioned\]

Billing basis vector: \[No of requests; Aggregated RAM consumption; Network traffic volume\]

## Appendix B - Sample property and aspect space expressed in Golem notation

The following is a sample set of properties which was used to illustrate the concept. This is now superseded by Golem Computing Resource Standards.

While the property space is generally unbounded, and any participant of Golem ecosystem is able to extend property set \(ie. introduce and start using new property names\) it is practical to provide guidelines for property semantics and naming conventions.

Possible space of Resource properties is listed below for illustration:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Namespace</b>
      </th>
      <th style="text-align:left"><b>Property</b>
      </th>
      <th style="text-align:left"><b>Type</b>
      </th>
      <th style="text-align:left"><b>Description</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">golem.svc</td>
      <td style="text-align:left">name</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Name of Service/App</td>
    </tr>
    <tr>
      <td style="text-align:left">description.url</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Pointer to human-readable description/specification of the Svc/App?</td>
      <td
      style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">version</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Service/App version</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">usagevector</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">Specification of Usage counters used for eg. billing of the Service/App
        usage</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">payload.type</td>
      <td style="text-align:left">List?</td>
      <td style="text-align:left">Coarse-grained indication of &#x201C;class&#x201D; of the service/app,
        eg. &#x201C;hostDirect&#x201D;, &#x201C;docker&#x201D;,</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">docker.imageid</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">For Docker-based payloads - Id of Docker image...</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">resource.constraints</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Resource Constraints specification required by the Service/App.</td>
      <td
      style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">golem.inf</td>
      <td style="text-align:left">cores</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">No of Cores</td>
    </tr>
    <tr>
      <td style="text-align:left">ram.gb</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">RAM (GB)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">disk.gb</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">Provisioned storage (GB)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">net.band</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">Network bandwidth?</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">net.publicip</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Does this host have public IP?</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">net.publicport.available</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">Does this host have this port publicly available?</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">net.publicport.range</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Express visible port range of the host.</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">net.availability</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">Network availability %</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">cpu.platform</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">CPU platform (x86, ARM&#x2026;)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">cpu.bit</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">CPU bitness (32, 64)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">cpu.features</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">CPU features (eg. SGX)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">gpu.name</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">GPU Model</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">gpu.count</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">GPU card count</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">gpu.vendor</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">GPU vendor (Intel, NVIDIA, AMD)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">gpu.family</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">GPU family (i.e. architecture, f.e. Vega)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">gpu.ram.gb</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">GPU RAM (GB)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">...</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">golem.env</td>
      <td style="text-align:left">country.iso</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">Country code in ISO 3166-1</td>
    </tr>
    <tr>
      <td style="text-align:left">os.name</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">OS Name</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">os.version</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">OS Version</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">vm.platform</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">Virtualization platform</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">container.platform</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">Available container platform (Docker, etc.)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">container.version</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Available container version</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">app.platform</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">Available application platform (JVM, .NET CLR, etc.)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">golem.identity</td>
      <td style="text-align:left">org.name</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Name of organization.</td>
    </tr>
    <tr>
      <td style="text-align:left">golem.contract</td>
      <td style="text-align:left">deposit</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">Deposit of funds (eg. assigned to the Demand by Requestor)</td>
    </tr>
    <tr>
      <td style="text-align:left">concent.required</td>
      <td style="text-align:left">Boolean (string: &#x201C;true&#x201D;;&#x201D;false&#x201D;)</td>
      <td style="text-align:left">Specifies that Provider requires readiness to involve Concent in case
        of dispute</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">payment.scheme</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">
        <p>Upfront, after, pay-as-you-go with interval on specified counters?</p>
        <p>This setting would also determine invoicing patterns.</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">golem.apps</td>
      <td style="text-align:left">blender.perf.index</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">Performance Index in Golem Blender application</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">svc.aws.ec2</td>
      <td style="text-align:left">region</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">AWS EC2 Region</td>
    </tr>
    <tr>
      <td style="text-align:left">ecu</td>
      <td style="text-align:left">Numeric</td>
      <td style="text-align:left">AWS EC2 VM ECU</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">svc.reliablerepu</td>
      <td style="text-align:left">score</td>
      <td style="text-align:left">List</td>
      <td style="text-align:left">Reputation score provided by &#x201C;ReliableRepu&#x201D; (sample 3rd
        party reputation provider).</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">svc.&lt;service provider n namespace&gt;</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">app.&lt;application 1&gt; namespace</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

Possible space of property aspects is listed below for illustration:

| **Namespace** | **Aspect** | **Type** | **Description** |
| :--- | :--- | :--- | :--- |
|  |  |  |  |
| golem.aspect | attester.name | String | Name of Property Attestation Provider |
| attester.thumbprint | String | Thumbprint of a Property Attestation Provider certificate |  |
| proofofwork.difficulty | Integer | This indicates the engine is expected to validate an existing proof of work executed by node, and the work has been of a specified difficulty |  |
| proofofwork.timestamp | ? |  |  |

