# Distributed Text Services API - Navigation Endpoint

The Navigation endpoint provides a list of passages that are available for a given resource. Its direction is parent-to-child by default.

## Scheme

Here is the scheme for the current draft. Everything that is not marked as Optional is mandatory.

JSON wide attributes :

Item properties :
- `@base` is the URI of the Document API at which we can retrieve passages
- `@id` is the ID of the current request
- `dts:citeDepth` defines the maximum depth of the document, *e.g.* if the a document has up to three levels, `dts:citeDepth` should be three
- `dts:level` defines the level of the reference given. 
- `dts:passage` contains a URI template to the Document endpoint
- `member` is a list of passages
  - A list of passages can be made of single `ids` : `[{"ref": "a"}, {"ref": "b"}, {"ref": "1.1"}]`
  - A list of passages can be made of ranges : `[{"start": "a", "end": "b"}]`
  - (Optional) `dts:citeType` contains information about passage type for each member. *e.g.* `{"ref": "1.2", "dts:citeType": "Poem"}`
  - (Optional) `dts:dublincore` contains Dublin Core Terms metadata for each passage : `{"ref": "1.2", "dts:dublincore": {"dc:author": "Balzac"}}`
  - (Optional) `dts:extensions` contains metadata from other namespaces


## URI 

### Query Parameters

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| ref | passage identifier (used together with `id`) | GET    |
| level | Depth for passages we want to retrieve identifiers of  | GET    |
| start | (For range) Start of the range passages (inclusive, not to be used with `passage`) | GET |
| end |  (For range) End of the range of passages (inclusive, requires `start`, not to be used with `passage`) | GET |
| groupSize | Retrieve passages in groups of this size instead of single units | GET |
| max | Allows for limiting the number of results and getting pagination | GET | 
| exclude | Exclude keys in members' object such as `exclude=dts:extensions` | GET |

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
  },  
  "@type": "IriTemplate",
  "template": "/dts/api/navigation/?id={collection_id}{&ref}{&level}{&start}{&end}{&page}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "collection_id",
      "property": "hydra:freetextQuery",
      "required": true
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "passage",
      "property": "hydra:freetextQuery",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "page",
      "property": "hydra:freetextQuery",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "level",
      "property": "hydra:freetextQuery",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "start",
      "property": "hydra:freetextQuery",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "end",
      "property": "hydra:freetextQuery",
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
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc",
    "dts:citeDepth" : 2,
    "dts:level": 1,
    "member": [
      {"ref": "1"},
      {"ref": "2"},
      {"ref": "3"}
    ],
    "dts:passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}"
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
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=2",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"ref": "1.1"},
      {"ref": "1.2"},
      {"ref": "2.1"},
      {"ref": "2.2"},
      {"ref": "3.1"},
      {"ref": "3.2"}
    ],
    "dts:passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}"
}
```

### Children of a passage

The client wants to retrieve a list of passage identifiers that are part of the document *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and its passage `1`.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&ref=1`

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
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&ref=1",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"ref": "1.1"},
      {"ref": "1.2"}
    ],
    "dts:passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}"
}
```

### Descendants of a passage

The client wants to retrieve a list of grand-children passage identifiers that are part of the document *urn:cts:latinLit:phi1294.phi001.perseus-lat2* and its passage `1`.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1&level=2`

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
    },
    "@id":"/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1",
    "dts:citeDepth" : 3,
    "dts:level": 3,
    "member": [
      {"ref": "1.1.1"},
      {"ref": "1.1.2"},
      {"ref": "1.2.1"},
      {"ref": "1.2.2"}
    ],
    "dts:passage": "/dts/api/document/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2{&ref}{&start}{&end}"
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
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=0&start=1&end=3",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"ref": "1"},
      {"ref": "2"},
      {"ref": "3"}
    ],
    "dts:passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}"
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
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=1&start=1&end=3",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"ref": "1.1"},
      {"ref": "1.2"},
      {"ref": "2.1"},
      {"ref": "2.2"},
      {"ref": "3.1"},
      {"ref": "3.2"},
    ],
    "dts:passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}"
}
```

### Passages grouped by the provider 

The client wants to retrieve a list of grand-children ranges of two identifiers that are part of the document *urn:cts:latinLit:phi1294.phi001.perseus-lat2* and its passage `1`.

#### Example of url : 

- `/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1&level=2&groupSize=2`

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
    "@id":"/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1&level=2&groupSize=2",
    "dts:citeDepth" : 3,
    "dts:level": 3,
    "member": [
      {"start": "1.1.1", "end": "1.1.2"},
      {"start": "1.2.1", "end": "1.2.2"},
    ],
    "dts:passage": "/dts/api/document/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2{&ref}{&start}{&end}"
}
```

