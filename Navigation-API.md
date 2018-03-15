# Distributed Text Services - Navigation API

Navigation API's role is to provide a list of passages that are available for a given resource. It's direction is parent-to-children by default.

## Scheme

Here is the scheme for the current draft. Everything that is not marked as Optional is mandatory.

JSON wide attributes :

Item properties :
- `@base` is the URI of the Document API at which we can retrieve passages
- `@id` is the ID if the current document
- `passage` is a list of passage

## URI 

### Query Parameters

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| passage | passage identifier (used together with `id`) | GET    |
| level | Depth for passages we want to retrieve identifiers of  | GET    |
| start | (For range) Start of the passage we want descendants of | GET |
| end |  (For range) End of the passage we want descendants of | GET |
| types | ([**Not definitive. See issue 78**](https://github.com/distributed-text-services/collection-api/issues/78)) Collect categories of passage as you retrieve their identifier | GET |

### Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |

### URI Template

Here is a template of the URI for Collection API. The route itself (`/dts/api/navigation/`) is up to the implementer.

```json
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@type": "IriTemplate",
  "template": "/dts/api/navigation/?id={collection_id}&passage={passage}&level={level}&start={start}&end={end}&page={page}",
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
        "passage": "http://purl.org//dts-ontology/#passage"
    },
    "@base": "/dts/api/document/",
    "@id":"urn:cts:greekLit:tlg0012.tlg001.opp-grc5",
    "passage": ["1", "2", "3"]
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
        "passage": "http://purl.org//dts-ontology/#passage"
    },
    "@base": "/dts/api/document/",
    "@id":"urn:cts:greekLit:tlg0012.tlg001.opp-grc5",
    "passage": ["1.1",  "1.2", "2.1", "2.2", "3.1", "3.2"]
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
        "passage": "http://purl.org//dts-ontology/#passage"
    },
    "@base": "/dts/api/document/",
    "@id":"urn:cts:greekLit:tlg0012.tlg001.opp-grc5",
    "passage": ["1.1", "1.2"]
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
        "passage": "http://purl.org//dts-ontology/#passage"
    },
    "@base": "/dts/api/document/",
    "@id":"urn:cts:greekLit:tlg0012.tlg001.opp-grc5",
    "passage": ["1.1.1", "1.1.2", "1.2.1", "1.2.2"]
}
```

### References between two milestones

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
        "passage": "http://purl.org//dts-ontology/#passage"
    },
    "@base": "/dts/api/document/",
    "@id":"urn:cts:greekLit:tlg0012.tlg001.opp-grc5",
    "passage": ["2"]
}
```

### Descendant references between two milestones

The client wants to retrieve a list of passage identifiers which are between two milestones.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&level=1&start=1&end=3`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "passage": "http://purl.org//dts-ontology/#passage"
    },
    "@base": "/dts/api/document/",
    "@id":"urn:cts:greekLit:tlg0012.tlg001.opp-grc5",
    "passage": ["2.1", "2.2"]
}
```

### Retrieval of typology of references

**Waiting for [Issue #78](https://github.com/distributed-text-services/collection-api/issues/78)**

### Retrieval of titles

**Waiting for [Issue #78](https://github.com/distributed-text-services/collection-api/issues/78)**