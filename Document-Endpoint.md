# Distributed Text Services API : Document Endpoint

The documents endpoint is used to access the data for documents, as opposed to metadata (which is found in collections).  The representation of a document is up to the implementation.

## Default Scheme

- Implementations of the DTS document endpoint **can** support as many response formats as the content provider wishes.
- Implementations of the DTS document endpoint  **must**, at minimum, support an `application/tei+xml` response.
- The scheme for the `application/tei+xml` needs to be containing the `<TEI>` rootnode of the namespace `http://www.tei-c.org/ns/1.0`.
- If the document or passage returned is a reconstruction, the reconstruction of the required fragment should be embedded in the `<fragment>` node of the DTS Namespace (`https://w3id.org/dts/api#`) such as

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <!-- XML or string of the passage requested here -->
  </dts:fragment>
</TEI>
```
- If the request returns a complete document, it is returned directly, without a `dts:fragment` element.
- There is no limitation to what can be contained by `dts:fragment` or what its siblings can be. The only limiting factor is that `dts:fragment` should contain the requested fragment.

## URI 

### Query Parameters

The documents endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| passage | passage identifier (used together with `id` can't be used with `start` and `end`) | n/a    |
| start | (For range) Start of a range of passages (can't be used with `passage`) | GET |
| end |  (For range) End of a range of passages (requires `start` and no `passage`) | GET |

### Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |
| Content-Type | Content type of the response (By default : `application/tei+xml`)|

### URI Template

Here is a template of the URI for Document API. The route itself (`/dts/api/document/`) is up to the implementer.

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
  },
  "@type": "IriTemplate",
  "template": "/dts/api/document/?id={collection_id}&passage={passage}&level={level}&start={start}&end={end}&page={page}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "collection_id",
      "required": true
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "passage",
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

### Retrieve a passage using `passage`

Retrieve the passage `1` of `bgu.11.2029`

#### Example of url : 

- GET `/dts/api/documents/?id=bgu.11.2029&passage=2`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/documents/?id=bgu.11.2029&passage=1>; rel="prev", </dts/api/documents/?id=bgu.11.2029&passage=3>; rel="next" | 

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
| Link | </dts/api/documents/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.2.1&end=1.2.2>; rel="next" | 

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

Retrieve the full document bgu.11.2029

#### Example of url : 

- GET `/dts/api/documents/?id=bgu.11.2029`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |

#### Response

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-model href="http://www.stoa.org/epidoc/schema/8.13/tei-epidoc.rng" type="application/xml" schematypens="http://relaxng.org/ns/structure/1.0"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0" xml:id="hgv9566">
   <teiHeader>
      <fileDesc>
         <titleStmt>
            <title>Torzollquittung</title>
         </titleStmt>
         <publicationStmt>
            <idno type="filename">9566</idno>
            <idno type="TM">9566</idno>
            <idno type="ddb-perseus-style">0001;11;2029</idno>
            <idno type="ddb-filename">bgu.11.2029</idno>
            <idno type="ddb-hybrid">bgu;11;2029</idno>
         </publicationStmt>
      </fileDesc>
      <encodingDesc>
         <p>This file encoded to comply with EpiDoc Guidelines and Schema version 8
                <ref>http://www.stoa.org/epidoc/gl/5/</ref>
         </p>
      </encodingDesc>
   </teiHeader>
   <text>
      <body>
         <div type="commentary" subtype="general">
            <p>Tag: 9. Aug. Zur Datierung vgl. P.Customs, S. 114 und 154 zu Nr. 238 und zum Tagesdatum ZPE 106, 1995, S. 194.</p>
         </div>
         <div type="bibliography" subtype="principalEdition">
            <listBibl>
               <bibl type="publication" subtype="principal">
                  <title level="s" type="abbreviated">BGU</title>
                  <biblScope type="volume">11</biblScope>
                  <biblScope type="numbers">2029</biblScope>
               </bibl>
            </listBibl>
         </div>
         <div type="bibliography" subtype="corrections">
            <head>BL-Einträge nach BL-Konkordanz</head>
            <listBibl>
               <bibl type="BL">
                  <biblScope type="volume">VIII</biblScope> 
                  <biblScope type="pages">51</biblScope>
               </bibl>
               <bibl type="BL">
                  <biblScope type="volume">X</biblScope> 
                  <biblScope type="pages">22</biblScope>
               </bibl>
            </listBibl>
         </div>
         <div type="bibliography" subtype="illustrations">
            <p>keine</p>
         </div>
      </body>
   </text>
</TEI>
```
