Navigation Endpoint
====================

## Introduction

The Navigation endpoint provides information about a Resource's internal structures (e.g., book, chapter, line, etc.) and how they are referenced. While the Collection endpoint allows for finding Resources within a collection, the Navigation endpoint describes the relationship between structural divisions in the text. References found in the Navigation endpoint can also be used to retrieve corresponding portions of the Resource's text from the Document endpoint.

The Navigation endpoint provides JSON objects that describe sections in a Resource's citation tree. A citation tree is represented by a structured hierarchy of `CitableUnit` objects. In general, applications using the Navigation endpoint are moving down and forward through the document.

Using the Navigation endpoint, clients will be able to:

- construct a table of contents
- display a novel by chapter or a book of poetry by poem
- find out how to traverse a Resource from one structural section to another
- construct links to specific sections of the Resource
- find metadata about a specific section of a Resource (e.g., the writer of one letter in a correspondence)

## What Is a Citation Tree?

<!-- TODO: Add text and image here --->
When discussing a text, it is common to use labels to identify sections of the text. When referring to pages, for example, we might identify them with numbers. In other situations we might divide the text into logical structures that are hierarchically organized. For example, we might identify "Paragraph 4" within "Section D" which falls within "Chapter 1." In both cases, these structuring labels form citation trees, though with different depths. (A depth of 1 for "pages" and a depth of 3 for "chapter"/"section"/"paragraph.")

The depth of a citation tree can be uneven. In a Ph.D. thesis, for example, the introduction may not be subdivided, while a chapter will often have subdivisions.

It can be that the same text can be discussed using more than one different structuring hierarchy. In the context of a text that has both been printed and made available online, it is not rare to find people referring to the printed version by "page" and the web version by "paragraph." These competing citation trees can often lead to confusion.

Taking the novel *Dracula*, the text can be modeled as follows:

![Citation tree example diagram](assets/img/citation-tree_dracula-example.png)

## Scheme for Navigation Endpoint Responses

### `Navigation` Object

The top-level response object is a `Navigation` object answering a query about the citation tree of a Resource, containing details about nodes within that tree.

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `@id` | URL | Y | The absolute URL of the current request including any query parameters. |
| `@type` | string | Y | The object's RDF class which must be "Navigation". |
| `passage` | URL template | Y | The URI template to the Document endpoint at which the text of nodes in the citation tree can be retrieved. |
| `navigation` | URL template | Y | The URI template to the Navigation endpoint at which the citation tree structure can be queried. |
| `resource`| Resource | Y | The `Resource` whose citation tree is being queried. |
| `ref` | CitableUnit | N | The `CitableUnit` in the citation tree which is being queried. |
| `start` | CitableUnit | N | The `CitableUnit` at the beginning of the range in the citation tree which is being queried. |
| `end` | CitableUnit | N |  The `CitableUnit` at the end of the range in the citation tree which is being queried. |
| `member` | list | N | A list of `CitableUnit` in the subtree specified by the query parameters. |

Because the `Navigation` object is a top-level object in the API, each object must also have a `@context` property pointing to a DTS JSON-LD context object such as "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json".

### Resource

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `@id` | URI | Y | The URI of the `Resource`. |
| `@type` | string | Y | The object's RDF class which must be "Resource". |
| `collection` | URL template | Y | The URI template to the Collection endpoint at which the `Resource` can be found. |
| `citationTrees` | list | Y | A list of `CitationTree` objects |

If a `Resource` has a single `CitationTree`, that `CitationTree` object cannot have an `identifier`.

If a `Resource` has multiple `CitationTree`s, then the first listed in `citationTrees` is the default `CitationTree` and cannot have an `identifier`.

If a `Resource` is a document without a citation tree, the `citationTrees` property is an empty list.

### CitationTree

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `identifier` | string | N | The string identifier of the `CitationTree`. |
| `@type` | string | Y | The object's RDF class which must be "CitationTree". |
| `citeStructure` | list | N | A list of `CiteStructure` objects. |
| `maxCiteDepth` | int | Y | An integer defining the maximum depth of the Resource's citation tree. |
| `description` | string | N | A human readable description of the citation tree. |

### CiteStructure

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `citeStructure` | list | N | A list of `CiteStructure` objects. |
| `citeType` | string | N | A type of textual unit that appears at a given level in the citation tree. (E.g., "chapter", "verse") |

