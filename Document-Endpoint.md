# Distributed Text Services API : Document Endpoint

The documents endpoint is used to access the data for documents, as opposed to metadata (which is found in collections).  The representation of a document is up to the implementation.

## Default Scheme

- Implementations of the DTS document endpoint **can** support as many response formats as the content provider wishes.
- Implementations of the DTS document endpoint  **must**, at minimum, support an `application/tei+xml` response.
- The scheme for the `application/tei+xml` needs to be containing the `<TEI>` rootnode of the namespace `http://www.tei-c.org/ns/1.0`.
- If the document or passage returned is a reconstruction, the reconstruction of the required fragment should be embedded in the `<fragment>` element of the DTS Namespace (`https://w3id.org/dts/api#`) such as

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <!-- XML or string of the passage requested here -->
  </dts:fragment>
</TEI>
```
- If the request returns a complete document, it is returned directly, without a `dts:fragment` element.
- There is no limitation to what can be contained by `dts:fragment` or what its siblings can be. The only limiting factor is that `dts:fragment` should contain the requested fragment. This permits an implementation to return contextual material elsewhere in the document alongside the requested fragment.  

## URI 

### Query Parameters

The documents endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| ref | passage identifier (used together with `id` can't be used with `start` and `end`) | n/a    |
| start | (For range) Start of a range of passages (can't be used with `ref`) | GET |
| end |  (For range) End of a range of passages (requires `start` and no `ref`) | GET |

### Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |
| Content-Type | Content type of the response (By default : `application/tei+xml`)|

#### Possible values and their signification for Link

When applicable, the following links must be provided in the Link property of the header :

| Name of the property | Description of its value |
| -------------------- | ------------------------ |
| prev | Previous passage of the document in the Document endpoint |
| next | Next passage of the document in the Document endpoint |
| up | Parent passage of the document in the Document endpoint |
| first | First passage of the document in the Document endpoint  |
| last | Last passage of the document in the Document endpoint |
| contents | Link to the Navigation Endpoint for the current document |
| collection | Link to the Collection endpoint for the current document |

### URI Template

Here is a template of the URI for Document API. The route itself (`/dts/api/document/`) is up to the implementer.

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  },
  "@type": "IriTemplate",
  "template": "/dts/api/document/?id={collection_id}&ref={ref}&start={start}&end={end}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "collection_id",
      "required": true
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "ref",
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
    }
  ]
}
```

## Examples

### Retrieve a passage using `ref`

Retrieve the passage `2` of `bgu;11;2029`

#### Example of url : 

- GET `/dts/api/documents/?id=bgu;11;2029&ref=2`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/documents/?id=bgu;11;2029&ref=1>; rel="prev", </dts/api/documents/?id=bgu;11;2029&ref=3>; rel="next", </dts/api/documents/?id=bgu;11;2029&ref=6>; rel="last", </dts/api/navigation/?id=bgu;11;2029>; rel="contents", </dts/api/collection/?id=bgu;11;2029>; rel="collection" | 

#### Response

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

### Retrieve a passage using start and end

Retrieve the passages 1.1.1 to the passage 1.1.2

#### Example of url : 

- GET `/dts/api/documents/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.1.1&end=1.1.2`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/documents/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.2.1&end=1.2.2>; rel="next", </dts/api/documents/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=5.5.5&end=5.5.6>; rel="last", </dts/api/navigation/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1>; rel="contents", </dts/api/collection/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1>; rel="collection" | 

#### Response

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


### Retrieve a full document

Retrieve the full document bgu;11;2029

#### Example of url : 

- GET `/dts/api/documents/?id=bgu;11;2029`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/navigation/?id=bgu;11;2029>; rel="contents", </dts/api/collection/?id=bgu;11;2029>; rel="collection" | 

#### Response

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
