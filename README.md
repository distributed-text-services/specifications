# Distributed Text Services (DTS)

## What

> The DTS Specification is currently in Candidate Recommendation Status.

The Distributed Text Services (DTS) Specification defines a Hypermedia-Driven Web API for working with collections of text as machine-actionable data.
It specifies 3 distinct operation endpoints:

- The [Collection Endpoint](Collection-Endpoint.md) is used to navigate the text collection contents (e.g. to support collection browsing)
- The [Navigation Endpoint](Navigation-Endpoint.md) is used to navigate within a single text document (e.g. to support table of contents browsing)
- The [Document Endpoint](Document-Endpoint.md) is used to retrieve complete or partial texts

The Collection and Navigation endpoints return JSON adhering to the [W3C Hydra standard](http://www.hydra-cg.com/spec/latest/core/). The Document returns TEI/XML of the requested text or fragment.

## Why

The Distributed Text Services effort has been inspired, informed and influenced by the [Canonical Text Services protocol](http://cite-architecture.github.io/cts/). The Canonical Text Services (CTS) protocol has allowed many classical, canonical texts encoded in TEI to be made available in a machine-actionable, linked open data fashion. However,  CTS is tightly coupled to its text identifier system, which is not adaptable to the citation systems used by more modern content or other forms of writing, such as papyri or inscriptions. The CTS API also did not meet all community needs related to scalability of response formats and definition of response MIME types. These issues have limited its application outside of the set of classical texts for which it was originally designed.

To address these limitations and enable standardized, machine-actionable operations across a wide variety of texts, a group of interested scholars and technologists have collaborated to develop the Distributed Text Services specification. We hope this will become a community-supported standard for achieving interoperability
and shared services and tools for working with digital text collections.

The DTS API provides the following core functionality to clients:

* Retrieve lists of collection members
    * collections may be in nested hierarchies of collections
* Retrieve metadata about individual collection items
* Retrieve lists of citeable passages within a text
    * citation hierarchies may contain multiple levels
    * citation hierarchies may vary within a document
    * citation hierarchies may be requested as groups of client-defined sizes (e.g. groups of 10 lines)
* Retrieve metadata about the citation structure of a document
* Retrieve a single text passage at any level of the citation hierarchy
* Retrieve a range of text passages with a clearly defined start and end passage
* Retrieve an entire text


The DTS API imposes the following requirements on implementors:

* Implementations are not required to support all 3 endpoints. The Navigation and Document endpoints are optional (although the Collection endpoint is of limited use without the other two).
* Collections and documents may use any identifier scheme which can expressed safely as a URL parameter
* Implementation of the Document endpoint requires the ability to express text as TEI/XML

## Who

The following individuals have contributed to this effort.

Bridget Almas

Hugh Cayless

Thibault Clérice

Zachary Fletcher

Vincent Jolivet

Pietro Liuzzo

Emmanuelle Morlock

Jonathan Robie

Matteo Romanello

James Tauber

Jeffrey C. Witt

## Reference Implementations

* The Capitains (https://capitains.org) suite of tools and guidelines will fully support the DTS API. This implementation will be deployed at https://texts.alpheios.net/api/cts

* ...

## How to get involved

Send a request to join https://groups.google.com/forum/#!forum/distributed-text-services


