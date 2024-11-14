# Distributed Text Services (DTS)

[<i class="fa fa-comments"></i> Google group](https://groups.google.com/forum/#!forum/distributed-text-services){: .btn .btn--info .btn--x--small}
[<i class="fa fa-bug"></i> Raise Issues and Ask Questions](https://github.com/distributed-text-services/specifications/issues){: .btn .btn--info .btn--x--small}
[<i class="fa fa-file-code"></i> Check Source and Propose Changes](https://github.com/distributed-text-services/specifications){: .btn .btn--info .btn--x--small}

*The DTS Specification is currently in a public comment period following the 1-alpha release.*

## What is DTS?

**The Distributed Text Services (DTS) Specification defines an API for working with collections of text as machine-actionable data.**

Publishers of digital text collections can use the DTS API to help them make their textual data Findable, Accessible, Interoperable and Reusable (FAIR).

DTS enables _machine-consumption_ of digital text collections, and can be used by consumers of these collections in a variety of ways, such as for data analysis and the development of user-interfaces, tools and services.

DTS is a _specification_ for an API and not in and of itself an implementation of that API. Reference Implementations are available (see below) and individual text publishers are encouraged to implement this API in their own projects where appropriate.

See the [FAQ](FAQ.html) for more information and a list of frequently asked questions about DTS.

## Supported Operations

DTS specifies 3 distinct operation endpoints:

- Navigation _across texts_ is supported by the [Collection Endpoint](versions/unstable/#collection-endpoint)
- Navigation _within a text_ is supported by the [Navigation Endpoint](versions/unstable/#navigation-endpoint)
- Retrieval of complete or partial texts is supported by the [Document Endpoint](versions/unstable/#document-endpoint)

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

*[MyDapytains](https://github.com/distributed-text-services/MyDapytains) is a python implementation by a member of the technical committee which uses Flask for web and SaxonC implementation for XPath, XSLT and XQuery.
*[DoTS](https://github.com/chartes/dots) is an implementation from Biblissima + & the Ecole nationale des chartes built on top of BaseX.

### Implementation of older versions (Before 1-alpha)

The following open source libraries implement the DTS API (1.0 Draft):

* [TEI Publisher client](https://teipublisher.com/exist/apps/tei-publisher/doc/blog/tei-publisher-50.xml)
  * Implementation of the DTS API + a Client interface to the API
  * Currently supports browsing collections and document retrieval, but not navigation.
* [MyCapytain](https://github.com/Capitains/MyCapytain/tree/3.0.0) + [Nautlius](https://github.com/Capitains/Nautilus/tree/dts-draft-1)
  * MyCapytain is a Python library that implements the DTS data model and Nautlius is a Python library that implements the DTS API Endpoints backed by the MyCapytain libary. Both operate on TEI text collections that adhere to the [Capitains Guidelines](http://capitains.org/pages/guidelines)
* [Perseids DTS API](https://github.com/perseids-project/dts-api/)
  * Ruby on Rails implementation of the DTS API . Operates on TEI text collections that adhere to the [Capitains Guidelines](http://capitains.org/pages/guidelines)

## Known Corpora Accessible via the DTS API

The following open source libraries implement the DTS API (1.0 Draft):

* Dracor <https://dev.dracor.org/api/v1/dts>
* Ecole Nationale des Chartes <http://dev.chartes.psl.eu/api/nautilus/dts> and <http://http://dev.chartes.psl.eu/dts-demo/>
  * A small collection of contemporaneous and medieval French literature. The contemporaneous texts are lightly marked up, and the medieval texts are finely annotated.  Uses the MyCapytain/Nautilus libraries.
* Alpheios <http://texts.alpheios.net/api/dts>
  * A small collection of Latin and Greek texts that have been aligned with linguistic annotations for learning ancient languages. Uses the MyCapytain/Nautilus libraries.
* Perseids <https://dts.perseids.org/>
  * Serves all textual resources available from Perseus within the Ancient Greek and Latin corpora as well as some resources in Hebrew and Farsi.
* Beta maṣāḥǝft <http://betamasaheft.eu/>
  * Collection of written artefacts from the highlands of Ethiopia and Eritrea mainly in Gǝʿǝz (Classical Ethiopic). In the collection are present both transcriptions of manuscripts and editions of textual units. The scarce availability of transcriptions as well as available editions means that the actual text contents are few in comparison with the textual units and written artefacts identified and described.
* Epigraphische Datenbank Heidelberg <https://edh-www.adw.uni-heidelberg.de/api/dts/>
  * A corpus of 80,000 short texts from the Latin epigraphic databases.

