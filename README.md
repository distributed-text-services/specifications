# Distributed Text Services (DTS)

[<i class="fa fa-comments"></i> Google group](https://groups.google.com/forum/#!forum/distributed-text-services){: .btn .btn--info .btn--x--small}
[<i class="fa fa-bug"></i> Raise Issues and Ask Questions](https://github.com/distributed-text-services/specifications/issues){: .btn .btn--info .btn--x--small} 
[<i class="fa fa-file-code"></i> Check Source and Propose Changes](https://github.com/distributed-text-services/specifications){: .btn .btn--info .btn--x--small}

*The DTS Specification is currently a First Public Working Draft.*

## Quick Introduction

**The Distributed Text Services (DTS) Specification defines an API for working with collections of text as machine-actionable data.**

It specifies 3 distinct operation endpoints:

- The [Collections Endpoint](Collections-Endpoint.md) is used to navigate the text collection contents
- The [Navigation Endpoint](Navigation-Endpoint.md) is used to navigate within a single text document
- The [Documents Endpoint](Documents-Endpoint.md) is used to retrieve complete or partial texts

The Collections and Navigation endpoints are specified to return  LD+JSON adhering to the [W3C Hydra standard](http://www.hydra-cg.com/spec/latest/core/). The Documents endpoint is specified to return TEI/XML of the requested text or fragment.

Note that DTS is a *specification* for an API and not in and of itself an implementation of that API. Reference Implementations are available (see below)
and individual text publishers are encouraged to implement this API in their own projects where appropriate.

## History

The Distributed Text Services effort has been inspired, informed and influenced by the [Canonical Text Services protocol](http://cite-architecture.github.io/cts/) (CTS). CTS has allowed many classical, canonical texts encoded in TEI to be made available in a machine-actionable, linked open data fashion. However, the CTS API is tightly coupled to the CTS URN identifier system which does not support citation systems used by more modern content or other forms of writing, such as papyri or inscriptions. The API also does not adhere to modern community standards for Web APIs.

To address these limitations and enable standardized, machine-actionable operations across a wide variety of texts, a group of interested scholars and technologists have collaborated to develop the Distributed Text Services specification. We hope this will become a community-supported standard for achieving interoperability
and shared services and tools for working with digital text collections.

The DTS API provides the following core capabilities to clients:

* Retrieve lists of collection members
* Retrieve metadata about individual collection items
* Retrieve lists of citeable passages within a text
* Retrieve lists of citeable passages within a text as groups of client-defined sizes (e.g. groups of 10 lines)
* Retrieve metadata about the citation structure of a document
* Retrieve a single text passage at any level of the citation hierarchy
* Retrieve a range of text passages with a clearly defined start and end passage
* Retrieve an entire text

The DTS API enables its implementors to support:

* any identifier scheme for collections and documents which can expressed safely as a URL parameter
* collections of collections
* text citation hierarchies of multiple levels
* text citation hierarchies which vary within a document

The DTS API imposes the following requirements on implementors:

* Implementation of the Documents endpoint requires the ability to express text as TEI/XML

## Reference Implementations

* The Capitains (https://capitains.org) suite of tools and guidelines will fully support the DTS API. This implementation may be
used by a wide variety of text publishers. It is currently deployed by:
    * The Alpheios Project (https://alpheios.net) at http://texts.alpheios.net/api/dts
    * École des Chartes ( http://dev.chartes.psl.eu/api/nautilus/dts and http://http://dev.chartes.psl.eu/dts-demo/ for a small interface based on it)

* Beta maṣāḥǝft: Manuscripts of Ethiopia and Eritrea (http://betamasaheft.eu/)

