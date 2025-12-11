# Frequently Asked Questions about DTS

## What is DTS?

DTS is an API for collections of TEI documents.

## Why does TEI need an API for collections?

DTS provides a standard way for clients to interact with collections of TEI documents. A standard API allows users to access many text collections using the same client software. It also allows editors to publish text collections in a usable way that existing clients can use.

## Where can I download DTS and start using it?

You can’t.  DTS defines the way that programs communicate with each other. An end-user will use this software.  For software developers, there are libraries that support DTS, they are listed [here](../index.html#reference-implementations).

## What can clients do with these documents?

Anything the client knows how to do with textual data. For instance, this data can be presented to users in a reading environment, analyzed for linguistic, literary, and discourse features, or presented in tools that allow users to annotate the text to create useful data.

## Will DTS help make my texts FAIR?

Yes! Publishers of digital text collections can use the DTS API to help them make their textual data Findable, Accessible, Interoperable and Reusable (FAIR).

DTS supports FAIR data practices for textual data by:

* encouraging publishers to use stable persistent identifiers for their texts and their collections
* supporting the use of standard vocabularies for the descriptive metadata
* enabling expression of metadata separate from the textual content itself
* providing documented (but unconstrained) access to the information about the structure of a textual resource, down to the level of a citation
* enabling detailed specification of relations among textual resources

## What kind of API is DTS?

DTS is a REST API that works a lot like a web browser.  When client software makes a request, the server responds by giving it a document.  The client can use the information in this document to make further requests. The API is defined purely in terms of Web requests using HTTP and the documents and headers that the server provides in response to those requests.  This means that DTS is language independent, easy to debug, and scales well to large numbers of users. For API geeks, here are the buzzwords: DTS is a pure hypermedia-centric REST API, defined in terms of HTTP conventions.

DTS is built like you would build a website: everything is discoverable, sorted so that users (clients) can easily find what they want. On top of that, it uses vocabularies that are linked and shared and which you can find all over the web of data.

## What is Hydra?  Why Hydra?

Hydra provides a good framework for building REST APIs.  We wanted to use a standard instead of starting from scratch. We wanted good support for JSON and pure hypermedia-based APIs. Hydra provides core functionality and provides extensibility that allows us to customize to fit our model.  (We tried three or four other approaches before choosing Hydra.  We know there are religious debates about APIs, this one worked well for our use cases.)

## If I implement the DTS API for my text collection what does it enable?

Implementing the DTS API enables consumers of the data to easily retrieve:

* Lists of collection members
* Metadata about individual collection items
* Lists of citeable passages within a text
* Lists of citeable passages within a text as groups of client-defined sizes (e.g. groups of 10 lines)
* Metadata about the citation structure of a document
* A single text passage at any level of the citation hierarchy
* A range of text passages with a clearly defined start and end passage
* An entire text

## What identifier schemes does DTS require and support?

DTS supports any identifier scheme for collections and documents which can expressed safely as a URL parameter.

## Does DTS supported nested collections (e.g. collections of collections)?

Yes.

## Does DTS support text citation hierarchies of multiple levels?

Yes.

## Does DTS support text citation hierarchies which vary within a document?

Yes.

## Can I use DTS if I don't publish my texts in TEI/XML?

Yes, partially.

The DTS Collection and Navigations endpoints can be used to provide navigation capabilities across
collections of texts and within a textual document regardless of the data format. However, the Document endpoint only supports
TEI texts.

## What is the relationship between DTS and CTS? Are they redundant?

The Distributed Text Services effort was inspired, informed and influenced by the [Canonical Text Services protocol](http://cite-architecture.github.io/cts/) (CTS). CTS has allowed many classical, canonical texts encoded in TEI to be made available in a machine-actionable, linked open data fashion. However, the CTS API is tightly coupled to the CTS URN identifier system which does not support citation systems used by more modern content or other forms of writing, such as papyri or inscriptions. The API also does not adhere to modern community standards for Web APIs.

DTS was created to address these limitations and enable standardized, machine-actionable operations across a wide variety of texts. DTS is a community-driven initiative that defines a Hypermedia-driven Web Application Programming Interface (API) for working with collections of text as machine-actionable linked data. The DTS specification does not dictate how collections should be organized, what type of persistent identifiers should be used to reference them, what ontologies to use for metadata, how the texts themselves are structured, or how the API itself is to be implemented. It aims to be as generic as possible, providing simple operations for navigating collections, navigating within a text and retrieving textual content. By defining a standard, easily adoptable specification for navigating and interacting with a text collection as machine-actionable data, DTS hopes to provide a standard way to share and reuse collections of textual data.