### CitableUnit

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `identifier` | string | Y | The string identifier of the `CitableUnit`. |
| `@type` | string | Y | The object's RDF class which must be "CitableUnit". |
| `level` | int | Y | A number identifying the depth at which the `CitableUnit` is found within the citation tree of the `Resource`. |
| `parent` | nullable string | Y | The string identifier of the hierarchical parent of the `CitableUnit` in the `Resource`. |
| `citeType` | string | N | The type of textual unit corresponding to the `CitableUnit` in the Resource. (E.g., "chapter", "verse") |
| `dublinCore` | object | N | Dublin Core Terms metadata describing the `CitableUnit`. |
| `extensions` | object | N | Metadata for the `CitableUnit` from vocabularies other than Dublin Core Terms. |

#### Unique `identifier`s

The `identifier` of a `CitableUnit` must be unique within a `CitationTree` for a `Resource`.

#### Values for the `parent` Property

If the `CitableUnit` parent is the root level of the `Resource`, the value returned for `parent` should be `null`.

```json
{"identifier": "Luke",
 "level": 1,
 "@type": "CitableUnit",
 "parent": null}
```

## URI for Navigation Endpoint Requests

### Query Parameters

| Name | Type | Description                              | Methods | Constraints |
|------|------ | ------------------------------------|---------| ----------- |
| resource   | URI | The unique identifier for the Resource being navigated |  GET    ||
| ref | string | The string identifier of a single node in the citation tree for the Resource, used as the point of reference for the query. | GET    | NOT used with `start` and `end` |
| start | string | The string identifier of a node in the citation tree for the resource, used as the starting point for a range that serves as the reference point for the query. This parameter is inclusive, so the starting point is considered part of the specified range. | GET | NOT used if a `ref` is specified, requires `end` as well |
| end |  string | The string identifier of a node in the citation tree for the resource, used as the ending point for a range of passages that serves as the reference point for the query. This parameter is inclusive, so the supplied ending point is considered part of the specified range. | GET | NOT used if a `ref` is specified, requires `start` as well |
| down | int | The maximum depth of the citation subtree to be returned, relative to the specified `ref`, the deeper of the `start`/`end` `CitableUnit`, or if these are not provided relative to the root. A value of `-1` indicates the bottom of the `Resource` citation tree. | GET    |If `down` is not provided only retrieve information about the queried `CitableUnit` |
| tree | string | The string identifier for a `CitationTree` of the `Resource`. | GET | NOT used to query the default `CitationTree` |
| page | int | The number of identifying a page in paginated query results. | GET | |

#### Errors

Some combinations of query parameters and their values must return a 4XX HTTP Error. A 400 Bad Request Error should be returned when:
  - no `resource` value is provided
  - both `ref` and either `start` or `end` is specified
  - `start` is provided without also specifying `end`, or vice versa
A 404 Not Found Error should be returned when
  - a query specifies a `ref`, `start`, or `end` value that does not exist in the queried `CitationTree`
  - a query specifies a `tree` value that does not correspond to an existing `CitationTree` for the `Resource`

#### Usage of `tree`

All requests on a `Resource` without any `CitationTree`s at the `Navigation` endpoint will return an empty `member` list and must not return an HTTP error.

If no `tree` parameter is specified, the default `CitationTree` of the `Resource` must be used to traverse the `Resource`.

#### Usage of `down`

| `down` | `ref` | `start`/`end` | Result |
| ------ | ----- | ------------- | ------ |
| absent | absent | absent | 400 Bad Request Error |
| absent | present | absent | Information about the `CitableUnit` identified by `ref`. No member property in the `Navigation` object. |
| absent | absent | present | Information about the `CitableUnit`s identified by `start` and by `end`. No member property in the `Navigation` object. |
| 0 | present | absent | Information about the `CitableUnit` identified by `ref` along with a `member` property that is a list of `CitableUnit`s that are siblings (sharing the same parent) including the current `CitableUnit` identified by `ref`. |
| 0 | absent | present | 400 Bad Request Error |
| > 0 | absent | absent | A `member` list of `CitableUnit`s including the citation tree from the root to the depth requested in `down`. |
| > 0 | present | absent | A `member` list of `CitableUnit`s including the citation tree from the `CitableUnit` identified by `ref` to the depth requested in `down`. |
| > 0 | absent | present | A `member` list of `CitableUnit`s including the citation tree between the `start` and `end` `CitableUnit`s inclusive, down to a depth relative to the deeper of the `CitableUnit`s identified by `start` and `end`. |
| -1 | absent | absent | A `member` list of `CitableUnit`s including the citation tree from the root to the deepest level of the `CitationTree`. |
| -1 | present | absent | A `member` list of `CitableUnit`s including the citation tree from the `CitableUnit` identified by `ref` to the deepest level of the `CitationTree`. |
| -1 | absent | present | A `member` list of `CitableUnit`s including the citation tree between the `start` and `end` `CitableUnit`s inclusive, down to the deepest level of the `CitationTree`. |