### Retrieval of typology of references 

Some passages may have a metadata type. The `citeType` refers to the type of citable node has been evaluated as. The node expects a free text or a RDF Class. A default type can be given at the root of the response object.

#### Example of url : 

- `/api/dts/navigation/?id=http://data.bnf.fr/ark:/12148/cb11936111v`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

Example using *Les Liaisons Dangereuses* by Pierre Choderlos de Laclos

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "foo": "http://foo.bar/ontology"
    },
    "@id":"/api/dts/navigation/?id=http://data.bnf.fr/ark:/12148/cb11936111v",
    "dts:citeDepth" : 1,
    "dts:level": 1,
    "dts:citeType": "letter",
    "member": [
      // The two following items are not letters : the data provider notes this different
      { "ref": "Av", "dts:citeType": "preface"},
      { "ref": "Pr", "dts:citeType": "preface"},
      // Given the fact the following nodes have no citeType, they inherit of the root object citeType : letter
      { "ref": "1" },
      { "ref": "2" },
      { "ref": "3" },
      // And so on
    ],
    "dts:passage": "/dts/api/document/?id=http://data.bnf.fr/ark:/12148/cb11936111v{&ref}{&start}{&end}"
}
```


### Retrieval of titles and generic metadata

The client wants the list of passages with their title. If the given data provider has a title, then it will be provided.

#### Example of url : 

- `/api/dts/navigation/?id=http://data.bnf.fr/ark:/12148/cb11936111v`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

Example using *Les Liaisons Dangereuses* by Pierre Choderlos de Laclos

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "foo": "http://foo.bar/ontology"
    },
    "@id":"/api/dts/navigation/?id=http://data.bnf.fr/ark:/12148/cb11936111v",
    "dts:citeDepth" : 1,
    "dts:level": 1,
    "member": [
      {
        "ref": "Av", 
        "dts:dublincore": {
        "dc:title": "Avertissement de l'Éditeur"
        }
      },
      {
        "ref": "Pr", 
        "dts:dublincore": {
          "dc:title": "Préface"
        }
      },
      {
        "ref": "1", 
        "dts:dublincore": {
          "dc:title": "Lettre 1"
        },
        "dts:extensions": {
          "foo:fictionalSender": "Cécile Volanges",
          "foo:fictionalRecipient": "Sophie Carnay"
        }
      },
      {
        "ref": "2", 
        "dts:dublincore": {
          "dc:title": "Lettre 2"
        },
        "dts:extensions": {
          "foo:fictionalSender": "La Marquise de Merteuil",
          "foo:fictionalRecipient": "Vicomte de Valmont"
        }
      },
      {
        "ref": "3", 
        "dts:dublincore": {
          "dc:title": "Lettre 3"
        },
        "dts:extensions": {
          "foo:fictionalSender": "Cécile Volanges",
          "foo:fictionalRecipient": "Sophie Carnay"
        }
      },
      // And so on
    ],
    "dts:passage": "/dts/api/document/?id=http://data.bnf.fr/ark:/12148/cb11936111v{&ref}{&start}{&end}"
}
```
