# Document Endpoint

The Document endpoint is used to access and modify the content of document, as opposed to metadata (which is found in collections). Implementations must at least support reading document using `GET` requests as described below. The other request methods (POST, PUT, DELETE) are not required and they may be implemented selectively. For example, an implementation could allow `PUT` operations to update the contents of a document, but not support the total removal of document sections using `DELETE`. If an implementation is going to allow modification of document contents please do so using these methods as described below. An implementation should always document clearly which methods are supported for this endpoint. This should be included both in the human-readable API guidelines and in the endpoint's machine-readable documentation using the `supportedOperation` term. (See the section [Machine Readable Endpoint Documentation](#machine-readable-documentation-of-supported-methods) below.)

## Default Request and Response Body Format

Implementations of the DTS Document endpoint **must**, at minimum, return textual data in an XML format compliant with the Text Encoding Initiative (TEI) guidelines (`application/tei+xml` response format). The XML must be well formed and valid. This should be the default response format.

Implementations **may** return requested data in as many other formats as the content provider wishes. Other formats should not be included in the default response, but should be returned (one format per request) when an alternate format is specified in the request.

The root node of the XML response must be the `<TEI>` root element of the namespace `http://www.tei-c.org/ns/1.0`. So a response would normally look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <!-- XML of the document requested here -->
</TEI>
```
If the request returns a complete document, it is returned directly inside this `<TEI>` element.

If the data to be returned is not a complete document, the textual data should be embedded in the `<fragment>` element of the DTS Namespace (`https://w3id.org/dts/api#`) like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <!-- XML or string of the passage requested here -->
  </dts:fragment>
</TEI>
```
There is no limitation as to what can be contained by `dts:fragment` or what its siblings can be provided they are well formed and valid. The only limiting factor is that `dts:fragment` should contain the requested textual data. This permits an implementation to return contextual material elsewhere within the root `TEI` node alongside the requested fragment. Note, though, that most metadata should be accessed through the `collection` endpoint instead.

For requests that include a document as the request body (`PUT` and `POST`) this request body is not required to follow these same guidelines. It is **recommended**, though, that implementations do accept TEI-compliant XML documents and fragments (as described [above](#default-request-and-response-body-format)) as the default. Implementations **may** also allow a request body in any other format, but only when a format is specified via the `format` query parameter.

## Preserving the Integrity of Edited Documents

Implementations of the DTS specification need not allow any write, update, or delete operations. If they do, however, this raises questions around how to maintain the integrity of the document on the server.

### Validating Changes to Documents

Because of its flexibility, at some points this specification relies on clients to ensure that the textual data they submit is correctly structured. This requires more than simply that clients submit valid XML when creating or editing a text segment. It is strongly recommended that servers implementing these methods offer robust validation of any submitted text. For servers using TEI XML in requests, this should include validating that the submitted text or fragment

- is well-formed XML
- satisfies the requirements of the TEI schema
- is consistent with the citation structure of the document being edited
- is consistent with the specific application of the TEI specification being used by the implementing project

Failure to meet any of these criteria should result in a failed request and an error response that pinpoints the problem in the submitted text as specifically as possible.

### Restricting the Scope of Editing Operations

This burden of validation can be mitigated by restricting editing operations to one bottom-level segment of the document at a time. In a document with a three-level structural hierarchy of book, paragraph, and line, this would mean that only one line could be edited or deleted at a time. Such a restriction would allow the server to easily control the XML document structure. It would also, though, make some larger-scale editing operations unwieldy.

This specification leaves the choice to enforce editing restrictions like this up to the implementer. If such restrictions are imposed, any request that is rejected as a result must be accompanied by a clear error message. The error message must (a) explain the restriction, and (b) provide a URL where the restriction is documented.

## URI

### Query Parameters

The Document endpoint supports the following query parameters:

| Name | Description                              | Methods |
|------|------------------------------------------|---------|
| id	| (Required) Identifier for a document. Where possible this should be a URI	| GET, POST, PUT, DELETE |
| ref   | Passage identifier (used together with `id`; can’t be used with `start` and `end`)	| GET, PUT, DELETE |
| start	| (For range) Start of a range of passages (can’t be used with `ref`)	| GET, PUT, DELETE |
| end	| (For range) End of a range of passages (requires `start` and no `ref`)	| GET, PUT, DELETE |
| after | (Optional) Passage after which the new segment should be inserted | POST |
| before | (Optional) Passage after which the new segment should be inserted | POST |
| token	 | (May be required by implementation) Authentication token for access control	| POST, PUT, DELETE |
| format | (Optional) Specifies a data format for response/request body other than the default	| GET, POST, PUT, DELETE |

Note that for GET requests one may either provide a `ref` parameter __or__ a pair of `start` and `end` parameters. A request cannot combine `ref` with the other two. If, say, a `ref` and a `start` are both provided this should cause the request to fail.

Two parameters are used only when creating new document sections using a POST request: `after` and `before`. These allow one to specify where a new text segment should be inserted. One or the other of `after` and `before` must be included in a POST request unless one is creating the first text segment in a new document.

Where the implementation needs to control who can access, create, or modify server data, the `token` parameter also allows for token-based authentication (as with OAuth 2.0) if desired. It is up to the implementation to decide how such tokens should be generated and processed.

### URI Template

Here is a template of the URI for the Document API. The route itself (here `/dts/api/document/`) is up to the implementer.

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  },
  "@type": "IriTemplate",
  "template": "/dts/api/document/?id={document_id}&dts:ref={dts:ref}&start={start}&end={end}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "collection_id",
      "required": true
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "dts:ref",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "start",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "end",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "after",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "before",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "token",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "format",
      "required": false
    }
  ]
}
```

