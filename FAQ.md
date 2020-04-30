## Frequently Asked Questions about DTS

**Q. How does DTS support FAIR data practices?**

DTS supports FAIR data practices for textual data by:

* encouraging publishers to use stable persistent identifiers for their texts and their collections
* supporting the use of standard vocabularies for the descriptive metadata
* enabling expression of metadata separate from the textual content itself
* providing documented (but unconstrained) access to the information about the structure of a textual resource, down to the level of a citation 
* enabling detailed specification of relations among textual resources

**Q. If I implement the DTS API for my text collection what does it enable?**

Implementing the DTS API enables consumers of the data to easily retrieve:

* Lists of collection members
* Metadata about individual collection items
* Lists of citeable passages within a text
* Lists of citeable passages within a text as groups of client-defined sizes (e.g. groups of 10 lines)
* Metadata about the citation structure of a document
* A single text passage at any level of the citation hierarchy
* A range of text passages with a clearly defined start and end passage
* An entire text

**Q. What identifier schemes does DTS require and support?**

DTS supports any identifier scheme for collections and documents which can expressed safely as a URL parameter.

**Q. Does DTS supported nested collections (e.g. collections of collections)?**

Yes.

**Q. Does DTS support text citation hierarchies of multiple levels?**

Yes.

**Q. Does DTS support text citation hierarchies text citation hierarchies which vary within a document?**

Yes.

**Q. Can I use DTS if I don't publish my texts in TEI/XML?**

Yes, partially.

The DTS Collections and Navigations endpoints can be used to provide navigation capabilities across 
collections of texts and within a textual document regardless of the data format. However, the Document endpoint only suppoorts
TEI texts.**

