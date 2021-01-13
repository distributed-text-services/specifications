# Navigation Endpoint

The Navigation endpoint provides a list of passages that are accessible for navigation from a given reference within a resource. Responses from the Navigation endpoint assume by default that the user is traversing the document downward in the citation tree. A request to the Navigation endpoint specifies a reference within a resource, and the response lists the references that are immediately above or below that reference point in the citation tree. The response also identifies the parent reference of that current node.

## Scheme

Here is the scheme for the current draft. Everything that is not marked as Optional is mandatory.

JSON wide attributes :

Item properties :
- `dts:passage` is the URI of the Documents API at which we can retrieve passages
- `@id` is the ID of the current request
- `dts:citeDepth` defines the maximum depth of the document, *e.g.* if the a document has up to three levels, `dts:citeDepth` should be three
- `dts:citeType` defines the default type of references listed in `member`
- `dts:level` defines the level of the reference given.
- `dts:passage` contains a URI template to the Documents endpoint
- `member` is a list of passages
  - A list of passages can be made of single `ids` : `[{"dts:ref": "a"}, {"dts:ref": "b"}, {"dts:ref": "1.1"}]`
  - A list of passages can be made of ranges : `[{"dts:start": "a", "dts:end": "b"}]`
  - (Optional) `dts:citeType` contains information about passage type for each member. *e.g.* `{"dts:ref": "1.2", "dts:citeType": "Poem"}`
  - (Optional) `dts:dublincore` contains Dublin Core Terms metadata for each passage : `{"dts:ref": "1.2", "dts:dublincore": {"dc:author": "Balzac"}}`
  - (Optional) `dts:extensions` contains metadata from other namespaces
- `dts:parent` is the unique ID for the hierarchical parent of the current node in the document structure, defined by the `ref` query parameter.

### Unique `ref` identifiers

Note that all identifiers used as a `dts:ref` value must be unique within the current resource. This is the case for the values retured in the `member` list as well as in the `dts:parent` property.

### Values for the `dts:parent` Property

The format for the returned `dts:parent` value will depend on where the current `ref` stands in the resource's hierarchical structure.

- If the requested `ref` is the identifier for the **resource as a whole**, and that resource has no hierarchical parent, the value returned for `dts:parent` should be the array `[null]`.
- If the requested `ref` identifies **one of the top level** of the resource's hierarchical divisions, the `dts:parent` property should be an object identifying the document as a whole and specifying that its `@type` is a "Resource". For example: `{"@type": "Resource", "@id": "urn:cts:greekLit:tlg0012.tlg001.opp-grc5"}`
- If the requested `ref` identifies **a node at a lower level** of the resource's hierarchical divisions, so that the parent is another division within the citation structure, the `dts:parent` value will be a list of objects much like the list returned for the `member` property, each object identifying one reference that is the current node's direct parent. In this case, though, each object should also include an `@type` value of "Reference". For example: `{"@type": "Reference", "dts:ref": "1.1.1"}`. If only one parent exists then a single object may be returned rather than an array of objects.


## URI

### Query Parameters

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| ref | passage identifier (used together with `id`) | GET    |
| level | Depth for passages we want to retrieve identifiers of  | GET    |
| start | (For range) Start of the range passages (inclusive, not to be used with `ref`) | GET |
| end |  (For range) End of the range of passages (inclusive, requires `start`, not to be used with `ref`) | GET |
| groupBy | Retrieve passages in groups of this size instead of single units | GET |
| max | Allows for limiting the number of results and getting pagination | GET |
| exclude | Exclude keys in members' object such as `exclude=dts:extensions` | GET |

### Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |

### URI Template