## Machine-Readable Endpoint Documentation

Machine-readable documentation for the endpoint should be made available in JSON-LD format following the Hydra Core Vocabulary specification. In this documentation the "supportedOperations" term must be used to specify which http methods the implementation supports for this endpoint. The value for "supportedOperations" should be an array of Hydra `Operation` objects, one object for each supported http method.

## Batch Requests

For the sake of simplicity and usability the DTS guidelines do not allow for batch operations, i.e. fetching or altering more than one passage of text in the same request. Implementations should encourage client software and other consumers of their API to use multiple asynchronous requests instead.

## GET Requests on the Document Endpoint

### GET query parameters

The only strictly required parameter for a `GET` Document request is `id`. This value specifies the document to be queried. If no particular reference is specified with other parameters, the response should return the entire document. If only one structural segment of the document is desired, this should be specified using the `ref` parameter. The `ref` value may specify a segment at any structural level of the document. For example, in a document segmented in two structural levels (book and line), a `ref` value of "5" should return all of book 5. A `ref` value of "5.2" should return just line 2 from book 5.

If a continuous sequence of structural segments is being requested, then no `ref` value should be specified. Instead, the `start` and `end` parameters should specify the first and last segments of the sequence to be returned. If only `start` is specified, with no `end` value, the request should be understood as having an implicit `end` parameter specifying the last structural segment of the document. In other words, a request with a `start` value of "5.2" and no explicit `end` value should return all of the document from 5.2 (inclusive) to the end. Likewise, if only an `end` parameter is specified, the server should understand an implicit `start` pointing to the first segment of the document.

As with `ref`, the level of specificity in the `start` and `ref` values should determine the structural level used to demarcate the segments returned. A query with "start=5.2&end=6.1" should return every line from 5.2 to 6.1 (inclusive). A query with "start=5&end=6" should return every line from the beginning of book 5 to the end of book 6 (inclusive).

No request should include both a `ref` parameter and either `start` or `end`. Any request that does include both should prompt a `400(Bad Request)` error.

### GET request body

A `GET` request should not include a body. If one is included it should be ignored.

### GET Responses

#### Status codes

A successful `GET` request to the Document endpoint should return the status code `200(OK)`.

If a `GET` request is unsuccessful because of problems with the request parameters, the return status code should be `400(Bad Request)` or a custom status code in the 4XX series signaling a more specific error.

If a `GET` request is unsuccessful because the specified id matches no document on the server, the return status code should be `404(Not Found)`

#### Successful GET response headers

The response after a successful `GET` request contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives location of api documentation as well as links to neighbouring document segments and other related api endpoints |
| Content-Type | Content type of the response body (by default `application/tei+xml`)|

##### Link header

When applicable, the following links must be provided in the 'Link' header:

| Name of the property | Description of its value |
| -------------------- | ------------------------ |
| http://www.w3.org/ns/hydra/core#apiDocumentation | The URL for the Hydra-compliant machine-readable documentation for this implementation of the Document endpoint |
| prev | Previous passage of the document in the Document endpoint |
| next | Next passage of the document in the Document endpoint |
| up | Parent passage of the document in the Document endpoint. If the current request is already for the entire document, no `up` link will be provided. If the only parent is the entire document, the `up` value will link to the document as a whole. |
| first | First passage of the document in the Document endpoint  |
| last | The URL for the last passage of the document in the Document endpoint |
| contents | The URL for the Navigation Endpoint for the current document |
| collection | The URL for the Collection endpoint for the current document |

#### Successful response body

The response body after a successful `GET` request should contain an XML document representing the requested text segment. This XML should be formatted as described above under "[Default Request and Response Body Format](#default-request-and-response-body-format)".

#### Unsuccessful response headers

An unsuccessful `GET` request should return the following headers:

| name | description |
|------|-------------|
| Link | The URL for the Hydra-compliant machine-readable `Document` endpoint documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation |
| Content-type | Content type of the response body (by default `application/tei+xml`) |

#### Unsuccessful response body

