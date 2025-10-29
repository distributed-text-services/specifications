# Distributed Text Services (DTS)

[<i class="fa fa-comments"></i> Google group](https://groups.google.com/forum/#!forum/distributed-text-services){: .btn .btn--info .btn--x--small}
[<i class="fa fa-bug"></i> Raise Issues and Ask Questions](https://github.com/distributed-text-services/specifications/issues){: .btn .btn--info .btn--x--small}
[<i class="fa fa-file-code"></i> Check Source and Propose Changes](https://github.com/distributed-text-services/specifications){: .btn .btn--info .btn--x--small}

## Distributed Text Services 1.0 Release Candidate Published – July 11, 2025

After years of development, independent implementations, interoperability testing, and a public comment period, the Distributed Text Services Technical Committee is pleased to announce the publication of the Distributed Text Services 1.0 Release Candidate (RC1).

We strongly encourage you to implement DTS now and share your feedback. The specification is feature-complete and stable, and we expect only minor clarifications before the final 1.0 release in three months (11/10/2025). Early adoption will ensure your projects are ready and help the community validate the standard in real-world use.

## What is DTS?

**The Distributed Text Services (DTS) Specification defines an API for working with collections of text as machine-actionable data.**

Publishers of digital text collections can use the DTS API to help them make their textual data Findable, Accessible, Interoperable and Reusable (FAIR).

DTS enables _machine-consumption_ of digital text collections, and can be used by consumers of these collections in a variety of ways, such as for data analysis and the development of user-interfaces, tools and services.

DTS is a _specification_ for an API and not in and of itself an implementation of that API. Reference Implementations are available (see below) and individual text publishers are encouraged to implement this API in their own projects where appropriate.

See the [FAQ](FAQ.html) for more information and a list of frequently asked questions about DTS.

## Supported Operations

DTS specifies 3 distinct operation endpoints:

- Navigation _across texts_ is supported by the [Collection Endpoint](versions/1.0rc1/#collection-endpoint)
- Navigation _within a text_ is supported by the [Navigation Endpoint](versions/1.0rc1/#navigation-endpoint)
- Retrieval of complete or partial texts is supported by the [Document Endpoint](versions/1.0rc1/#document-endpoint)

The Collection and Navigation endpoints return JSON-LD. The Document endpoint is specified to return TEI/XML of the requested text or fragment.

## Capabilities

The DTS API provides the following core capabilities to clients:

* Retrieve lists of collection members
* Retrieve metadata about individual collection items
* Retrieve lists of citeable passages within a text
* Retrieve metadata about the citation structure of a document
* Retrieve a single text passage at any level of the citation hierarchy
* Retrieve a range of text passages with a clearly defined start and end passage
* Retrieve an entire text

## Generic Implementations

This list includes implementations that should be generic enough to be compatible with a lot of different corpora.

* [MyDapytains](https://github.com/distributed-text-services/MyDapytains) is a python implementation by a member of the technical committee which uses Flask for web and SaxonC implementation for XPath, XSLT and XQuery.
* [DoTS](https://github.com/chartes/dots) is an implementation from Biblissima + & the Ecole nationale des chartes built on top of BaseX.
* [SCDH/dts-transformations](https://github.com/SCDH/dts-transformations) is a XSLT-based implementation based on CiteStructure for document and navigation endpoints.

## Conformance Evaluation

Matteo Romanello built a toolkit to evaluate implementation against the specifications: <https://github.com/mromanello/DTS-validator>.

## Known Corpora Accessible via the DTS API

The following open access libraries implement the DTS API :

* (1.0rc1) CoMMA <https://comma.inria.fr>
* (1.0rc1) Dracor <https://dev.dracor.org/api/v1/dts>
* (1.0-alpha) Ecole Nationale des Chartes <https://dots.chartes.psl.eu/demo/api/dts/>
  * A small collection of contemporaneous and medieval French literature. The contemporaneous texts are lightly marked up, and the medieval texts are finely annotated.  Uses the MyCapytain/Nautilus libraries.
* (1.0 Draft) Alpheios <http://texts.alpheios.net/api/dts>
  * A small collection of Latin and Greek texts that have been aligned with linguistic annotations for learning ancient languages. Uses the MyCapytain/Nautilus libraries.
* (1.0 Draft) Perseids <https://dts.perseids.org/>
  * Serves all textual resources available from Perseus within the Ancient Greek and Latin corpora as well as some resources in Hebrew and Farsi.
* (1.0 Draft) Beta maṣāḥǝft <http://betamasaheft.eu/>
  * Collection of written artefacts from the highlands of Ethiopia and Eritrea mainly in Gǝʿǝz (Classical Ethiopic). In the collection are present both transcriptions of manuscripts and editions of textual units. The scarce availability of transcriptions as well as available editions means that the actual text contents are few in comparison with the textual units and written artefacts identified and described.
* (1.0 Draft) Epigraphische Datenbank Heidelberg <https://edh-www.adw.uni-heidelberg.de/api/dts/>
  * A corpus of 80,000 short texts from the Latin epigraphic databases.