#### Handling Requests with No Matching `CitableUnit`s

A `Navigation` endpoint request may specify a level in a `Resource`'s `CitationTree` that does not exist. One may, e.g., provide a `down` value of `3` when only `1` lower level exists in the `Resource`'s `CitationTree`. In this case the `member` list will simply include any `CitableUnit`s that do satisfy the parameters.

If there are no `CitableUnit`s at all that satisfy the parameters of a `Navigation` endpoint request:

- the request must not raise an error
- the `Navigation` object `member` property must be an empty list.

For example, if the `ref` is at the bottom level of the queried `CitationTree`, and a `down` of 2 is provided in the request, the response will provide an empty list as its `member` value.

#### Order of `CitableUnit`s in `member`

The `CitableUnits` listed in the returned `member` property must be in document order as defined in the XPath 3.1 specification: https://www.w3.org/TR/xpath-31/#dt-document-order.

This is a "pre-order, depth first" traversal moving through nodes of the citation tree as follows:

!["pre-order, depth first traversal example"](./assets/img/tree-traversal_example.png)

### URI Template

Here is a template of the URI for Navigation API. The route itself (`/dts/api/navigation/`) is up to the implementer.

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
  "@type": "IriTemplate",
  "template": "/dts/api/navigation{?resource,ref,down,start,end,tree,page}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "resource",
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
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "tree",
      "property": "hydra:freetextQuery",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "page",
      "property": "hydra:freetextQuery",
      "required": false
    }
  ]
}
```

## Response Headers

Responses from the `Navigation` endpoint should include an HTTP response header identifying the `Content-Type` of the response as `application/ld+json`.

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

## Examples

### Writing Conventions for the Examples

In the following examples, JavaScript comments such as /* ... */ are used to indicate that more information may be found at this point in the Response body.

### Example 1: Requesting top-level `CitableUnit`s of a `Resource`

The client wants to retrieve a list of `CitableUnit`s that are part of the `Resource` with the identifier "https://en.wikisource.org/wiki/Dracula". In other words, the client wants a list of the top-level nodes in the resource's citation tree.

#### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula=&down=1`