If a `GET` response returns an error code, the response body should contain an XML object following the DTS specification for XML status responses (https://w3id.org/dts/api). For example, this would be a well-formed response body for a `400(Bad Request)` error:

```xml
<error statusCode="400" xmlns="https://w3id.org/dts/api">
  <title>Invalid request parameters</title>
  <description>The query parameters were not correct.</description>
</error>
```

For a `400(Bad Request)` error, the `<description>` should provide as much information as possible about which submitted data was missing or unacceptable. It should indicate whether:
- there were missing required query parameters
- any query parameters held unacceptable values, such as a document id that does not match any document on the server
- the query parameters were acceptable but no such section exists for the requested document

It is strongly recommended that projects implement more specific 4XX-series error responses to handle specific  errors. In such cases the `<description>` of the error response should include both an explanation of the issue and the URL for the necessary documentation.

### GET Example 1: Retrieve a passage using `ref`

Retrieve the passage `2` of the document labeled by the identifier `https://papyri.info/ddbdp/bgu;11;2029/source`.

#### GET request url

- GET `/dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source&ref=2`

#### Successful GET response status

- 200(OK)

#### Successful GET response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation", </dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source&ref=1>; rel="prev", </dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source&ref=3>; rel="next", </dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source&ref=6>; rel="last", </dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source>;rel="up", </dts/api/navigation/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="contents", </dts/api/collection/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="collection" |

#### Successful GET response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <teiHeader>
      <fileDesc>
         <titleStmt>
            <title>bgu.11.2029</title>
         </titleStmt>
         <publicationStmt>
            <authority>Duke Collaboratory for Classics Computing (DC3)</authority>
            <idno type="filename">bgu.11.2029</idno>
            <idno type="ddb-perseus-style">0001;11;2029</idno>
            <idno type="ddb-hybrid">bgu;11;2029</idno>
            <idno type="HGV">9566</idno>
            <idno type="TM">9566</idno>
            <availability>
               <p>© Duke Databank of Documentary Papyri. This work is licensed under a <ref type="license" target="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 License</ref>.</p>
            </availability>
         </publicationStmt>
      </fileDesc>
   </teiHeader>
   <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <lb n="1"/><expan>τετελ<ex>ώνηται</ex></expan> <expan>δι<ex>ὰ</ex></expan> <expan>πύλ<ex>ης</ex></expan> Διονυσιάδος
   </dts:fragment>
</TEI>
```

### GET Example 2: Retrieve a passage using `start` and `end`

Retrieve the passages 1.1.1 to the passage 1.1.2 of the document labeled by the identifier `urn:cts:latinLit:phi1318.phi001.perseus-lat1`.

#### GET request url

- GET `/dts/api/document/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.1.1&end=1.1.2`

#### Successful GET response status

- 200(OK)

#### Successful GET response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation", </dts/api/document/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.2.1&end=1.2.2>; rel="next", </dts/api/document/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=5.5.5&end=5.5.6>; rel="last", </dts/api/navigation/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&ref=1.2>; rel="up", </dts/api/navigation/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1>; rel="contents", </dts/api/collection/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1>; rel="collection" |

#### Successful GET response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
       <text xml:lang="lat">
          <body>
             <div type="edition" xml:lang="lat" n="urn:cts:latinLit:phi1318.phi001.perseus-lat1">
                <div type="textpart" n="1" subtype="book">
                   <div type="textpart" n="1" subtype="letter">
                      <div type="textpart" n="1" subtype="section">
                         <p>Frequenter hortatus es, ut epistulas, si quas paulo curatius scripsissem, colligerem publicaremque. Collegi non servato temporis ordine - neque enim historiam componebam -, sed ut quaeque in manus venerat.</p>
                      </div>
                      <div type="textpart" n="2" subtype="section">
                         <p>Superest ut nec te consilii nec me paeniteat obsequii. Ita enim fiet, ut eas quae adhuc neglectae iacent requiram et si quas addidero non supprimam. Vale.</p>
                      </div>
                   </div>
                </div>
             </div>
          </body>
       </text>
   </dts:fragment>
</TEI>
```

### GET Example 3: Retrieve a full document

Retrieve the full document labeled by the identifier `https://papyri.info/ddbdp/bgu;11;2029/source`

#### GET request url

- GET `/dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source`

#### Successful GET response status

- 200(OK)

#### Successful GET response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation", </dts/api/navigation/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="contents", </dts/api/collection/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="collection" |

#### Successful GET response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-model href="http://www.stoa.org/epidoc/schema/8.16/tei-epidoc.rng" type="application/xml" schematypens="http://relaxng.org/ns/structure/1.0"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0"
     n="0001;11;2029"
     xml:id="bgu.11.2029"
     xml:lang="en">
   <teiHeader>
      <fileDesc>
         <titleStmt>
            <title>bgu.11.2029</title>
         </titleStmt>
         <publicationStmt>
            <authority>Duke Collaboratory for Classics Computing (DC3)</authority>
            <idno type="filename">bgu.11.2029</idno>
            <idno type="ddb-perseus-style">0001;11;2029</idno>
            <idno type="ddb-hybrid">bgu;11;2029</idno>
            <idno type="HGV">9566</idno>
            <idno type="TM">9566</idno>
            <availability>
               <p>© Duke Databank of Documentary Papyri. This work is licensed under a <ref type="license" target="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 License</ref>.</p>
            </availability>
         </publicationStmt>
         <sourceDesc>
            <p/>
         </sourceDesc>
      </fileDesc>
      <profileDesc>
         <langUsage>
            <language ident="en">English</language>
            <language ident="grc">Greek</language>
         </langUsage>
      </profileDesc>
      <revisionDesc>
          <change when="2011-12-14" who="http://papyri.info/editor/users/gabrielbodard">rationalized languages in langUsage</change>
          <change when="2011-12-14" who="http://papyri.info/editor/users/gabrielbodard">changed editor names to URIs</change>
          <change when="2010-05-05" who="http://papyri.info/editor/users/gabrielbodard">changed schema; added xml:space=preserve; indented; moved title/@n to idno</change>
          <change when="2009-11-12" who="http://papyri.info/editor/users/gabrielbodard">Added language la-Grek</change>
          <change when="2009-06-27" who="http://papyri.info/editor/users/gabrielbodard">Converted from TEI P4 (EpiDoc DTD v. 6) to P5 (EpiDoc RNG schema)</change>
          <change when="2008-12-22" who="http://papyri.info/about">Automated split from transcoder files</change>
      </revisionDesc>
   </teiHeader>
   <text>
      <body>
         <head n="9566" xml:lang="en">
            <date>AD161-9</date>
            <placeName>Dionysias</placeName>
         </head>
         <div xml:lang="grc" type="edition" xml:space="preserve"><ab>
    <lb n="1"/><expan>τετελ<ex>ώνηται</ex></expan> <expan>δι<ex>ὰ</ex></expan> <expan>πύλ<ex>ης</ex></expan> Διονυσιάδος

    <lb n="2"/>λιμένος Μέμφεως Ζώσιμος

    <lb n="3"/><expan><supplied reason="lost">ἐ</supplied><unclear>ξ</unclear>ά<ex>γων</ex></expan> <supplied reason="lost"><expan>ἐ<ex>πὶ</ex></expan> </supplied><expan><supplied reason="lost">κα</supplied>μ<ex>ή</ex>λ<ex>οις</ex></expan> δυσὶ <num value="2"/> ἐλαίου <expan>μετ<ex>ρητὰς</ex></expan> ἐννέα <num value="9"/>

    <lb n="4"/><gap reason="lost" quantity="4" unit="character"/><unclear>ρ</unclear>αχ<add place="above">ε</add> τεσσεράκοντα πέντε <num value="45"/>

    <lb n="5"/><supplied reason="lost"><expan><ex>ἔτους</ex></expan> </supplied><gap reason="lost" quantity="1" unit="character"/><supplied reason="lost"> Ἀντ</supplied>ωνείνου καὶ Οὐήρου τῶν κυρίων

    <lb n="6"/><supplied reason="lost">Σεβασ</supplied>τῶν Μεσορὴ ἑκκαιδεκάτῃ. </ab></div>
      </body>
   </text>
</TEI>

```

## POST Requests on the Document Endpoint

The `POST` method of the Document endpoint allows for creation of new textual passages in a resource. __This method requires that the document already has a metadata record accessible via the Collection endpoint__.

Note that this method should __not__ be used to modify existing segments of text. In other words, if the specified reference label already exists in the document on the server, this method should return an error and refuse to perform the requested operation. If, for example, a document already has a line 6 following the existing line 5, then `POST` cannot be used to insert new or added text in that existing line 6. Modification of the text in existing document segments must be done using the `PUT` method instead.

### POST Query parameters

The only strictly required parameters for a `POST` Document request are `id` and (if supported) `token`. If neither `before` nor `after` is supplied, the server should interpret the request as supplying the initial form of a new document. In that case, the request should fail if (a) some text already exists for that document, or (b) the request body contains a `<dts:fragment>` element.

In most cases, though, the request will be inserting a new structural segment into a document that already contains some segments. In such instances, the query must include either a `before` or an `after` parameter to indicate where the new text segment will be inserted.

**Neither the `ref` parameter nor the pair of `start`/`end` parameters should be provided in a `POST` Document request. The reference information for the inserted segment(s) should be drawn on the server side from the text in the request body. If an implementation follows the DTS recommendation to accept POST data in TEI XML format, then the reference information should be extracted on the server side from the standard TEI attributes in the XML of the request body.** This is to avoid the possibility of conflicts in which the embedded referencing in the document body is different than the reference information provided through URL parameters.

Note that the `after` or `before` reference should also determine the depth of the insertion in the document's citation structure. The lowest structural level specified in the `after` or `before` reference is the level at which the contents of the XML `<dts:fragment>` will be inserted. In other words, the new segment will be taken to be a sibling to the segment specified in that reference.

If the implementation accepts a request body in any format other than TEI-compliant XML, this must be clearly and prominently communicated in the API documentation. If the implementation accepts multiple different data formats in the response body, one should be designated as the default. (Again, it is recommended but not required that the default be TEI-compliant XML.) Any `POST` request that uses a supported format other than the implementation's default must use the `format` parameter to indicate how its body is encoded.

### POST Request Body

When XML is being used as the default `POST` format, the request body must be a properly formed `<TEI>` root node containing TEI-compliant XML. See the further specifications under "[Default Request and Response Body Format](#default-request-and-response-body-format)" above.

Note that in most cases the actual text to be inserted is the __contents__ of the inner `<fragment>` element in the request body. The only exception to this rule is where no structural segment has yet been created for a document, in which case the entire `<TEI>` rootnode (along with all its contents) will be accepted as the initial form of the document.

### POST Responses

#### Status codes

A successful `POST` request to the Document endpoint should return the status code `201(Created)`.

If a `POST` request is unsuccessful because the specified id does not match any document on the server, the return status code should be "404(Not Found)". If a `POST` request is unsuccessful because of other problems with the request content (i.e., parameters or request body), then the return status code should be "400(Bad Request)" or a custom status code in the 4XX series signaling a more specific error.

A `POST` request should return "409(Conflict)" if it would create a new initial form of a document for which some segments already exist (even if those segments are empty). This would happen when no `after` or `before` parameter is supplied, but at least one segment of the document already exists on the server (even if it contains only an empty string).

A `POST` Document request also may not result in a text segment whose reference identifier is the same as that of an existing segment. If, for example, a document already includes a segment "4.23.8", then a `POST` request which would result in a *second* segment "4.23.8" should fail and return "409(Conflict)".

#### Successful response headers

The response headers after a successful `POST` Document request should include a `Location` value. This should be the URL where the newly inserted text segment(s) can be retrieved via a `GET` request. The response should also include a `Content-type` header with a value of "application/tei+xml". Finally, a `Link` header should specify the URL for the machine-readable endpoint documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation".

#### Successful response body

The response body after a successful `POST` request should contain an XML object representing the newly created text segment (and any children). This should be identical to the response a client would receive using a `GET` request to the `Location` indicated in the response header. This will also mean that, in implementations using TEI XML, most successful `POST` requests will return an XML document identical to the one originally sent by the client.

#### Unsuccessful response headers

In an unsuccessful request, the response headers should include a `Link` whose value is the URL for the Document endpoint documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation". The response should also include a `Content-type` header with a value of "application/tei+xml".

#### Unsuccessful response body

If a response returns an error code, the response body should contain an XML object following the DTS specification for XML status responses (https://w3id.org/dts/api). For example, this would be a well-formed response body for a `400(Bad Request)` error:

```xml
<error statusCode="400" xmlns="https://w3id.org/dts/api">
  <title>Improperly Formed Request Body</title>
  <description>The body of a `POST` request to the Document endpoint on this server must be properly formed XML. If it is submitting the initial text for a new document, the request body must be a <TEI> rootnode with properly formed and schema-compliant children.</description>
</error>
```

For a `400(Bad Request)` error, the `<description>` should provide as much information as possible about which submitted data was missing or unacceptable. It should indicate whether:
- there were missing required query parameters
- any query parameters held unacceptable values
- the request body did not have properly formed XML
- the request body was properly formed but carried unacceptable values

It is strongly recommended that projects implement more specific 4XX-series error responses to handle more specific validation errors such as
- XML that fails to satisfy the TEI schema
- XML that conflicts with the current document's citation structure
- XML that violates other project-specific conventions for implementing the TEI specification
In such cases the `<description>` of the error response should include both an explanation of the issue and the URL for the necessary documentation.

If a response returns a status of `409(Conflict)` then the XML `<description>` element should contain an indication that the request would result in a duplicate segment in the document's citation structure.

### POST Example 1: Creating the initial text for a new document

In this example we will submit the first structural segments to constitute a new document. Prior to this operation no segments exist for the document on the server. This first `POST` request only creates verses 1-2 of chapter 1 in the document identified by the URN `urn:cts:ancJewLit:1Enoch`.

Note that a metadata record for the document must already have been added via the Collection endpoint. This is the only way to establish a valid identifier for the document, which is then used for the Document `POST` request.

Note too that a `<teiHeader>` element precedes the main `<text>` element. If a `<teiHeader>` is included with the initial form of the document, its metadata should be extracted and integrated with the data accessible via the Collection endpoint.

#### POST request URL

Notice that this request omits the usual `before` or `after` parameters.

- `/api/dts/document/?id=urn:cts:ancJewLit:1Enoch&token=XXXXX`

#### POST request body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
    <text xml:lang="Ethiopic">
      <body>
        <div n="1" xml:id="1En1" type="Chapter">
          <div n="1:1" xml:id="1En1:1" type="Verse">
            <app xml:id="1">
              <rdg wit="#Bertalotto #p"> ቃለ፡​በረከት፡​ዘሄኖከ፡​ዘከመ፡​ባረከ፡​ኅሩያነ፡​ወጻድቃነ፡​እለ፡​ሀለዉ፡​ይኩኑ፡​በዕለተ፡​ምንዳቤ፡​ለአሰስሎ፡​ኵሎ፡​እኩያን፡​ወረሲዓን፡​</rdg>
            </app>
          </div>
          <div n="1:2" xml:id="1En1:2" type="Verse">
            <app xml:id="2">
              <rdg wit="#Bertalotto #p">ወአውሥአ፡​ሄኖክ፡​ወይቤ፡​ብእሲ፡​ጻድቅ፡​ዘእምኀበ፡​እግዚአብሔር፡​እንዘ፡​አዕይንቲሁ፡​ክሡታት፡​ወይሬኢ፡​</rdg>
            </app>
            <app xml:id="3">
              <rdg wit="#p">ራዕየ፡​</rdg>
              <rdg wit="#Bertalotto">ራእየ፡​</rdg>
            </app>
            <app xml:id="4">
              <rdg wit="#Bertalotto #p">ቅዱሰ፡​ዘበሰማያት፡​ዘአርአዩኒ፡​መላእክት፡​ወሰማዕኩ፡​እምኀቤሆሙ፡​ኵሎ፡​ወአእመርኩ፡​አነ፡​ዘእሬኢ፡​ወአኮ፡​ለዝ፡​ትውልድ፡​አላ፡​ለዘ፡​ይመጽኡ፡​ትውልድ፡​ርሑቃን፡​</rdg>
            </app>
          </div>
        </div>
      </body>
    </text>
</TEI>
```

#### Successful POST response status

- 201(Created)

#### Successful POST response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/document?id=urn:cts:ancJewLit:1Enoch |
| Content-Type  | application/tei+xml             |
| Link | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation" |
<!-- FIXME: Should we change the sample urls to something more obviously arbitrary,
like https://www.example.com/exampleapi ? -->

#### Successful POST response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
    <text xml:lang="Ethiopic">
      <body>
        <div n="1" xml:id="1En1" type="Chapter">
          <div n="1:1" xml:id="1En1:1" type="Verse">
            <app xml:id="1">
              <rdg wit="#Bertalotto #p"> ቃለ፡​በረከት፡​ዘሄኖከ፡​ዘከመ፡​ባረከ፡​ኅሩያነ፡​ወጻድቃነ፡​እለ፡​ሀለዉ፡​ይኩኑ፡​በዕለተ፡​ምንዳቤ፡​ለአሰስሎ፡​ኵሎ፡​እኩያን፡​ወረሲዓን፡​</rdg>
            </app>
          </div>
          <div n="1:2" xml:id="1En1:2" type="Verse">
            <app xml:id="2">
              <rdg wit="#Bertalotto #p">ወአውሥአ፡​ሄኖክ፡​ወይቤ፡​ብእሲ፡​ጻድቅ፡​ዘእምኀበ፡​እግዚአብሔር፡​እንዘ፡​አዕይንቲሁ፡​ክሡታት፡​ወይሬኢ፡​</rdg>
            </app>
            <app xml:id="3">
              <rdg wit="#p">ራዕየ፡​</rdg>
              <rdg wit="#Bertalotto">ራእየ፡​</rdg>
            </app>
            <app xml:id="4">
              <rdg wit="#Bertalotto #p">ቅዱሰ፡​ዘበሰማያት፡​ዘአርአዩኒ፡​መላእክት፡​ወሰማዕኩ፡​እምኀቤሆሙ፡​ኵሎ፡​ወአእመርኩ፡​አነ፡​ዘእሬኢ፡​ወአኮ፡​ለዝ፡​ትውልድ፡​አላ፡​ለዘ፡​ይመጽኡ፡​ትውልድ፡​ርሑቃን፡​</rdg>
            </app>
          </div>
        </div>
      </body>
    </text>
</TEI>
```

### POST Example 2: Creating a new section of text in an existing document

In this example we will create a new text segment in the existing document `urn:cts:ancJewLit:1Enoch`. Initially we created the document with just verses 1 and 2 of chapter 1. We will now add a third verse to that same first chapter.

#### POST request URL

- `/api/dts/document?id=urn:cts:ancJewLit:1Enoch&after=1:2&token=XXXXX`

#### POST request body

Note that since some text already exists for this document, the new segment is submitted in a `<dts:fragment>` element.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```

#### Successful POST response status

- `201(Created)`

#### Successful POST response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/document?id=urn:cts:ancJewLit:1Enoch&ref=1:3 |
| Content-Type  | application/tei+xml               |
| Link | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation" |

#### Successful POST response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```

## PUT Requests on the Document Endpoint

The `PUT` method of the Document endpoint allows for modification of existing textual passages in a resource.

Note that this method should __not__ be used to create new structural segments of text. For example, if the citation structure of the document already contains a section "5.7.31" then the `PUT` method may be used to modify that section. If section "5.7.31" does not already exist in the document on the server, an attempt to edit that segment using `PUT` should result in an error response. The `POST` method must first be used to add that segment to the document. Likewise, `PUT` should never result in the deletion of a segment in the document's citation structure. Of course, other XML entities below lowest level of a document's citation structure may be freely added or removed by the `PUT` method.

### PUT Query parameters

The three required parameters for a `PUT` Document request are `id`, `token` (if supported by the implementation), and `ref`. Only one structural segment of the document may be modified in a single `PUT` request.

The `ref` reference determines the structural depth of the modification. Suppose a document has a three-level citation structure represented in XML by nested `<div>` elements. A `PUT` request with a `ref` of "3.16.8" will replace just one `<div>` element at the lowest structural level. If the request is instead sent with a `ref` parameter of "3.16", the submitted `<div>` element will replace the second-level `<div>` with the identifier "3.16". In the latter case, the submitted `<div>` element will need to contain the same series of nested `<div>` elements that represent the existing bottom-level segments in the server's version of the file: "3.16.1", "3.16.2", "3.16.3", etc. It is recommended that implementations check the submitted text before performing upper-level modifications like this, to ensure that no lower-level structural segments are being created or destroyed. Alternately, an implementation may avoid this issue by only allowing `PUT` requests at the lowest level of the document's citation structure.

### PUT Request Body

As with `POST` requests, it is recommended that implementations accept TEI-compliant XML as the default format for the `PUT` request body. This makes particularly good sense since it allows clients to modify and return the same XML document they retrieved by default with a `GET` request. If other data formats are supported (or used as the default) __the same formats must be accepted for `POST` and `PUT` request bodies__.

When XML is accepted, the body of the `PUT` request must be wrapped in a TEI rootnode containing a `<dts:fragment>` entity. See the further specifications under "[Default Request and Response Body Format](#default-request-and-response-body-format)" above.

The contents of this `<dts:fragment>` must be a single XML element (along with its enclosed text and/or children) representing the modified form of the target document section. This single element inside the `<dts:fragment>` will replace the corresponding XML element in the document on the server.

The text/XML submitted in the body of a Document's `PUT` request will completely replace the XML entity representing the identified text segment. So the submitted fragment __must include the outer element__ identified by the "ref" value. If you identify your submitted text as a modified version of line "12", and if each line is represented by a TEI `<l>` element, you would include the opening and closing tags `<l xml:id="12">` and `</l>` around the changed content. By including this outer tag in the `PUT` body we allow modification to be made to that outer element's attributes.

### PUT Responses

#### Status codes

If the specified section of the document is successfully modified, the response status should be `200(OK)`.

If no structural section exists with an identifier matching the `ref` parameter in the request, the response status should be `404(Not Found)`. Similarly, the response should be `404(Not Found)` if the id supplied does not correspond to any document on the server. If there is some other problem with the request data, the response status should be `400(Bad Request)` or a custom status code in the 4XX series signaling a more specific error.

#### Successful response headers

The response headers after a successful `PUT` request should include the following headers:

| name | description |
|------|-------------|
| Location | The URL where the newly modified document section can be retrieved via a `GET` request |
| Link | The URL for the `Document` endpoint documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation" (e.g., </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation") |
| Content-Type | Content type of the response body (by default `application/tei+xml`)|

