# Document Endpoint

The Document endpoint is used to access the content of document, as opposed to metadata (which is found in collections).

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


## URI

### Query Parameters

The Document endpoint supports the following query parameters:

| Name | Description                              |
|------|------------------------------------------|
| id	| (Required) Identifier for a document. Where possible this should be a URI	|
| ref   | Passage identifier (used together with `id`; can’t be used with `start` and `end`)	|
| start	| (For range) Start of a range of passages (can’t be used with `ref`)	|
| end	| (For range) End of a range of passages (requires `start` and no `ref`)	|
| format | (Optional) Specifies a data format for response/request body other than the default	|

Note that one may either provide a `ref` parameter __or__ a pair of `start` and `end` parameters. A request cannot combine `ref` with the other two. If, say, a `ref` and a `start` are both provided this should cause the request to fail.

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
      "variable": "format",
      "required": false
    }
  ]
}
```

## Batch Requests

For the sake of simplicity and usability the DTS guidelines do not allow for batch operations, i.e. fetching or altering more than one passage of text in the same request. Implementations should encourage client software and other consumers of their API to use multiple asynchronous requests instead.

## Requests on the Document Endpoint

### Query parameters

The only strictly required parameter for Document request is `id`. This value specifies the document to be queried. If no particular reference is specified with other parameters, the response should return the entire document. If only one structural segment of the document is desired, this should be specified using the `ref` parameter. The `ref` value may specify a segment at any structural level of the document. For example, in a document segmented in two structural levels (book and line), a `ref` value of "5" should return all of book 5. A `ref` value of "5.2" should return just line 2 from book 5.

If a continuous sequence of structural segments is being requested, then no `ref` value should be specified. Instead, the `start` and `end` parameters should specify the first and last segments of the sequence to be returned. If only `start` is specified, with no `end` value, the request should be understood as having an implicit `end` parameter specifying the last structural segment of the document. In other words, a request with a `start` value of "5.2" and no explicit `end` value should return all of the document from 5.2 (inclusive) to the end. Likewise, if only an `end` parameter is specified, the server should understand an implicit `start` pointing to the first segment of the document.

As with `ref`, the level of specificity in the `start` and `ref` values should determine the structural level used to demarcate the segments returned. A query with "start=5.2&end=6.1" should return every line from 5.2 to 6.1 (inclusive). A query with "start=5&end=6" should return every line from the beginning of book 5 to the end of book 6 (inclusive).

No request should include both a `ref` parameter and either `start` or `end`. Any request that does include both should prompt a `400(Bad Request)` error.

### Request body

A `GET` request should not include a body. If one is included it should be ignored.

### Responses

#### Status codes

A successful request to the Document endpoint should return the status code `200(OK)`.

If a request is unsuccessful because of problems with the request parameters, the return status code should be `400(Bad Request)` or a custom status code in the 4XX series signaling a more specific error.

If a request is unsuccessful because the specified id matches no document on the server, the return status code should be `404(Not Found)`

#### Successful response headers

The response after a successful request contains the following response headers:

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

The response body after a successful request should contain an XML document representing the requested text segment. This XML should be formatted as described above under "[Default Request and Response Body Format](#default-request-and-response-body-format)".

#### Unsuccessful response headers

An unsuccessful request should return the following headers:

| name | description |
|------|-------------|
| Link | The URL for the Hydra-compliant machine-readable `Document` endpoint documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation |
| Content-type | Content type of the response body (by default `application/tei+xml`) |

#### Unsuccessful response body

If a response returns an error code, the response body should contain an XML object following the DTS specification for XML status responses (https://w3id.org/dts/api). For example, this would be a well-formed response body for a `400(Bad Request)` error:

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

### Example 1: Retrieve a passage using `ref`

Retrieve the passage `2` of the document labeled by the identifier `https://papyri.info/ddbdp/bgu;11;2029/source`.

#### request url

- `/dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source&ref=2`

#### Successful response status

- 200(OK)

#### Successful response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation", </dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source&ref=1>; rel="prev", </dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source&ref=3>; rel="next", </dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source&ref=6>; rel="last", </dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source>;rel="up", </dts/api/navigation/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="contents", </dts/api/collection/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="collection" |

#### Successful response body

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

### Example 2: Retrieve a passage using `start` and `end`

Retrieve the passages 1.1.1 to the passage 1.1.2 of the document labeled by the identifier `urn:cts:latinLit:phi1318.phi001.perseus-lat1`.

#### request url

- `/dts/api/document/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.1.1&end=1.1.2`

#### Successful response status

- 200(OK)

#### Successful response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation", </dts/api/document/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.2.1&end=1.2.2>; rel="next", </dts/api/document/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=5.5.5&end=5.5.6>; rel="last", </dts/api/navigation/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&ref=1.2>; rel="up", </dts/api/navigation/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1>; rel="contents", </dts/api/collection/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1>; rel="collection" |

#### Successful response body

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

### Example 3: Retrieve a full document

Retrieve the full document labeled by the identifier `https://papyri.info/ddbdp/bgu;11;2029/source`

#### request url

- `/dts/api/document/?id=https://papyri.info/ddbdp/bgu;11;2029/source`

#### Successful response status

- 200(OK)

#### Successful response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/document/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation", </dts/api/navigation/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="contents", </dts/api/collection/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="collection" |

#### Successful response body

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
