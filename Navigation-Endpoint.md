# Navigation Endpoint

The Navigation endpoint provides a JSON object listing passages that are accessible for navigation from a given reference within a resource. While the Collection endpoint allows for traversing of collections and their constituent Resources, the Navigation endpoint is used for traversing the internal citation tree of an individual Resource.

Responses from the Navigation endpoint assume by default that the user is traversing the document downward in the citation tree. A request to the Navigation endpoint specifies a reference within a resource, and the response lists the references that are immediately above or below that reference point in the citation tree. The response also identifies the parent reference of that current node.

## Scheme for Navigation Endpoint Responses

These are the JSON properties allowed in the object returned from a Navigation request. All top-level properties are mandatory.

| name |  description                              |
| ---- | -----------------------------------------|
| `@id` | the ID of the current request. This value should be the request URL including any query parameters. |
| `maxCiteDepth` | a number defining the maximum depth of the document's citation tree. *E.g.*, if the a document has up to three levels, `maxCiteDepth` should be the number 3. |
| `citeType` | defines the default type of references listed in `member`. |
| `level` | a number identifying the hierarchical level of the requested identifier (i.e., in the `ref` or `start`/`end` query parameters), counted relative to the top of the document's citation tree. *E.g.*, if a the requested identifier is at the second hierarchical level (like `{"ref": "1.1"}`) then the `level` in the response should be the number 2. (The Resource as a whole is considered level 0.) |
| `passage` | the URI template to the Document endpoint at which the text of passages corresponding to these references can be retrieved. |
| `parent` | the unique passage identifier for the hierarchical parent of the current node in the document structure, defined by the `ref` query parameter. If the query specifies a range rather than a single `ref`, no parent should be specified and `parent` should have a value of "null". |
| `member` | a list of passage references matching the requested parameters. This can be a list of single `ids` as objects with `ref` values: `[{"ref": "a"}, {"ref": "b"}, {"ref": "1.1"}]`. Or the list can contain ranges with `start` and `end` values in place of `ref` values: `[{"start": "a", "end": "b"}, {"start": "1.1", "end": "1.3"}]`. Each item in the member list should also include its own `level` parameter specifying the hierarchical level of that identifier in the document.|


Within the `member` list, each object may have the following properties:

| name | constraint | description                              |
| ---- | ---------- | -----------------------------------------|
| `ref` | required unless `start` and `end` supplied | the unique passage identifier for one node in the citation structure of the Resource |
| `start` | required if `ref` is not present; requires that `end` is also supplied | the unique passage identifier for the first member of a range of sequential passages. This parameter is inclusive, so the supplied reference is considered part of the specified range. |
| `end` | required if `ref` is not present; requires that `start` is also supplied | the unique passage identifier for the last member of a range of sequential passages. This parameter is inclusive, so the supplied reference is considered part of the specified range. |
| `citeType` | optional | identifies the passage type for the specified passage or range. *E.g.*, `{"ref": "1.2", "citeType": "Poem"}` |
| `dublinCore` | optional | contains Dublin Core Terms metadata for the specified passage or range. *E.g.*, `{ref": "1.2", "dublinCore": {"author": "Balzac"}}` |
| `extensions` | Optional | contains metadata for the specified passage or range from other namespaces |


### Unique `ref` identifiers

Note that all identifiers used as a `ref` value must be unique within the current resource. This is the case for the values retured in the `member` list as well as in the `parent` property.

### Values for the `parent` Property

The format for the returned `parent` value will depend on where the current `ref` stands in the resource's hierarchical structure.

- If the requested `ref` is the identifier for the **resource as a whole**, and that resource has no hierarchical parent, the value returned for `parent` should be "null".
- If the requested `ref` identifies **one of the top level** of the resource's hierarchical divisions, the `parent` property should be an object identifying the document as a whole and specifying that its `@type` is a "Resource". For example: `{"@type": "Resource", "@id": "urn:cts:greekLit:tlg0012.tlg001.opp-grc5"}`
- If the requested `ref` identifies **a node at a lower level** of the resource's hierarchical divisions, so that the parent is another division within the citation structure, the `parent` value will be a list of objects much like the list returned for the `member` property, each object identifying one reference that is the current node's direct parent. In this case, though, each object should also include an `@type` value of "CitableUnit". For example: `{"@type": "CitableUnit", "ref": "1.1.1"}`. If only one parent exists then a single object may be returned rather than an array of objects.
- If the request is **relative to a *range*** rather than a single *reference*, then the request again cannot be relied upon to have a single common hierarchical parent. So the `parent` value for a range request should again be the value "null". If a client wishes to discover the parent for the milestone references at the start and end of the range (specified in the "start" and "end" query parameters), a seprate request should be made for each of these references as individual locations using the "ref" parameter.


## URI for Navigation Endpoint Requests

