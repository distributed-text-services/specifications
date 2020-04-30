# Frequently Asked Questions about DTS

## How does DTS support FAIR data practices?

DTS supports FAIR data practices for textual data by:

* encouraging publishers to use stable persistent identifiers for their texts and their collections
* supporting the use of standard vocabularies for the descriptive metadata
* enabling expression of metadata separate from the textual content itself
* providing documented (but unconstrained) access to the information about the structure of a textual resource, down to the level of a citation 
* enabling detailed specification of relations among textual resources

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

## Is DTS software that is ready to use?

DTS is a _specification_ for an API and not in and of itself an implementation of that API. Reference Implementations are available and individual text publishers are encouraged to implement this API in their own projects where appropriate.

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

The DTS Collections and Navigations endpoints can be used to provide navigation capabilities across 
collections of texts and within a textual document regardless of the data format. However, the Document endpoint only suppoorts
TEI texts.

## What is the relationship between DTS and CTS? Are they redundant?

The Distributed Text Services effort was inspired, informed and influenced by the [Canonical Text Services protocol](http://cite-architecture.github.io/cts/) (CTS). CTS has allowed many classical, canonical texts encoded in TEI to be made available in a machine-actionable, linked open data fashion. However, the CTS API is tightly coupled to the CTS URN identifier system which does not support citation systems used by more modern content or other forms of writing, such as papyri or inscriptions. The API also does not adhere to modern community standards for Web APIs.

DTS was created to address these limitations and enable standardized, machine-actionable operations across a wide variety of texts. DTS is a community-driven initiative that defines a Hypermedia-driven Web Application Programming Interface (API) for working with collections of text as machine-actionable linked data. The DTS specification does not dictate how collections should be organized, what type of persistent identifiers should be used to reference them, what ontologies to use for metadata, how the texts themselves are structured, or how the API itself is to be implemented. It aims to be as generic as possible, providing simple operations for navigating collections, navigating within a text and retrieving textual content. By defining a standard, easily adoptable specification for navigating and interacting with a text collection as machine-actionable data, DTS hopes to provide a standard way to share and reuse collections of textual data.