#### Response Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response Body

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula=&down=1",
    "passage": "https://example.org/dts/api/document/{?resource,ref,start,end,format}",
    "collection": "https://example.org/dts/api/collection/{?resource,page,nav}",
    "navigation": "https://example.org/dts/api/navigation/{?resource,ref,down,start,end,tree,page}",
    "resource": {
      "@id": "https://en.wikisource.org/wiki/Dracula",
      "@type": "Resource",
      "citationTrees": [
        {
          "@type": "CitationTree",
          "maxCiteDepth" : 3,
          "citeStructure": [
            {
              "@type": "CiteStructure",
              "citeType": "Chapter",
              "citeStructure": [
                {
                  "@type": "CiteStructure",
                  "citeType": "Journal Entry",
                  "citeStructure": [
                    {
                      "@type": "CiteStructure",
                      "citeType": "Paragraph"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    },
    "member": [
      {
        "identifier": "C1",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter",
        "dublinCore": {
          "title": "Chapter 1: Jonathan Harker's Journal"
        }
      },
      {
        "identifier": "C2",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter",
        "dublinCore": {
          "title": "Chapter 2: Jonathan Harker's Journal"
        }
      },
      {
        "identifier": "C3",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter",
        "dublinCore": {
          "title": "Chapter 3: Jonathan Harker's Journal"
        }
      },
    ],
}
```

### Example 2: Requesting the `Resource`'s complete citation tree down to a specified level

The client wants to retrieve a list of all `CitableUnit`s in the `Resource` identified as "https://en.wikisource.org/wiki/Dracula" down to the second level of the `Resource`'s citation tree (in this case chapters and their paragraphs). We are not requesting the descendants of any single upper-level node (i.e., chapter), so response will provide the entire top two levels of the citation tree.

#### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula=&down=2`

#### Response Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response Body

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=2",
    "passage": "https://example.org/dts/api/document/{?resource,ref,start,end,format}",
    "collection": "https://example.org/dts/api/collection/{?resource,page,nav}",
    "navigation": "https://example.org/dts/api/navigation/{?resource,ref,down,start,end,tree,page}",
    "resource": {
      "@id": "https://en.wikisource.org/wiki/Dracula",
      "@type": "Resource",
      "citationTrees": [
        {
          "@type": "CitationTree",
          "maxCiteDepth" : 3,
          "citeStructure": [
            {
              "@type": "CiteStructure",
              "citeType": "Chapter",
              "citeStructure": [
                {
                  "@type": "CiteStructure",
                  "citeType": "Journal Entry",
                  "citeStructure": [
                    {
                      "@type": "CiteStructure",
                      "citeType": "Paragraph"
                    }
                  ]
                }
              ]
            }
          ]
        }

      ]
    },
    "member": [
      {
        "identifier": "C1",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter"
      },
      {
        "identifier": "C1.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C1.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C2",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter"
      },
      {
        "identifier": "C2.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C2",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C2.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C2",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C2.E3",
        "@type": "CitableUnit",
        "level": 1,
        "parent": "C2",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C3",
        "@type": "CitableUnit",
        "level": 2,
        "parent": null,
        "citeType": "Chapter"
      },
      {
        "identifier": "C3.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C3",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C3.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C3",
        "citeType": "Journal Entry"
      },
    ],
}
```

### Example 3: Requesting all descendants of one top-level `CitableUnit` of a `Resource`

The client wants to retrieve a list of all `CitableUnit`s in the `Resource` identified by "https://en.wikisource.org/wiki/Dracula" that are children of its `Citable Unit` with the identifier "C1". In other words, the client is requesting the entire citation subtree below `CitableUnit` "C1".

#### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1&down=-1`

#### Response Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response Body

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1&down=-1",
    "passage": "https://example.org/dts/api/document/{?resource,ref,start,end,format}",
    "collection": "https://example.org/dts/api/collection/{?resource,page,nav}",
    "navigation": "https://example.org/dts/api/navigation/{?resource,ref,down,start,end,tree,page}",
    "resource": {
      "@id": "https://en.wikisource.org/wiki/Dracula",
      "@type": "Resource",
      "citationTrees": [
        {
          "@type": "CitationTree",
          "maxCiteDepth" : 3,
          "citeStructure": [
            {
              "@type": "CiteStructure",
              "citeType": "Chapter",
              "citeStructure": [
                {
                  "@type": "CiteStructure",
                  "citeType": "Journal Entry",
                  "citeStructure": [
                    {
                      "@type": "CiteStructure",
                      "citeType": "Paragraph"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    },
    "member": [
      {
        "identifier": "C1",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter"
      },
      {
        "identifier": "C1.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C1.E1,P1",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E1",
        "citeType": "Paragraph"
      },
      {
        "identifier": "C1.E1,P2",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E1",
        "citeType": "Paragraph"
      },
      {
        "identifier": "C1.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C1.E2,P1",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E2",
        "citeType": "Paragraph"
      },
      {
        "identifier": "C1.E2,P2",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E2",
        "citeType": "Paragraph"
      },
    ],
}
```

### Example 4: Requesting descendants of a top-level `CitableUnit` down to grandchildren

The client again wants to retrieve a list of `CitableUnit`s in the citation tree for the `Resource` identified by "https://en.wikisource.org/wiki/Dracula" that are descendants its `CitableUnit` with the identifier "C1" (entitled "Chapter 1"), but only down to the second level of the citation tree below "C1". In other words, we are retrieving the subtree below `CitableUnit` "C1" but including only three levels of the tree, "C1" itself and the two levels below.

(Note that since the resource "https://en.wikisource.org/wiki/Dracula" has only three levels to its citation tree, the resulting `member` list will have the same contents as the one returned in Example 3, even though the query parameters differ.)

#### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula?ref=C1&down=2`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula?ref=C1&down=2",
    "passage": "https://example.org/dts/api/document/{?resource,ref,start,end,format}",
    "collection": "https://example.org/dts/api/collection/{?resource,page,nav}",
    "navigation": "https://example.org/dts/api/navigation/{?resource,ref,down,start,end,tree,page}",
    "resource": {
      "@id": "https://en.wikisource.org/wiki/Dracula",
      "@type": "Resource",
      "citationTrees": [
        {
          "@type": "CitationTree",
          "maxCiteDepth" : 3,
          "citeStructure": [
            {
              "@type": "CiteStructure",
              "citeType": "Chapter",
              "citeStructure": [
                {
                  "@type": "CiteStructure",
                  "citeType": "Journal Entry",
                  "citeStructure": [
                    {
                      "@type": "CiteStructure",
                      "citeType": "Paragraph"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    },
    "member": [
      {
        "identifier": "C1",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter"
      },
      {
        "identifier": "C1.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C1.E1,P1",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E1",
        "citeType": "Paragraph"
      },
      {
        "identifier": "C1.E1,P2",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E1",
        "citeType": "Paragraph"
      },
      {
        "identifier": "C1.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C1.E2,P1",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E2",
        "citeType": "Paragraph"
      },
      {
        "identifier": "C1.E2,P2",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E2",
        "citeType": "Paragraph"
      },
    ],
}
```

### Example 5: Requesting children of a lower-level `CitableUnit`

The client wants to retrieve a list of `CitableUnit`s that are part of the `Resource` "https://en.wikisource.org/wiki/Dracula", including the unit "C1.E1" and its direct children.

#### Example of url:

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1.E1&down=1`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id":"https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1.E1&down=1",
    "passage": "https://example.org/dts/api/document/{?resource,ref,start,end,format}",
    "collection": "https://example.org/dts/api/collection/{?resource,page,nav}",
    "navigation": "https://example.org/dts/api/navigation/{?resource,ref,down,start,end,tree,page}",
    "resource": {
      "@id": "https://en.wikisource.org/wiki/Dracula",
      "@type": "Resource",
      "citationTrees": [
        {
          "@type": "CitationTree",
          "maxCiteDepth" : 3,
          "citeStructure": [
            {
              "@type": "CiteStructure",
              "citeType": "Chapter",
              "citeStructure": [
                {
                  "@type": "CiteStructure",
                  "citeType": "Journal Entry",
                  "citeStructure": [
                    {
                      "@type": "CiteStructure",
                      "citeType": "Paragraph"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    },
    "member": [
      {
        "identifier": "C1.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C1.E1,P1",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E1",
        "citeType": "Paragraph"
      },
      {
        "identifier": "C1.E1,P2",
        "@type": "CitableUnit",
        "level": 3,
        "parent": "C1.E1",
        "citeType": "Paragraph"
      }
    ],
}
```

### Example 6: Requesting descendants of the `CitationUnit`s in a specified range

The client wants to retrieve a list of `CitableUnit`s in a specified range, including the direct children of the specified `start` and `end` points. Since the `level` parameter has a value of `1`, the returned `member` list includes two levels of the citation tree for the specified range.

#### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=1&start=C1&end=C3`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=1&start=C1&end=C3",
    "passage": "https://example.org/dts/api/document/{?resource,ref,start,end,format}",
    "collection": "https://example.org/dts/api/collection/{?resource,page,nav}",
    "navigation": "https://example.org/dts/api/navigation/{?resource,ref,down,start,end,tree,page}",
    "resource": {
      "@id": "https://en.wikisource.org/wiki/Dracula",
      "@type": "Resource",
      "citationTrees": [
        {
          "@type": "CitationTree",
          "maxCiteDepth" : 3,
          "citeStructure": [
            {
              "@type": "CiteStructure",
              "citeType": "Chapter",
              "citeStructure": [
                {
                  "@type": "CiteStructure",
                  "citeType": "Journal Entry",
                  "citeStructure": [
                    {
                      "@type": "CiteStructure",
                      "citeType": "Paragraph"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    },
    "member": [
      {
        "identifier": "C1",
        "@type": "CitableUnit",
        "level": 1,
        "parent": "C1",
        "citeType": "Chapter"
      },
      {
        "identifier": "C1.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C1.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C2",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter"
      },
      {
        "identifier": "C2.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C2",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C2.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C2",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C3",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter"
      },
      {
        "identifier": "C3.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C3",
        "citeType": "Journal Entry"
      },
      {
        "identifier": "C3.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C3",
        "citeType": "Journal Entry"
      },
    ],
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