Here is a template of the URI for Navigation API. The route itself (`/dts/api/navigation/`) is up to the implementer.

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  },
  "@type": "IriTemplate",
  "template": "/dts/api/navigation/?id={collection_id}{&ref}{&level}{&start}{&end}{&page}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "id",
      "property": "hydra:freetextQuery",
      "required": true
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "dts:ref",
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
      "variable": "groupBy",
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
      "variable": "dts:start",
      "property": "hydra:freetextQuery",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "dts:end",
      "property": "hydra:freetextQuery",
      "required": false
    }
  ]
}
```

## Examples

### Example 1: Requesting top-level children of a textual Resource

The client wants to retrieve a list of passage identifiers that are part of the textual Resource identified as  *urn:cts:greekLit:tlg0012.tlg001.opp-grc5*. In other words, the client wants a list of the top-level references in the resource's citation structure. So the `id` parameter supplied in the query is the identifier of the Resource as a whole.

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
        "dts": "https://w3id.org/dts/api#"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc",
    "dts:citeDepth" : 2,
    "dts:level": 1,
    "member": [
      {"dts:ref": "1"},
      {"dts:ref": "2"},
      {"dts:ref": "3"}
    ],
    "dts:passage": "/dts/api/documents/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "dts:parent": [null]
}
```

### Example 2: Requesting all descendants of a textual Resource at a specified level

The client wants to retrieve a list of passage identifiers that are part of the textual Resource identified by *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and can be found at the second level of the citation tree of the document. Although the second structural level is being returned, we are not requesting the descendants of any single upper-level node. The references returned are in relation to the document as a whole. So the parent node is the whole document.

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
        "dts": "https://w3id.org/dts/api#"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=2",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"dts:ref": "1.1"},
      {"dts:ref": "1.2"},
      {"dts:ref": "2.1"},
      {"dts:ref": "2.2"},
      {"dts:ref": "3.1"},
      {"dts:ref": "3.2"}
    ],
    "dts:passage": "/dts/api/documents/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "dts:parent": {"@type": "Resource", "@dts:ref": "/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc"}
}
```

### Example 3: Requesting children of one top-level structural division of a Resource

The client wants to retrieve a list of passage identifiers that are children of the textual Resource identified by *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and its division `1`. Since the specified reference, the point from which the descendants are viewed, is at the top level of the structural hierarchy, the parent in the return value is still the document as a whole.

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
        "dts": "https://w3id.org/dts/api#"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&ref=1.1",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"dts:ref": "1.1"},
      {"dts:ref": "1.2"}
    ],
    "dts:passage": "/dts/api/documents/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "dts:parent": {"@type": "Resource", "@dts:ref": "/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc"}
}
```

### Example 4: Requesting grandchild descendants of a top-level structural division

The client wants to retrieve a list of grand-children passage identifiers that are part of the textual Resource identified by *urn:cts:latinLit:phi1294.phi001.perseus-lat2* and its passage `1`. Since the specified reference, the point from which the descendants are viewed, is at the top level of the structural hierarchy, the parent in the return value is still the document as a whole.

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
        "dts": "https://w3id.org/dts/api#"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1",
    "dts:citeDepth" : 3,
    "dts:level": 3,
    "member": [
      {"dts:ref": "1.1.1"},
      {"dts:ref": "1.1.2"},
      {"dts:ref": "1.2.1"},
      {"dts:ref": "1.2.2"}
    ],
    "dts:passage": "/dts/api/documents/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2{&ref}{&start}{&end}",
    "dts:parent": {"@type": "Resource", "dts:ref": "/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2"}
}
```

### Example 5: Requesting children of a lower-level structural division

The client wants to retrieve a list of child passage identifiers that are part of the textual Resource identified by *urn:cts:latinLit:phi1294.phi001.perseus-lat2* and its passage "1.1". The returned parent is the direct parent (or parents) of the specified reference ("1.1"). Since it is not the document as a whole, the parent `@type` is "Reference" rather than "Resource".

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1.1&level=1`

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
        "dts": "https://w3id.org/dts/api#"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1.1",
    "dts:citeDepth" : 3,
    "dts:level": 3,
    "member": [
      {"dts:ref": "1.1.1"},
      {"dts:ref": "1.1.2"}
    ],
    "dts:passage": "/dts/api/documents/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2{&ref}{&start}{&end}",
    "dts:parent": {"@type": "Reference", "dts:ref": "1"}
}
```

### Example 6: Requesting a ranges of passage references between milestones

The client wants to retrieve a list of passage identifiers which are between two milestones. In this case there is no single parent node shared by the whole requested range, so no parent is returned. Since the reference list to be returned is at the *same* structural level as the supplied milestones, the `level` query parameter is "0".

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
        "dts": "https://w3id.org/dts/api#"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=0&start=1&end=3",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"dts:ref": "1"},
      {"dts:ref": "2"},
      {"dts:ref": "3"}
    ],
    "dts:passage": "/dts/api/documents/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "dts:parent": [null]
}
```