#### Successful response body

In a successful `PUT` request, the response body should be an XML object with the same structure as the body of a `GET` response, but containing the the newly modified contents of the specified text section. Although the implementation may require other formats for the `PUT` request body, __the response body *must* by default be formatted in XML as detailed [above](#default-request-and-response-body-format).__ By comparing the original `GET` response body with the response body from the `PUT` request, the client can quickly recognize whether the correct information was updated on the server. Where TEI-compliant XML is accepted as the default for request bodies, this will also mean that the response body is identical to the body submitted in the `PUT` request.

#### Unsuccessful response headers

The response after an unsuccessful `PUT` request should include the following headers:

| name | description |
|------|-------------|
| Link | The URL for the `Document` endpoint documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation" (e.g., </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation") |
| Content-type | Content type of the response body (by default `application/tei+xml`) |

#### Unsuccessful response body

If a response returns an error code, the response body should contain an XML object following the DTS specification for XML status responses (https://w3id.org/dts/api). For example, this would be a well-formed response body for a `400(Bad Request)` error:

```xml
<error statusCode="400" xmlns="https://w3id.org/dts/api">
  <title>Improperly Formed Request Body</title>
  <description>The body of a `PUT` request to the Document endpoint on this server must be properly formed XML.</description>
</error>
```

If a response returns a status of `404(Not Found)` then the `<description>` contents should indicate that no section with the requested `ref` identifier exists and that the section must be created before it can be modified.

If the status code is `400(Bad Request)` then the `<description>` should clarify which parts of the request data were not acceptable.

### PUT Example 1: Changing the child elements in a bottom-level section

In this example we will change the contents of the section with the reference "1:3" in the document with the id "urn:cts:ancJewLit:1Enoch". Since this document is a critical edition, that section contains a series of TEI `<app>` elements, each of which contains a set of parallel `<rdg>` elements with alternate textual variants. In the middle `<app>` element we will add a third `<rdg>` option that is empty, representing a manuscript which lacks any corresponding words.

#### PUT request URL

- `/api/dts/document?id=urn:cts:ancJewLit:1Enoch&ref=1:3&token=XXXXX`

#### PUT request body

If you compare this with the initial XML submitted in the `POST` Example 1 [above](#post-example-1-creating-the-initial-text-for-a-new-document), you will notice that the modification in this case involves changes to the XML child elements, not just to the text they contain. This is permissible since those child elements are below the lowest *structural* level of the document (i.e., the lowest level that can be referenced in the document's standard referencing scheme).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
          <rdg wit="#a"></rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```

##### Successful PUT response status

- `200(OK)`

##### Successful PUT response headers

| key | value |
| --- | ----- |
| Location      | /api/dts/document?id=urn:cts:ancJewLit:1Enoch&ref=1:3 |
| Link          | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation" |
| Content-Type  | application/tei+xml               |

##### Successful PUT response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
          <rdg wit="#a"></rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```

## DELETE on the Document Endpoint

The `DELETE` method of the Document endpoint allows for removal of a segment of text from a document. Note that the `DELETE` operation removes the specified segments entirely from the document's reference structure. So if you `DELETE` the segment designated "12.6.2" from a document, there should thereafter be no line "2" in section "12.6" of the server document. If, instead, you simply want to remove the text of a segment, leaving the segment itself in the document structure, you should use the `PUT` method to replace the text with an empty string.

### DELETE Query Parameters

The query parameters that *must* be accepted for a `DELETE` request are the `id` of the document, `token` (if supported by the implementation), and the same three parameters used in `GET` requests: `ref`, `start`, and `end`.

Multiple structural segments of the document may be deleted simultaneously. The `start` and `end` parameters should be used as described [above](#get-query-parameters) for the Document `GET` method, but with one key difference: __both__ a `start` and an `end` parameter must be provided. This is to prevent the accidental deletion of everything before or after the provided reference. If a `start` parameter is provided without a corresponding `end` (or vice versa), the request should trigger an error.

Note that the depth of the reference provided in the `ref` (or `start`/`end`) parameters determines the depth of the delete operation. If a document has three structural levels, and a `DELETE` request is submitted with a `ref` of "12.6", then *all* of the second-level section "12.6" will be removed from its structure. If a `ref` of "12" is submitted for that same document, all of the top-level section "12" will be removed.

### DELETE Request Body

`DELETE` requests do not include a body.

### DELETE Responses

#### Status codes

With a `DELETE` request the returned status code should vary depending on on whether the operation has already concluded when the response is sent. If the deletion is done synchronously, and is finished at response time, the returned status code should be `200(OK)`. If the request simply began an asynchronous deletion process, still incomplete at response time, then the response should return `202(Accepted)`.

If no item exists with the `id` specified in the request URL, the response status should be `404(Not Found)`. Similarly, if the document contains no structural segment matching the specific `ref` (or `start`/`end`) value, the response should return `404(Not Found)`. If there is some other problem with the request data, the response status should be `400(Bad Request)` or a custom status code in the 4XX series signaling a more specific error.

#### Successful response headers

The response for a successful `DELETE` request should include a "Content-Type" header indicating the content type of the response body. By default this must be `application/tei+xml`

Unlike with other methods, the response headers after a successful `DELETE` request should *not* include a `Location` header.

A "Link" header should also be included specifying the URL for the machine-readable API documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation".

#### Successful response body

In a successful `DELETE` request, the response body should be an XML object with the same structure as a `GET` response (see [above](#default-request-and-response-body-format)), but containing the old contents of the deleted text section. This allows the client to quickly recognize whether the correct segment(s) was removed on the server.

#### Unsuccessful response headers

The response after an unsuccessful `DELETE` request should include a "Content-type" header with the value "application/tei+xml" unless a non-default format has been specified in the request.

A "Link" header should also be included specifying the URL for the machine-readable API documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation".

#### Unsuccessful response body

If a response returns an error code, the response body should contain an XML object following the DTS specification for XML status responses (https://w3id.org/dts/api). For example, this would be a well-formed response body for a `404(Not Found)` error:

```xml
<error statusCode="404" xmlns="https://w3id.org/dts/api">
  <title>Segment Not Found</title>
  <description>The document you requested does exist, but no structural segment exists corresponding to the reference in your request.</description>
</error>
```

If a response returns a status of `404(Not Found)` then the `description` value should indicate whether the problem arose from the requested `id` value or the requested `ref` value. In other words, does the requested document not exist on the server, or does that document not contain the requested structural segment(s)?

If the status code is `400(Bad Request)` then the `<description>` should clarify which parts of the request data were not acceptable.

### DELETE Example 1: Removing a Segment from a Document

In this example we will use a `DELETE` request to remove the section with the reference "1:3" from the document with the `id` "urn:cts:ancJewLit:1Enoch". Since this document is a critical edition, that section contains XML markup for the variant readings as well as the text of each reading. This will be a synchronous `DELETE` operation, so the successful response code will be `200(OK)`.

#### DELETE request URL

- `/api/dts/document?id=urn:cts:ancJewLit:1Enoch&ref=1:3&token=XXXXX`

#### DELETE request body

No body should be sent with the `DELETE` request.

#### Successful DELETE response status

- `200(OK)`

#### Successful DELETE response headers

| key | Value |
| --- | ----- |
| Content-Type  | application/ld+json             |
| Link          | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation" |

#### successful DELETE response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
          <rdg wit="#a"></rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```