### Query Parameters

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | the unique identifier (normally a URN) for the Resource being navigated |  GET    |
| ref | (NOT used with `start` and `end`) a single passage identifier providing the point of reference for the Navigation request within the Resource. Such identifiers should be unique within a given Resource. | GET    |
| start | (NOT used if a `ref` is specified, requires `end` as well) Start of the range passages to serve as the reference point for the Navigation request. This parameter is inclusive, so the supplied reference is considered part of the specified range. | GET |
| end |  (NOT used if a `ref` is specified, requires `start` as well) End of the range of passages to serve as the reference point for the Navigation request. This parameter is inclusive, so the supplied reference is considered part of the specified range. | GET |
| down | the depth (as a number or the string "max") for reference identifiers (members) to be retrieved, relative to the specified `ref` or `start`/`end` values. *E.g.*, if a request should return the children of the passage "1.2", then the `ref` parameter should be "1.2" and the `down` parameter should be `1`. The default value is `1`. If the descendants of the passage at the maximum depth of the document's structure are desired, a value of "max" may be supplied instead of a number. A value of `0` indicates that members should be at the same hierarchical level as the specified `ref` or `start`/`end` identifiers. When a `start` and `end` value for a range are supplied, a value of `0` indicates that the returned members should be all the references in the specified range *at the same hierarchical level* as the `start` and `end`. When a single `ref` value is supplied with a `down` value of 0, no members are returned, and the return object contains only metadata on the requested node itself.| GET    |
| groupBy | Retrieve passages in groups of this size instead of single units. This would normally mean that the `member` list returned would be a list of ranges, each of which contains this number of passages. | GET |
| max | Allows for limiting the number of results and getting pagination | GET |
| exclude | Exclude keys in members' object such as `exclude=extensions` | GET |

### Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |

### URI Template

Here is a template of the URI for Navigation API. The route itself (`/dts/api/navigation/`) is up to the implementer.

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
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
      "variable": "ref",
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
      "variable": "down",
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

### Example 1: Requesting top-level children of a textual Resource

The client wants to retrieve a list of passage identifiers that are part of the textual Resource identified as  *urn:cts:greekLit:tlg0012.tlg001.opp-grc5*. In other words, the client wants a list of the top-level references in the resource's citation structure. So the `id` parameter supplied in the query is the identifier of the Resource as a whole.

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&down=1`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&down=1",
    "maxCiteDepth" : 2,
    "level": 0,
    "member": [
      {"ref": "1", "level": 1},
      {"ref": "2", "level": 1},
      {"ref": "3", "level": 1}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "parent": null
}
```

### Example 2: Requesting all descendants of a textual Resource at a specified level

The client wants to retrieve a list of passage identifiers that are part of the textual Resource identified by *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and can be found at the second level of the citation tree of the document. Although the second structural level is being returned, we are not requesting the descendants of any single upper-level node. The references returned are in relation to the document as a whole. So the parent node is the whole document.

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&down=2`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&down=2",
    "maxCiteDepth" : 2,
    "level": 0,
    "member": [
      {"ref": "1.1", "level": 2},
      {"ref": "1.2", "level": 2},
      {"ref": "2.1", "level": 2},
      {"ref": "2.2", "level": 2},
      {"ref": "3.1", "level": 2},
      {"ref": "3.2", "level": 2}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "parent": {"@type": "Resource", "@ref": "/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc"}
}
```

### Example 3: Requesting children of one top-level structural division of a Resource

The client wants to retrieve a list of passage identifiers that are children of the textual Resource identified by *urn:cts:greekLit:tlg0012.tlg001.opp-grc5* and its division `1`. Since the desired members are direct children of the specified reference, the `down` parameter may be omitted and the default value of `1` will be used. Since the specified reference, the point from which the descendants are viewed, is at the top level of the structural hierarchy, the parent in the return value is still the document as a whole.

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&ref=1`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&ref=1",
    "maxCiteDepth" : 2,
    "level": 1,
    "member": [
      {"ref": "1.1", "level": 2},
      {"ref": "1.2", "level": 2}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "parent": {"@type": "Resource", "@ref": "/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc"}
}
```

### Example 4: Requesting grandchild descendants of a top-level structural division

The client wants to retrieve a list of grand-children passage identifiers that are part of the textual Resource identified by *urn:cts:latinLit:phi1294.phi001.perseus-lat2* and its passage `1`. Since the specified reference, the point from which the descendants are viewed, is at the top level of the structural hierarchy, the parent in the return value is still the document as a whole.

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1&down=2`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1&down=2",
    "maxCiteDepth" : 3,
    "level": 1,
    "member": [
      {"ref": "1.1.1", "level": 3},
      {"ref": "1.1.2", "level": 3},
      {"ref": "1.2.1", "level": 3},
      {"ref": "1.2.2", "level": 3}
    ],
    "passage": "/dts/api/document/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2{&ref}{&start}{&end}",
    "parent": {"@type": "Resource", "ref": "/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2"}
}
```

### Example 5: Requesting children of a lower-level structural division