### Example 7: Requesting descendants of the references in a specified range

The client wants to retrieve a list of passage identifiers which are between two milestones. Since the `level` query parameter is "1", the returned references are one level below the milestones specified. I.e., the returned references are all *children* of structural divisions between the two milestones. In this case there is no single parent node shared by the whole requested range, so no parent is returned.

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
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&level=1&start=1&end=3",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"dts:ref": "1.1"},
      {"dts:ref": "1.2"},
      {"dts:ref": "2.1"},
      {"dts:ref": "2.2"},
      {"dts:ref": "3.1"},
      {"dts:ref": "3.2"},
    ],
    "dts:passage": "/dts/api/documents/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "dts:parent": [null]
}
```

### Example 8: Passages grouped by the provider

The client wants to retrieve a list of grand-children ranges of two identifiers that are part of the textual Resource identified by *urn:cts:latinLit:phi1294.phi001.perseus-lat2* and its passage `1`. In this case the references are returned as ranges, each of which groups a number of sequential references equal to the `groupBy` request parameter. Since the specified ref is at the top structural level of the document, the returned parent is the entire Resource.

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1&level=2&groupBy=2`

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
      {"dts:start": "1.1.1", "dts:end": "1.1.2"},
      {"dts:start": "1.2.1", "dts:end": "1.2.2"},
    ],
    "dts:passage": "/dts/api/documents/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2{&ref}{&start}{&end}"
    "dts:parent": {"@type": "Resource", "dts:ref": "/dts/api/documents/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2{&ref}{&start}{&end}"}
}
```

### Example 9: Retrieval of typology of references

Some passages may have a metadata type. The `citeType` refers to the type of a citable node. The node expects a free text or a RDF Class. A default type can be given at the root of the response object.

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
        "foo": "http://foo.bar/ontology#"
    },
    "@id":"/api/dts/navigation/?id=http://data.bnf.fr/ark:/12148/cb11936111v",
    "dts:citeDepth" : 1,
    "dts:level": 1,
    "dts:citeType": "letter",
    "member": [
      // The two following items are not letters : the data provider notes this different
      { "dts:ref": "Av", "dts:citeType": "preface"},
      { "dts:ref": "Pr", "dts:citeType": "preface"},
      // Given the fact the following nodes have no citeType, they inherit of the root object citeType : letter
      { "dts:ref": "1" },
      { "dts:ref": "2" },
      { "dts:ref": "3" },
      // And so on
    ],
    "dts:passage": "/dts/api/documents/?id=http://data.bnf.fr/ark:/12148/cb11936111v{&ref}{&start}{&end}",
    "dts:parent": [null]
}
```


### Example 10: Retrieval of titles and generic metadata

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
        "foo": "http://foo.bar/ontology#"
    },
    "@id":"/api/dts/navigation/?id=http://data.bnf.fr/ark:/12148/cb11936111v",
    "dts:citeDepth" : 1,
    "dts:level": 1,
    "member": [
      {
        "dts:ref": "Av",
        "dts:dublincore": {
        "dc:title": "Avertissement de l'Éditeur"
        }
      },
      {
        "dts:ref": "Pr",
        "dts:dublincore": {
          "dc:title": "Préface"
        }
      },
      {
        "dts:ref": "1",
        "dts:dublincore": {
          "dc:title": "Lettre 1"
        },
        "dts:extensions": {
          "foo:fictionalSender": "Cécile Volanges",
          "foo:fictionalRecipient": "Sophie Carnay"
        }
      },
      {
        "dts:ref": "2",
        "dts:dublincore": {
          "dc:title": "Lettre 2"
        },
        "dts:extensions": {
          "foo:fictionalSender": "La Marquise de Merteuil",
          "foo:fictionalRecipient": "Vicomte de Valmont"
        }
      },
      {
        "dts:ref": "3",
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
    "dts:passage": "/dts/api/documents/?id=http://data.bnf.fr/ark:/12148/cb11936111v{&ref}{&start}{&end}",
    "dts:parent": [null]
}
```
