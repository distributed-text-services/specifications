# Distributed Text Services API - Navigation Endpoint

The Navigation endpoint provides a list of passages that are available for a given resource. Its direction is parent-to-child by default.

## Scheme

Here is the scheme for the current draft. Everything that is not marked as Optional is mandatory.

JSON wide attributes :

Item properties :
- `@base` is the URI of the Document API at which we can retrieve passages
- `@id` is the ID if the current document
- `passage` is a list of passages
  - A list of passages can be made of single `ids` : `["a", "b", "1.1"]`
  - A list of passages can be made of ranges : `[{"start": "a", "end": "b"}]`

## URI 

### Query Parameters

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| passage | passage identifier (used together with `id`) | GET    |
| level | Depth for passages we want to retrieve identifiers of  | GET    |
| start | (For range) Start of the range passages (inclusive, not to be used with `passage`) | GET |
| end |  (For range) End of the range of passages (inclusive, requires `start`, not to be used with `passage`) | GET |
| chunkSize | Retrieve passages in groups of this size instead of single units | GET |
| max | Allows for limiting the number of results and getting pagination | GET | 


### Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |

### URI Template

Here is a template of the URI for Collection API. The route itself (`/dts/api/navigation/`) is up to the implementer.

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
  },  
  "@type": "IriTemplate",
  "template": "/dts/api/navigation/?id={collection_id}{&passage}{&level}{&start}{&end}{&page}",
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
      "variable": "page",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "level",
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

### Children of a text identifier

The client wants to retrieve a list of passage identifiers that are part of the document *urn:cts:greekLit:tlg0012.tlg001.opp-grc5*.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc",
    "member": [
      {"ref": "1"},
      {"ref": "2"},
      {"ref": "3"}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&passage}{&level}{&start}{&end}"
}
```

### Descendants of a text identifier

The client wants to retrieve a list of passage identifiers that are part of the document *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and can be found at the second level of the citation tree of the document.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&level=2`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=2",
    "level": 2,
    "member": [
      {"ref": "1.1"},
      {"ref": "1.2"},
      {"ref": "2.1"},
      {"ref": "2.2"},
      {"ref": "3.1"},
      {"ref": "3.2"}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&passage}{&level}{&start}{&end}"
}
```

### Children of a passage

The client wants to retrieve a list of passage identifiers that are part of the document *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and its passage `1`.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&passage=1`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&passage=1",
    "member": [
      {"ref": "1.1"},
      {"ref": "1.2"}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&passage}{&level}{&start}{&end}"
}
```

### Descendants of a passage

The client wants to retrieve a list of grand-children passage identifiers that are part of the document *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and its passage `1`.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&passage=1&level=2`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&passage=1",
    "member": [
      {"ref": "1.1.1"},
      {"ref": "1.1.2"},
      {"ref": "1.2.1"},
      {"ref": "1.2.2"}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&passage}{&level}{&start}{&end}"
}
```

### Ranges of references

The client wants to retrieve a list of passage identifiers which are between two milestones.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&level=0&start=1&end=3`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=0&start=1&end=3",
    "member": [
      {"ref": "1"},
      {"ref": "2"},
      {"ref": "3"}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&passage}{&level}{&start}{&end}"
}
```

### Descendant reference ranges

The client wants to retrieve a list of passage identifiers which are between two milestones.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&level=2&start=1&end=3`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=1&start=1&end=3",
    "member": [
      {"ref": "1.1"},
      {"ref": "1.2"},
      {"ref": "2.1"},
      {"ref": "2.2"},
      {"ref": "3.1"},
      {"ref": "3.2"},
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&passage}{&level}{&start}{&end}"
}
```

### Passages grouped by the provider 

The client wants to retrieve a list of grand-children ranges of two identifiers that are part of the document *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and its passage `1`.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&passage=1&level=2&chunkSize=2`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&passage=1&level=2&chunkSize=2",
    "member": [
      {"start": "1.1.1", "end": "1.1.2"},
      {"start": "1.2.1", "end": "1.2.2"},
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&passage}{&level}{&start}{&end}"
}
```

### Retrieval of typology of references (**Future Draft Only**)

**Waiting for [Issue #78](https://github.com/distributed-text-services/collection-api/issues/78)**

### Retrieval of titles (**Future Draft Only**)

**Waiting for [Issue #80](https://github.com/distributed-text-services/collection-api/issues/80)**