The client wants to retrieve a list of child passage identifiers that are part of the textual Resource identified by *urn:cts:latinLit:phi1294.phi001.perseus-lat2* and its passage "1.1". The returned parent is the direct parent (or parents) of the specified reference ("1.1"). Since it is not the document as a whole, the parent `@type` is "CitableUnit" rather than "Resource".

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1.1&down=1`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"/api/dts/navigation/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2&ref=1.1&down=1",
    "maxCiteDepth" : 3,
    "level": 2,
    "member": [
      {"ref": "1.1.1", "level": 3},
      {"ref": "1.1.2", "level": 3}
    ],
    "passage": "/dts/api/document/?id=urn:cts:latinLit:phi1294.phi001.perseus-lat2{&ref}{&start}{&end}",
    "parent": {"@type": "CitableUnit", "ref": "1"}
}
```

### Example 6: Requesting a range of passage references between milestones

The client wants to retrieve a list of passage identifiers which are between two milestones. In this case there is no single parent node shared by the whole requested range, so no parent is returned. Since the reference list to be returned is at the *same* structural level as the supplied milestones, the `down` query parameter is "0".

<!---FIXME: parent retrieval requires separate requests !-->

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&down=0&start=1&end=3`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&down=0&start=1&end=3",
    "maxCiteDepth" : 2,
    "level": 1,
    "member": [
      {"ref": "1", "level": 1},
      {"ref": "2", "level": 1},
      {"ref": "3", "level": 1}
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "parent": null
}
```

### Example 7: Requesting descendants of the references in a specified range

The client wants to retrieve a list of passage identifiers which are between two milestones. Since the `level` query parameter is "1", the returned references are one level below the milestones specified. I.e., the returned references are all *children* of structural divisions between the two milestones. In this case there is no single parent node shared by the whole requested range, so no parent is returned.

#### Example of url :

- `/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&down=1&start=1&end=3`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"/api/dts/navigation/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc&down=1&start=1&end=3",
    "maxCiteDepth" : 2,
    "level": 1,
    "member": [
      {"ref": "1.1", "level": 2},
      {"ref": "1.2", "level": 2},
      {"ref": "2.1", "level": 2},
      {"ref": "2.2", "level": 2},
      {"ref": "3.1", "level": 2},
      {"ref": "3.2", "level": 2},
    ],
    "passage": "/dts/api/document/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc{&ref}{&start}{&end}",
    "parent": null
}
```

### Example 8: Retrieval of typology of references

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
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"/api/dts/navigation/?id=http://data.bnf.fr/ark:/12148/cb11936111v",
    "maxCiteDepth" : 1,
    "level": 0,
    "citeType": "letter",
    "member": [
      // The two following items are not letters : the data provider notes this different
      { "ref": "Av", "citeType": "preface", "level": 1},
      { "ref": "Pr", "citeType": "preface", "level": 1},
      // Given the fact the following nodes have no citeType, they inherit of the root object citeType : letter
      { "ref": "1", "level": 1},
      { "ref": "2", "level": 1},
      { "ref": "3", "level": 1},
      // And so on
    ],
    "passage": "/dts/api/document/?id=http://data.bnf.fr/ark:/12148/cb11936111v{&ref}{&start}{&end}",
    "parent": null
}
```


### Example 9: Retrieval of titles and generic metadata

The client wants the list of passages with their title and domain-specific vocabularies. If the given data provider has a title, then it will be provided.

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
        "@vocab": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
        "foo": "http://foo.bar/ontology#"
    },
    "@id":"/api/dts/navigation/?id=http://data.bnf.fr/ark:/12148/cb11936111v",
    "maxCiteDepth" : 1,
    "level": 0,
    "member": [
      {
        "ref": "Av",
        "level": 1,
        "dublinCore": {
        "title": "Avertissement de l'Éditeur"
        }
      },
      {
        "ref": "Pr",
        "level": 1,
        "dublinCore": {
          "title": "Préface"
        }
      },
      {
        "ref": "1",
        "level": 1,
        "dublinCore": {
          "title": "Lettre 1"
        },
        "extensions": {
          "foo:fictionalSender": "Cécile Volanges",
          "foo:fictionalRecipient": "Sophie Carnay"
        }
      },
      {
        "ref": "2",
        "level": 1,
        "dublinCore": {
          "title": "Lettre 2"
        },
        "extensions": {
          "foo:fictionalSender": "La Marquise de Merteuil",
          "foo:fictionalRecipient": "Vicomte de Valmont"
        }
      }
      {
        "ref": "3",
        "level": 1,
        "dublinCore": {
          "title": "Lettre 3"
        },
        "extensions": {
          "foo:fictionalSender": "Cécile Volanges",
          "foo:fictionalRecipient": "Sophie Carnay"
        }
      },
      // And so on
    ],
    "passage": "/dts/api/document/?id=http://data.bnf.fr/ark:/12148/cb11936111v{&ref}{&start}{&end}",
    "parent": null
}
```
