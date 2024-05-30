Specifications
===============================

## Status of this Document

- This Version: unstable
- Previous Version: [1-alpha](./../1-alpha/)

<span style="color : red;">The current version is the closest we got to a release candidate. Feel free to provide feedback, typo corrections, etc.</span>

## Changelogs

- 2024-05-24
  - Removed `totalItems` from the Collection Endpoints (See https://github.com/distributed-text-services/specifications/issues/248, problem pointed out by @kbrueckmann)
  - `passage` property (URI template) moved to `document` for consistency between Collection and Navigation endpoint (See https://github.com/distributed-text-services/specifications/issues/249, problem pointed out by @philippepons)
    - Harmonized the examples to have the resource already populated, as per Collection URI Templates
  - `collection` property (URI template) added to `Collection` and `Resource` object (See https://github.com/distributed-text-services/specifications/issues/250, problem pointed out by @philippepons) 
  - Fixed a typo in the Entry endpoint table, where it referenced Resources.

## Editors

Editors:

- Hugh Cayless
- Thibault Clérice
- Jonathan Robie
- Ian Scott
- Bridget Almas

Published by the editors under the CC-BY 4.0 license. 

### How to cite:

**APA:** 

> Cayless, H., Clérice, T., Jonathan, R., Scott, I., & Almas, B. Distributed Text Services Specifications (Version 1-alpha) [Computer software]. https://github.com/distributed-text-services/specifications`

**Bibtex**:

```latex
@software{Cayless_Distributed_Text_Services,
author = {Cayless, Hugh and Clérice, Thibault and Jonathan, Robie and Scott, Ian and Almas, Bridget},
license = {CC-BY-4.0},
title = {{Distributed Text Services Specifications}},
url = {https://github.com/distributed-text-services/specifications},
version = {1-alpha}
}
```

## Introduction

The Distributed Text Services (DTS) provides a standard way for clients to interact with collections of documents. A standard API allows users to access many text collections using the same client software. It also allows editors to publish text collections in a usable way that existing clients can process.

The DTS specifications was built for supporting TEI documents, but can support any kind of textual document.

## Structure

The Distributed Text Services API implements one root [Entry point](#entry-endpoint) and three different endpoints:

- Navigation _across texts_ is supported by the [Collection Endpoint](#collection-endpoint)
- Navigation _within a text_ is supported by the [Navigation Endpoint](#navigation-endpoint)
- Retrieval of complete or partial texts is supported by the [Document Endpoint](#document-endpoint)

## Concepts

### General vocabulary

<!-- Needs improvement maybe -->

- *Collection*: a named aggregation of digital `Resource`s. Collections may contain other Collections or Resources.
- *Resource*: A document
- *Citable Unit*: A portion of a Resource identified by a reference string.
f
<!-- 
- *Citation Tree*: A list of references, in document order, corresponding to the hierarchical structure of a Resource.
 -->

### Citation Tree

When discussing a text, it is common to use labels to identify sections of the text. When referring to pages, for example, we might identify them with numbers. In other situations we might divide the text into logical structures that are hierarchically organized. For example, we might identify "Paragraph 4" within "Section D" which falls within "Chapter 1." In both cases, these structuring labels form citation trees, though with different depths. (A depth of 1 for "pages" and a depth of 3 for "chapter"/"section"/"paragraph.")

The depth of a citation tree can be uneven. In a Ph.D. thesis, for example, the introduction may not be subdivided, while a chapter will often have subdivisions.

It can be that the same text can be discussed using more than one different structuring hierarchy. In the context of a text that has both been printed and made available online, it is not rare to find people referring to the printed version by "page" and the web version by "paragraph." These competing citation trees can often lead to confusion.

Taking the novel *Dracula*, the text can be modeled as follows:

![Citation tree example diagram](assets/img/citation-tree_dracula-example.png)

## About URI Templates

The Distributed Text Services specifications reuse the RFC 6570 for defining the syntax of *URI templates* used to build URL on the API in its various endpoints.

The RFC 6570 provides a variety of ways to describe URIs "query parameters" using template. IN RFC 6570, parameters are wrapped in `{}`. Multiple parameters may be wrapped together using `,` to join them. Parameters can be prefixed so that their resolution is the following:

> Example values are: the `var` parameter has a value "value", `x` has the value "x". `empty` is empty, `null` is [known to the processor to be nullable](https://datatracker.ietf.org/doc/html/rfc6570#section-2.3) and has the null value.


|  Example Template | Expansion                          |
| ----------------- | ---------------------------------- |
|    {var}          | value                             |
|    {/var}         | /value                             |
|    {;var}         | ;var=value                         |
|    {?var}         | ?var=value                         |
|    {&var}         | &var=value                         |
|    {/var,x}       | /value/1024                        |
|    {?var,x}       | ?var=value&x=1024                  |
|    {&var,x}       | &var=value&x=1024                  |
|    {?var,x,empty} | ?var=value&x=1024&empty=           |
|    {?var,x,nullv} | ?var=value&x=1024                  |

*Source: Multiple value example from the [RFC 6570](https://datatracker.ietf.org/doc/html/rfc6570#section-3.2.1)*

If you take the Collection endpoints, which is based around two main parameters, `id` and `nav`, a URI template using the query parameters **MAY** be: `/api/dts/collection/{?id,nav}` which provides for the client a way to build URLs such that `/api/dts/collection/?id=https://en.wikisource.org/wiki/Dracula` works.

### URI Templates examples for static website

If a user were to define a `Collection` endpoint using static files, without any ability to parse query parameters, their URI template **MAY** be `/api/dts/collection/{/id,nav*}` which, in turn, helps build URLs such as `/api/dts/collection/Dracula/children`. Note such implementations limits the usage of identifier for `Collection`.`@id` to string, as any URL would break path parsing.

For endpoints with many optional parameters, such as the `Navigation` endpoint, a static implementation **MAY** look like `/navigation/{resource}{;tree}{;down}{/ref,start,end}`. As `tree` and `down` are optional parameters, there **MUST** be a way to understand their presence or absence. On the other hand, the parameters `ref`, `start` and `end` are mutually excluding, such that the definition of the URI template allows for using this syntax of the RFC 6570. For such a URI template, `/navigation/resource/Dracula;down=-1/5` would produce the equivalent of `/navigation/?resource=Dracula&ref=5&down=-1` with the `tree` parameter being undefined.

## Conformance

This section defines the conformance requirements for DTS servers.

DTS does not define conformance requirements for clients.  Any client
that can use a DTS server successfully for its own purposes can be
called a DTS client.  Few clients will use all DTS features.

All DTS servers **MUST** implement all endpoints:

- Entry
- Collection
- Document
- Navigation

All DTS servers **MUST** implement all valid calls on these endpoints as
documented, with one exception:

A Level 0 DTS Server need not support the `start` and `end` parameters
for the Navigation endpoint.  It **SHOULD** raise an error if either of
these parameters are used in a request.

Note: Level 0 conformance allows minimal DTS implementations,
including implementations created by generating static files.

A Level 1 DTS Server supports all valid calls as documented in this
specification.

## Entry Endpoint

A client MUST be able to retrieve the adress of the various endpoints at the root of the API. This endpoint provides this information.

### Scheme for Base API Endpoint Responses

Everything that is not marked as Optional is mandatory.

JSON wide attributes :

- `@context` provides DCT, TEI and DTS namespace prefixes

Item properties :

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `@id` | URI | Y | The identifier of the API, generally its URL. |
| `@type` | `EntryPoint` | Y | Default Value: `EntryPoint`. |
| `dtsVersion` | string | Y | The version of the DTS specification providing the response. Default is "1-alpha". |
| `collection` | URI Template | Y | Link to the Collection API endpoint. |
| `navigation` | URI Template | Y | Link to the Navigation API endpoint. |
| `document` | URI Template | Y | Link to the Document API endpoint. |


### Example

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
  "dtsVersion": "1-alpha",
  "@id": "/api/dts/",
  "@type": "EntryPoint",
  "collection": "/api/dts/collection/{?id,page,nav}",
  "navigation" : "/api/dts/navigation/{?resource,ref,start,end,down,tree,page}",
  "document": "/api/dts/document/{?resource,ref,start,end,tree,mediaType}"
}
```

## Collection Endpoint

The collection endpoint is used for navigating collections. A collection contains metadata for the collection itself and an array of members. Each member is either a `Collection` or a `Resource`.

DTS does not specify any particular hierarchy of collections. A collection might provide all `Resource`s in a flat collection or a collection hierarchy organized by geography, time, or any other convenient logical grouping. The same `Collection` or `Resource` may exist in more than one collection.

### Scheme for Collection API Responses

Everything that is not marked as Optional is mandatory.

JSON wide attributes :

- `@context` provides DCT, TEI and DTS namespace prefixes

Item properties :


| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `@id` | URI | Y | The identifier of the `Collection` or `Resource`. |
| `@type` | `Collection` or `Resource` | Y | The type |
| `dtsVersion` | string | Y | The version of the DTS specification providing the response. Default is "1-alpha". |
| `title` | string | Y | A name for the Collection or Resource. Additional names may be placed in `dublinCore` using `title`, e.g. for internationalization. |
| `totalParents` | int | Y | Total number of parent `Collection`s or `Resource`s. |
| `maxCiteDepth` | int | Y (if `@type` is `Resource`) | The maximum depth of the Citation Tree. |
| `description` | string | N | Short description of the `Collection` or `Resource`. Additional descriptions may be placed in `dublinCore` using `description`, e.g. for internationalization. |
| `member` | array | N | Contains members of the collection. |
| `dublinCore` | MetadataObject | N | Contains metadata following the Dublin Core Terms scheme. |
| `extensions` | MetadataObject | N | Contains any supplementary metadata following other schemes. |
| `collection` | URI Template | Y | Link to the Collection API endpoint for the `Resource`. |
| `navigation` | URI Template | Y (if `@type` is `Resource`) | Link to the Navigation API endpoint for the `Resource`. |
| `document` | URI Template | Y (if `@type` is `Resource`) | Link to the Document API endpoint for the `Resource`. |
| `download` | URI or array | N | A link or a key: value list of media type: link to downloadable versions of the `Resource` |
| `citeStructure` | array | N | A list of Citation Structures, outlining the types of citation in the `Resource`s citation tree. |

Additional properties for `Resource` objects:

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `mediaTypes` | array | N | An array of string identifiers for the response body media types (Content-Type values) supported for the `Resource` in Document endpoint queries. |

#### `MetadataObject` Structure

In order to make metadata parsable across implementations of the APIs, we restrict the depth of properties in JSON-LD metadata objects.

`MetadataObject` is an object whose properties **MUST** be defined in the `@context`. [Dublin Core Terms](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/) are already provided by the base `@vocab` provided by DTS.

The values of this object's properties **MAY** be:

- a literal that cannot be localized (such as `int` or `float`),
- a URI,
- an array of objects with `@value` and `@lang` properties,
- an array of URIs.

### URI for Collection Endpoint Request

#### Query Parameters

The collection endpoint supports the following query parameters:

| Name | Type | Description | Methods | Constraints |
|------|----- | ----------- | ------- | ----------- |
| id | URI | Identifier for a collection or document. | GET | |
| page| int | Page of the current collection's members | GET | |
| nav | string | Determines whether the content of the `member` property represents parent or child items. | GET | `children` (default) or `parents` |

### Examples

#### Root collection

This is an example of a top-level Collection that groups texts into 3 categories.

##### Example Request

- `/api/dts/collection/`
- `/api/dts/collection/?id=general`

##### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "@id": "general",
    "@type": "Collection",
    "collection": "/api/dts/collection/{?id,page,nav}",
    "dtsVersion": "1-alpha",
    "totalParents": 0,
    "totalChildren": 2,
    "title": "Collection Générale de l'École Nationale des Chartes",
    "dublinCore": {
        "publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "title": [
            {"lang": "fr", "value": "Collection Générale de l'École Nationale des Chartes"}
        ]
    },
    "member": [
        {
             "@id" : "cartulaires",
             "title" : "Cartulaires",
             "description": "Collection de cartulaires d'Île-de-France et de ses environs",
             "@type" : "Collection",
             "collection": "/api/dts/collection/?id=cartulaires{&page,nav}",
             "totalParents": 1,
             "totalChildren": 10
        },
        {
             "@id" : "lasciva_roma",
             "title" : "Lasciva Roma",
             "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
             "@type" : "Collection",
             "collection": "/api/dts/collection/?id=lasciva_roma{&page,nav}",
             "totalParents": 1,
             "totalChildren": 1
        },
        {
             "@id" : "lettres_de_poilus",
             "title" : "Correspondance des poilus",
             "description": "Collection de lettres de poilus entre 1917 et 1918",
             "@type" : "Collection",
             "collection": "/api/dts/collection/?id=lettres_de_poilus{&page,nav}",
             "totalParents": 1,
             "totalChildren": 10000
        }
    ]
}
```


#### Child collection containing a single work

The example is a child of the parent root collection. It contains a single textual work as a member collection.

##### Example Request

- `/api/dts/collection/?id=lasciva_roma`

##### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "dtsVersion": "1-alpha",
    "@id": "lasciva_roma",
    "@type": "Collection",
    "collection": "/api/dts/collection/{?id,page,nav}",
    "totalParents": 1,
    "totalChildren": 3,
    "title" : "Lasciva Roma",
    "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
    "dublinCore": {
        "creator": [
            "Thibault Clérice", "http://orcid.org/0000-0003-1852-9204"
        ],
        "title" : [
            {"lang": "la", "value": "Lasciva Roma"},
        ],
        "description": [
            {
                "lang": "en",
                "value": "Collection of primary sources of interest in the studies of Ancient World's sexuality"
            }
        ],
    },
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001",
            "title" : "Priapeia",
            "dublinCore": {
                "type": [
                    "http://chs.harvard.edu/xmlns/cts#work"
                ],
                "creator": [
                    {"lang": "en", "value": "Anonymous"}
                ],
                "language": ["la", "en"],
                "description": [
                    { "lang": "en", "value": "Anonymous lascivious Poems" }
                ],
            },
            "@type" : "Collection", 
             "collection": "/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001{&page,nav}",
            "totalParents": 1,
            "totalChildren": 1
        }
    ]
}
```

#### Child collection representing a single work

The example is a child collection. It represent a single textual work and its members are textual Resources that are individual expressions of that work. These Resources are therefore Readable Collections.

##### Note

Although, this is optional, the expansion of `@type:Resource`'s metadata is advised to avoid multiple API calls.

##### Example Request

- `/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001`

##### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "dtsVersion": "1-alpha",
    "@id": "urn:cts:latinLit:phi1103.phi001",
    "@type": "Collection",
    "collection": "/api/dts/collection/{?id,page,nav}",
    "title" : "Priapeia",
    "dublinCore": {
        "type": ["http://chs.harvard.edu/xmlns/cts#work"],
        "creator": [
            {"lang": "en", "value": "Anonymous"}
        ],
        "language": ["la", "en"],
        "title": [{"lang": "la", "value": "Priapeia"}],
        "description": [{
           "lang": "en",
            "value": "Anonymous lascivious Poems "
        }]
    },
    "totalParents": 1,
    "totalChildren": 1,
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "@type": "Resource",
            "title" : "Priapeia",
            "description": "Priapeia based on the edition of Aemilius Baehrens",
            "totalParents": 1,
            "totalChildren": 0,
            "dublinCore": {
                "title": [{"lang": "la", "value": "Priapeia"}],
                "description": [{
                   "lang": "en",
                   "value": "Anonymous lascivious Poems "
                }],
                "type": [
                    "http://chs.harvard.edu/xmlns/cts#edition",
                    "Text"
                ],
                "source": ["https://archive.org/details/poetaelatinimino12baeh2"],
                "dateCopyrighted": 1879,
                "creator": [
                    {"lang": "en", "value": "Anonymous"}
                ],
                "contributor": ["Aemilius Baehrens"],
                "language": ["la", "en"]
            },
            "collection": "/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1(&nav}",
            "document": "/api/dts/document?resource=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1{&ref,start,end,tree,mediaType}",
            "navigation": "/api/dts/navigation?resource=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1{&ref,start,end,tree}",
            "download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
            "citationTrees": [
                {
                  "@type": "CitationTree",
                  "maxCiteDepth" : 2,
                  "citeStructure" : [
                    {
                      "citeType": "poem",
                      "citeStructure": [
                          {
                              "citeType": "line"
                          }
                      ]
                    }
                  ]
                }
            ]
        }
    ]
}
```

#### Child Readable Collection (i.e. a textual Resource)

This example is a child Readable Collection, i.e. a textual Resource which is composed of passages of readable text. The response includes fields which identify the urls for the other 2 DTS api endpoints for further exploration of this Collection: references for retrieval of passage references and passage for retrieval of the entire collection of text passages (i.e the full document itself).

##### Example Request

- `/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1`

##### Response Example 1

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "dtsVersion": "1-alpha",
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalParents": 1,
    "totalChildren": 0,
    "dublinCore": {
        "title": [{"lang": "la", "value": "Priapeia"}],
        "description": [{
           "lang": "en",
            "value": "Anonymous lascivious Poems "
        }],
        "type": [
            "http://chs.harvard.edu/xmlns/cts#edition",
            "Text"
        ],
        "source": ["https://archive.org/details/poetaelatinimino12baeh2"],
        "dateCopyrighted": 1879,
        "creator": [
            {"lang": "en", "value": "Anonymous"}
        ],
        "contributor": ["Aemilius Baehrens"],
        "language": ["la", "en"]
    },
    "collection": "/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1(&nav}",
    "document": "/api/dts/document?resource=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1{&ref,start,end,tree,mediaType}",
    "navigation": "/api/dts/navigation?resource=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1{&ref,start,end,tree}",
    "download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth" : 2,
        "citeStructure" : [
          {
            "citeType": "poem",
            "citeStructure": [
                {
                    "citeType": "line"
                }
            ]
          }
        ]
      }
    ]
}
```

##### Response Example 2

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "dtsVersion": "1-alpha",
    "@id": "https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "@type" : "Resource",
    "title" : "Bucolica",
    "totalParents": 1,
    "totalChildren": 0,
    "collection": "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica(&nav}",
    "document": "/api/dts/document?resource=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica{&ref,start,end,tree,mediaType}",
    "navigation": "/api/dts/navigation?resource=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica{&ref,start,end,tree}",
    "download": "https://github.com/sjhuskey/Calpurnius_Siculus/blob/master/editio.xml",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth" : 2,
        "citeStructure" : [
          {
            "citeType": "front_matter"
          },
          {
            "citeType": "poem",
            "citeStructure": [
                {
                    "citeType": "line"
                }
            ]
          }
        ]
      }
    ]
}
```


#### Paginated Child Collection

This is an example of a paginated request for a Child Collection's members.

##### Example Request

- `/api/dts/collection/?id=lettres_de_poilus&page=19`

##### Example Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "dtsVersion": "1-alpha",
    "@id" : "lettres_de_poilus",
    "@type" : "Collection",
    "collection": "/api/dts/collection/{?id,page,nav}",
    "totalParents": 1,
    "totalChildren": 10000,
    "title": "Lettres de Poilus",
    "dublinCore": {
        "publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "title": [
            {"lang": "fr", "value" : "Lettres de Poilus"}
        ]
    },
    "member": [
      "..."
    ],
    "view": {
        "@id": "/api/dts/collection/?id=lettres_de_poilus&page=19",
        "@type": "PartialCollectionView",
        "first": "/api/dts/collection/?id=lettres_de_poilus&page=1",
        "previous": "/api/dts/collection/?id=lettres_de_poilus&page=18",
        "next": "/api/dts/collection/?id=lettres_de_poilus&page=20",
        "last": "/api/dts/collection/?id=lettres_de_poilus&page=500"
    }
}
```

#### Parent Collection Query

This is an example of a query for the parents of a Collection.

##### Example Request

The example comes from Papyri.info and concerns a document that has been published in three different corpora. The record `https://papyri.info/ddbdp/p.louvre;1;4` thus belongs to the BGU collection, the Chrest.Wilck. collection, and the P.Louvre collection.

- `/api/dts/collection/?id=https://papyri.info/ddbdp/p.louvre;1;4&nav=parents`

##### Example Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "dtsVersion": "1-alpha",
    "@id": "https://papyri.info/ddbdp/p.louvre;1;4",
    "@type" : "Resource",
    "title" : "Housekeeping Book from Temple at Soknopaios (with Festive Calendar)",
    "totalParents": 3,
    "totalChildren": 0,
    "dublinCore": {
        "title": [{"lang": "de", "value": "Haushaltsbuch des Soknopaios  - Heiligtums (mit Festkalender)"}],
        "type": [
            "http://lawd.info/ontology/WrittenWork"
        ],
        "language": ["grc", "de"]
    },
    "collection": "/api/dts/collection/?id=https://papyri.info/ddbdp/p.louvre;1;4(&nav}",
    "document": "/api/dts/document?resource=https://papyri.info/ddbdp/p.louvre;1;4{?ref,start,end,tree,mediaType}",
    "navigation": "/api/dts/navigation?id=https://papyri.info/ddbdp/p.louvre;1;4{?ref,start,end,tree}",
    "download": "https://papyri.info/ddbdp/p.louvre;1;4/source",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth" : 2,
        "citeStructure" : [
          {
            "citeType": "column",
            "citeStructure": [
                {
                    "citeType": "line"
                }
            ]
          }
        ]
      }
    ],
    "member": [
        {
            "@id" : "https://papyri.info/ddbdp/bgu;1",
            "title" : "BGU 1",
            "dublinCore": {
                "type": [
                    "http://lawd.info/ontology/WrittenWork"
                ],
                "language": ["grc", "de"],
            },
            "@type" : "Collection",
            "collection": "/api/dts/collection/?id=https://papyri.info/ddbdp/bgu;1{&page,nav}",
            "totalParents": 1,
            "totalChildren": 299,
        },
        {
            "@id" : "https://papyri.info/ddbdp/chr.wilck",
            "title" : "Chrest.Wilck. 92",
            "dublinCore": {
                "type": [
                    "http://lawd.info/ontology/WrittenWork"
                ],
                "language": ["grc", "de"],
            },
            "@type" : "Collection",
            "collection": "/api/dts/collection/?id=https://papyri.info/ddbdp/chr.wilck{&page,nav}",
            "totalParents": 1,
            "totalChildren": 182,
        },
        {
            "@id" : "https://papyri.info/ddbdp/p.louvre1",
            "title" : "P.Louvre 1",
            "dublinCore": {
                "type": [
                    "http://lawd.info/ontology/WrittenWork"
                ],
                "language": ["grc", "de"],
            },
            "@type" : "Collection",
            "collection": "/api/dts/collection/?id=https://papyri.info/ddbdp/p.louvre1{&page,nav}",
            "totalParents": 1,
            "totalChildren": 93,
        }
    ]
}
```

## Navigation Endpoint

The Navigation endpoint provides information about a Resource's internal structures (e.g., book, chapter, line, etc.) and how they are referenced. While the Collection endpoint allows for finding Resources within a collection, the Navigation endpoint describes the relationship between structural divisions in the text. References found in the Navigation endpoint can also be used to retrieve corresponding portions of the Resource's text from the Document endpoint.

The Navigation endpoint provides JSON objects that describe sections in a Resource's citation tree. A citation tree is represented by a structured hierarchy of `CitableUnit` objects. In general, applications using the Navigation endpoint are moving down and forward through the document.

Using the Navigation endpoint, clients will be able to:

- construct a table of contents
- display a novel by chapter or a book of poetry by poem
- find out how to traverse a Resource from one structural section to another
- construct links to specific sections of the Resource
- find metadata about a specific section of a Resource (e.g., the writer of one letter in a correspondence)

### Scheme for Navigation Endpoint Responses

#### `Navigation` Object

The top-level response object is a `Navigation` object answering a query about the citation tree of a Resource, containing details about nodes within that tree.

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `@id` | URL | Y | The absolute URL of the current request including any query parameters. |
| `@type` | string | Y | The object's RDF class which must be "Navigation". |
| `dtsVersion` | string | Y | The version of the DTS specification providing the response. Default is "1-alpha". |
| `document` | URI template | Y | The URI template to the Document endpoint at which the text of nodes in the citation tree can be retrieved. |
| `navigation` | URI template | Y | The URI template to the Navigation endpoint at which the citation tree structure can be queried. |
| `resource`| Resource | Y | The `Resource` whose citation tree is being queried. |
| `ref` | CitableUnit | N | The `CitableUnit` in the citation tree which is being queried. |
| `start` | CitableUnit | N | The `CitableUnit` at the beginning of the range in the citation tree which is being queried. |
| `end` | CitableUnit | N |  The `CitableUnit` at the end of the range in the citation tree which is being queried. |
| `member` | array | N | An array of `CitableUnit` in the subtree specified by the query parameters. |

Because the `Navigation` object is a top-level object in the API, each object must also have a `@context` property pointing to a DTS JSON-LD context object such as "https://distributed-text-services.github.io/specifications/context/1-alpha1.json".

#### Resource

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `@id` | URI | Y | The URI of the `Resource`. |
| `@type` | string | Y | The object's RDF class which must be "Resource". |
| `collection` | URL template | Y | The URI template to the Collection endpoint at which the `Resource` can be found. |
| `citationTrees` | array | Y | An array of `CitationTree` objects |
| `mediaTypes` | array | N | An array of string identifiers for the response body media types (Content-Type values) supported for the `Resource` in Document endpoint queries. |

If a `Resource` has a single `CitationTree`, that `CitationTree` object cannot have an `identifier`.

If a `Resource` has multiple `CitationTree`s, then the first listed in `citationTrees` is the default `CitationTree` and cannot have an `identifier`.

If a `Resource` is a document without a citation tree, the `citationTrees` property is an empty array.

Values in the `mediaTypes` array must correspond to content types that the implementation supports as values for the `mediaType` query parameter when requesting segments of the `Resource`'s content via the Document endpoint.

#### CitationTree

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `identifier` | string | N | The string identifier of the `CitationTree`. |
| `@type` | string | Y | The object's RDF class which must be "CitationTree". |
| `citeStructure` | array | N | An array of `CiteStructure` objects. |
| `maxCiteDepth` | int | Y | An integer defining the maximum depth of the Resource's citation tree. |
| `description` | string | N | A human readable description of the citation tree. |

#### CiteStructure

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `citeStructure` | array | N | An array of `CiteStructure` objects. |
| `citeType` | string | N | A type of textual unit that appears at a given level in the citation tree. (E.g., "chapter", "verse") |

#### CitableUnit

| Name |  Type  |  Required | Description                              |
| ---- | ------ | --------- | -----------------------------|
| `identifier` | string | Y | The string identifier of the `CitableUnit`. |
| `@type` | string | Y | The object's RDF class which must be "CitableUnit". |
| `level` | int | Y | A number identifying the depth at which the `CitableUnit` is found within the citation tree of the `Resource`. |
| `parent` | nullable string | Y | The string identifier of the hierarchical parent of the `CitableUnit` in the `Resource`. |
| `citeType` | string | N | The type of textual unit corresponding to the `CitableUnit` in the Resource. (E.g., "chapter", "verse") |
| `dublinCore` | [MetadataObject](#metadataobject-structure) | N | Dublin Core Terms metadata describing the `CitableUnit`. |
| `extensions` | [MetadataObject](#metadataobject-structure) | N | Metadata for the `CitableUnit` from vocabularies other than Dublin Core Terms. |

##### Unique `identifier`s

The `identifier` of a `CitableUnit` must be unique within a `CitationTree` for a `Resource`.

##### Values for the `parent` Property

If the `CitableUnit` parent is the root level of the `Resource`, the value returned for `parent` should be `null`.

```json
{
  "identifier": "Luke",
  "level": 1,
  "@type": "CitableUnit",
  "parent": null
}
```

### URI for Navigation Endpoint Requests

#### Query Parameters

| Name | Type | Description                              | Methods | Constraints |
|------|------ | ------------------------------------|---------| ----------- |
| resource   | URI | The unique identifier for the Resource being navigated |  GET    ||
| ref | string | The string identifier of a single node in the citation tree for the Resource, used as the point of reference for the query. | GET    | NOT used with `start` and `end` |
| start | string | The string identifier of a node in the citation tree for the resource, used as the starting point for a range that serves as the reference point for the query. This parameter is inclusive, so the starting point is considered part of the specified range. | GET | NOT used if a `ref` is specified, requires `end` as well |
| end |  string | The string identifier of a node in the citation tree for the resource, used as the ending point for a range of passages that serves as the reference point for the query. This parameter is inclusive, so the supplied ending point is considered part of the specified range. | GET | NOT used if a `ref` is specified, requires `start` as well |
| down | int | The maximum depth of the citation subtree to be returned, relative to the specified `ref`, the deeper of the `start`/`end` `CitableUnit`, or if these are not provided relative to the root. A value of `-1` indicates the bottom of the `Resource` citation tree. | GET    |If `down` is not provided only retrieve information about the queried `CitableUnit` |
| tree | string | The string identifier for a `CitationTree` of the `Resource`. | GET | NOT used to query the default `CitationTree` |
| page | int | The number of identifying a page in paginated query results. | GET | |

##### Errors

Some combinations of query parameters and their values must return a 4XX HTTP Error. A 400 Bad Request Error should be returned when:
  - no `resource` value is provided
  - both `ref` and either `start` or `end` is specified
  - `start` is provided without also specifying `end`, or vice versa
A 404 Not Found Error should be returned when
  - a query specifies a `ref`, `start`, or `end` value that does not exist in the queried `CitationTree`
  - a query specifies a `tree` value that does not correspond to an existing `CitationTree` for the `Resource`

##### Usage of `tree`

All requests on a `Resource` without any `CitationTree`s at the `Navigation` endpoint will return an empty `member` array and must not return an HTTP error.

If no `tree` parameter is specified, the default `CitationTree` of the `Resource` must be used to traverse the `Resource`.

##### Usage of `down`, `ref`, `start` and `end`

| `down` | `ref` | `start`/`end` | Result |
| ------ | ----- | ------------- | ------ |
| absent | absent | absent | 400 Bad Request Error |
| absent | present | absent | Information about the `CitableUnit` identified by `ref`. No member property in the `Navigation` object. |
| absent | absent | present | Information about the `CitableUnit`s identified by `start` and by `end`. No member property in the `Navigation` object. |
| 0 | present | absent | Information about the `CitableUnit` identified by `ref` along with a `member` property that is an array of `CitableUnit`s that are siblings (sharing the same parent) including the current `CitableUnit` identified by `ref`. |
| 0 | absent | present | 400 Bad Request Error |
| > 0 | absent | absent | A `member` array of `CitableUnit`s including the citation tree from the root to the depth requested in `down`. |
| > 0 | present | absent | A `member` array of `CitableUnit`s including the citation tree from the `CitableUnit` identified by `ref` to the depth requested in `down`. |
| > 0 | absent | present | A `member` array of `CitableUnit`s including the citation tree between the `start` and `end` `CitableUnit`s inclusive, down to a depth relative to the deeper of the `CitableUnit`s identified by `start` and `end`. |
| -1 | absent | absent | A `member` array of `CitableUnit`s including the citation tree from the root to the deepest level of the `CitationTree`. |
| -1 | present | absent | A `member` array of `CitableUnit`s including the citation tree from the `CitableUnit` identified by `ref` to the deepest level of the `CitationTree`. |
| -1 | absent | present | A `member` array of `CitableUnit`s including the citation tree between the `start` and `end` `CitableUnit`s inclusive, down to the deepest level of the `CitationTree`. |

##### Handling Requests with No Matching `CitableUnit`s

A `Navigation` endpoint request may specify a level in a `Resource`'s `CitationTree` that does not exist. One may, e.g., provide a `down` value of `3` when only `1` lower level exists in the `Resource`'s `CitationTree`. In this case the `member` array will simply include any `CitableUnit`s that do satisfy the parameters.

If there are no `CitableUnit`s at all that satisfy the parameters of a `Navigation` endpoint request:

- the request must not raise an error
- the `Navigation` object `member` property must be an empty array.

For example, if the `ref` is at the bottom level of the queried `CitationTree`, and a `down` of 2 is provided in the request, the response will provide an empty array as its `member` value.

##### Order of `CitableUnit`s in `member`

The `CitableUnits` listed in the returned `member` property must be in document order as defined in the XPath 3.1 specification: https://www.w3.org/TR/xpath-31/#dt-document-order.

This is a "pre-order, depth first" traversal moving through nodes of the citation tree as follows:

!["pre-order, depth first traversal example"](./assets/img/tree-traversal_example.png)


### Response Headers

Responses from the `Navigation` endpoint should include an HTTP response header identifying the `Content-Type` of the response as `application/ld+json`.

| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |

### Examples

#### Writing Conventions for the Examples

In the following examples, JavaScript comments such as `/* ... */` are used to indicate that more information may be found at this point in the Response body.

Note, too, that the `CitableUnit` objects in most of the examples are kept minimal for the sake of clear illustration. In many actual implementations each of these would include `dublinCore` and `extensions` properties containing additional metadata. The choice of what additional content to include in these properties, though, is up to the implementer.

#### Example 1: Requesting top-level `CitableUnit`s of a `Resource`

The client wants to retrieve an array of `CitableUnit`s that are part of the `Resource` with the identifier "https://en.wikisource.org/wiki/Dracula". In other words, the client wants an array of the top-level nodes in the resource's citation tree. Since no `ref`, `start`, or `end` query parameters are specified, the response object lacks any `ref`, `start` or `end` properties. It simply returns the requested `CitableUnits` as an array in the `member` property of the `Navigation` object.

##### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=1`

##### Response Headers

| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |

##### Response Body

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
  "dtsVersion": "1-alpha",
  "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=1",
  "document": "https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula{&ref,start,end,tree,mediaType}",
  "collection": "https://example.org/api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula{&page,nav}",
  "navigation": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula{&ref,down,start,end,tree,page}",
  "resource": {
    "@id": "https://en.wikisource.org/wiki/Dracula",
    "@type": "Resource",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth": 3,
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
        "title": {
          "lang": "en",
          "value": "Chapter 1: Jonathan Harker's Journal"
        }
      }
    },
    {
      "identifier": "C2",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {
          "lang": "en",
          "value": "Chapter 2: Jonathan Harker's Journal - Continued"
        }
      }
    },
    {
      "identifier": "C3",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {
          "lang": "en",
          "value": "Chapter 3: Jonathan Harker's Journal - Continued"
        }
      }
    }
    /* ... */
  ]
}
```

#### Example 2: Requesting the `Resource`'s complete citation tree down to a specified level

The client wants to retrieve an array of all `CitableUnit`s in the `Resource` identified as "https://en.wikisource.org/wiki/Dracula" down to the second level of the `Resource`'s citation tree (in this case chapters and their paragraphs). We are not requesting the descendants of any single upper-level node (i.e., chapter), so response will provide the entire top two levels of the citation tree. Since no `ref`, `start`, or `end` query parameters are specified, the returned `Navigation` object has no `ref`, `start`, or `end` properties. The requested array of `CitableUnits` are simply returned in the `member` property.

##### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=2`

##### Response Headers

| Key          | Value                             |
| ------------ | --------------------------------- |
| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |

##### Response Body

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "dtsVersion": "1-alpha",
    "@id":"https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=2",
    "document": "https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula{&ref,start,end,tree,mediaType}",
    "collection": "https://example.org/api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula{&page,nav}",
    "navigation": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula{&ref,down,start,end,tree,page}",
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
          "title": {"lang": "en", "value": "Chapter 1: Jonathan Harker's Journal"}
        }
      },
      {
        "identifier": "C1.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry",
        "dublinCore": {
          "title": {"lang": "en", "value": "3 May. Bistritz"}
        }
      },
      {
        "identifier": "C1.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C1",
        "citeType": "Journal Entry",
        "dublinCore": {
          "title": {"lang": "en", "value": "4 May"}
        }
      },
      /* ... */
      {
        "identifier": "C2",
        "@type": "CitableUnit",
        "level": 1,
        "parent": null,
        "citeType": "Chapter",
        "dublinCore": {
          "title": {"lang": "en", "value": "Chapter 2: Jonathan Harker's Journal - Continued"}
        }
      },
      {
        "identifier": "C2.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C2",
        "citeType": "Journal Entry",
        "dublinCore": {
          "title": {"lang": "en", "value": "5 May"}
        }
      },
      {
        "identifier": "C2.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C2",
        "citeType": "Journal Entry",
        "dublinCore": {
          "title": {"lang": "en", "value": "7 May"}
        }
      },
      /* ... */
      {
        "identifier": "C3",
        "@type": "CitableUnit",
        "level": 2,
        "parent": null,
        "citeType": "Chapter",
        "dublinCore": {
          "title": {"lang": "en", "value": "Chapter 3: Jonathan Harker's Journal - Continued"}
        }
      },
      {
        "identifier": "C3.E1",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C3",
        "citeType": "Journal Entry",
        "dublinCore": {
          "title": {"lang": "en", "value": "8 May continued"}
        }
      },
      {
        "identifier": "C3.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C3",
        "citeType": "Journal Entry",
        "dublinCore": {
          "title": {"lang": "en", "value": "Midnight"}
        }
      },
      /* ... */
    ],
}
```

#### Example 3: Requesting all descendants of one top-level `CitableUnit` of a `Resource`

The client wants to retrieve an array of all `CitableUnit`s in the `Resource` identified by "https://en.wikisource.org/wiki/Dracula" that are children of its `Citable Unit` with the identifier "C1". In other words, the client is requesting the entire citation subtree below `CitableUnit` "C1". Here a `ref` query parameter _is_ supplied, so the returned `Navigation` object includes the corresponding `CitableUnit` in its `ref` property. The requested citation subtree is also included in the `member` property.

##### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1&down=-1`

##### Response Headers

| Key          | Value                             |
| ------------ | --------------------------------- |
| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |

##### Response Body

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
  "dtsVersion": "1-alpha",
  "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1&down=-1",
  "document": "https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula{&ref,start,end,tree,mediaType}",
  "collection": "https://example.org/api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula{&page,nav}",
  "navigation": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula{&ref,down,start,end,tree,page}",
  "resource": {
    "@id": "https://en.wikisource.org/wiki/Dracula",
    "@type": "Resource",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth": 3,
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
  "ref": {
    "identifier": "C1",
    "@type": "CitableUnit",
    "level": 1,
    "parent": null,
    "citeType": "Chapter",
    "dublinCore": {
      "title": {
        "lang": "en",
        "value": "Chapter 1: Jonathan Harker's Journal"
      }
    }
  },
  "member": [
    {
      "identifier": "C1",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {
          "lang": "en",
          "value": "Chapter 1: Jonathan Harker's Journal"
        }
      }
    },
    {
      "identifier": "C1.E1",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C1",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": { "lang": "en", "value": "3 May. Bistritz" }
      }
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
    /* ... */
    {
      "identifier": "C1.E2",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C1",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": { "lang": "en", "value": "4 May" }
      }
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
    }
    /* ... */
  ]
}
```

#### Example 4: Requesting descendants of a top-level `CitableUnit` down to grandchildren

The client wants to retrieve the citation subtree below `CitableUnit` "C1" but including only three levels of the tree, "C1" itself and the two levels below it. More precisely, we are retrieving an array of `CitableUnit`s in the citation tree for the `Resource` identified by "https://en.wikisource.org/wiki/Dracula" that are descendants its `CitableUnit` with the identifier "C1", but only down to the second level of the citation tree below "C1".

(Note that since the resource "https://en.wikisource.org/wiki/Dracula" has only three levels to its citation tree, the resulting `member` array will have the same contents as the one returned in Example 3, even though the query parameters differ.)

##### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula?ref=C1&down=2`

##### Headers

| Key          | Value                             |
| ------------ | --------------------------------- |
| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |

##### Response

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
  "dtsVersion": "1-alpha",
  "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula?ref=C1&down=2",
  "document": "https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula{&ref,start,end,tree,mediaType}",
  "collection": "https://example.org/api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula{&page,nav}",
  "navigation": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula{&ref,down,start,end,tree,page}",
  "resource": {
    "@id": "https://en.wikisource.org/wiki/Dracula",
    "@type": "Resource",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth": 3,
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
  "ref": {
    "identifier": "C1",
    "@type": "CitableUnit",
    "level": 1,
    "parent": null,
    "citeType": "Chapter",
    "dublinCore": {
      "title": {
        "lang": "en",
        "value": "Chapter 1: Jonathan Harker's Journal"
      }
    }
  },
  "member": [
    {
      "identifier": "C1",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {
          "lang": "en",
          "value": "Chapter 1: Jonathan Harker's Journal"
        }
      }
    },
    {
      "identifier": "C1.E1",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C1",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "3 May. Bistritz"}
      }
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
    /* ... */
    {
      "identifier": "C1.E2",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C1",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "4 May"}
      }
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
    }
    /* ... */
  ]
}
```

#### Example 5: Requesting children of a lower-level `CitableUnit`

The client wants to retrieve `CitableUnit` "C1.E1" of the `Resource` "https://en.wikisource.org/wiki/Dracula", along with its direct children. We are again retrieving a portion of the citation tree two levels deep, but now beginning from a node at the second level of that tree. The `ref` property of the returned `Navigation` object will contain the node "C1.E1", the reference point for the query. All of the desired `CitableUnit`s are also included as an array in the `member` property.

##### Example of url:

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1.E1&down=1`

##### Headers

| Key          | Value                             |
| ------------ | --------------------------------- |
| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |

##### Response

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
  "dtsVersion": "1-alpha",
  "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1.E1&down=1",
  "document": "https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula{&ref,start,end,tree,mediaType}",
  "collection": "https://example.org/api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula{&page,nav}",
  "navigation": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula{&ref,down,start,end,tree,page}",
  "resource": {
    "@id": "https://en.wikisource.org/wiki/Dracula",
    "@type": "Resource",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth": 3,
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
  "ref": {
      "identifier": "C1.E1",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C1",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "3 May. Bistritz"}
      }
  },
  "member": [
    {
      "identifier": "C1.E1",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C1",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "3 May. Bistritz"}
      }
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
    /* ... */
    {
      "identifier": "C1.E1,P9",
      "@type": "CitableUnit",
      "level": 3,
      "parent": "C1.E1",
      "citeType": "Paragraph"
    },
  ]
}
```

#### Example 6: Requesting descendants of the `CitationUnit`s in a specified range

The client wants to retrieve an array of `CitableUnit`s in a specified range, including the direct children of the specified `start` and `end` points "C1" and "C3". Since the `level` parameter has a value of `1`, the returned `member` array includes two levels of the citation tree for the specified range. Since the `start` and `end` query parameters have been used (rather than `ref`), the returned `Navigation` object will include the nodes at the beginning and end of the range in its `start` and `end` properties. The entire range will be included in the `member` property. This array of `CitableUnit`s will include "C1" through to "C3" at the top level of the citation tree, as well as all of the immediate children for each of those top-level nodes.

##### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=1&start=C1&end=C3`

##### Headers

| Key          | Value                             |
| ------------ | --------------------------------- |
| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |

##### Response

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
  "dtsVersion": "1-alpha",
  "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&down=1&start=C1&end=C3",
  "document": "https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula{&ref,start,end,tree,mediaType}",
  "collection": "https://example.org/api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula{&page,nav}",
  "navigation": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula{&ref,down,start,end,tree,page}",
  "resource": {
    "@id": "https://en.wikisource.org/wiki/Dracula",
    "@type": "Resource",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth": 3,
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
  "start": {
      "identifier": "C1",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {"lang": "en", "value": "Chapter 1: Jonathan Harker's Journal"}
      }
  },
  "end": {
      "identifier": "C3",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {"lang": "en", "value": "Chapter 3: Jonathan Harker's Journal - Continued"}
      }
  },
  "member": [
    {
      "identifier": "C1",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {"lang": "en", "value": "Chapter 1: Jonathan Harker's Journal"}
      }
    },
    {
      "identifier": "C1.E1",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C1",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "3 May. Bistritz"}
      }
    },
    {
      "identifier": "C1.E2",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C1",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "4 May"}
      }
    },
    /* ... */
    {
      "identifier": "C2",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {"lang": "en", "value": "Chapter 2: Jonathan Harker's Journal - Continued"}
      }
    },
    {
      "identifier": "C2.E1",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C2",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "5 May"}
      }
    },
    {
      "identifier": "C2.E2",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C2",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "7 May"}
      }
    },
    /* ... */
    {
      "identifier": "C3",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {"lang": "en", "value": "Chapter 3: Jonathan Harker's Journal - Continued"}
      }
    },
    {
      "identifier": "C3.E1",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C3",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "8 May continued"}
      }
    },
    {
      "identifier": "C3.E2",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C3",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "Midnight"}
      }
    }
    /* ... */
    {
      "identifier": "C3.E6",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C3",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "Later: the Morning of 16 May"}
      }
    }
  ]
}
```

#### Example 7: Discovering the typology of `CitableUnit`s in the `Resource`

Some `CitableUnit`s may have a metadata type expressed in its `citeType` property. If a client wants to know which `citeType`s can be found at each level of the citation tree, they may find this information in the `Resource` object that is included as the `resource` property of each `Navigation` object. Each `Resource` includes a `citationTrees` property which is an array of `CitationTree` objects (often only one). These `CitationTree` objects each have, in turn, a `citeStructure` property containing a tree of the document's hierarchical levels and the types available at each level.

The same `resource` value will be included with every `Navigation` response object for a given `Resource`. So the typology of `CitableUnit`s may be retrieved from any request response. Note that it is *not* permitted to submit a request on the `Resource`'s root node, i.e. to make a request without any of `ref`, `start`, `end`, and `down`. This will result in a 400 Bad Request Error. So if the client is only interested in such metadata about the `Resource` as a whole, they must still select a part of the citation tree as the focus of their query.

Alternately, the same typology of `CitableUnits` and `CiteStructure` may be retrieved from the `Collection` endpoint by querying the same `Resource` there. This is actually the preferred point of access for metadata about the `Resource` as a whole.

##### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1`

##### Response Headers

| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |


```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
  "dtsVersion": "1-alpha",
  "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1",
  "document": "https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula{&ref,start,end,tree,mediaType}",
  "collection": "https://example.org/api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula{&page,nav}",
  "navigation": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula{&ref,down,start,end,tree,page}",
  "resource": {
    "@id": "https://en.wikisource.org/wiki/Dracula",
    "@type": "Resource",
    "citationTrees": [
      {
        "@type": "CitationTree",
        "maxCiteDepth": 3,
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
  "ref": {
    "identifier": "C1",
    "@type": "CitableUnit",
    "level": 1,
    "parent": null,
    "citeType": "Chapter",
    "dublinCore": {
      "title": {
        "lang": "en",
        "value": "Chapter 1: Jonathan Harker's Journal"
      }
    }
  },
  "member": [
    {
      "identifier": "C1",
      "@type": "CitableUnit",
      "level": 1,
      "parent": null,
      "citeType": "Chapter",
      "dublinCore": {
        "title": {
          "lang": "en",
          "value": "Chapter 1: Jonathan Harker's Journal"
        }
      }
    },
  ]
}
```

#### Example 8: Requesting title and other metadata for one `CitableUnit`

If a client wants to retrieve the title and any other metadata for one `CitableUnit`, this may be obtained from the `dublinCore` and `extensions` properties of that `CitableUnit` in the response to any query that includes it. To retrieve one `CitableUnit` on its own, simply specify its identifier as the value for the `ref` query parameter.

It is up to the implementer to decide what optional metadata to provide using the `dublinCore` and `extensions` properties. In this example the `dublinCore` object for "C1.E2" includes values for the Dublin Core Terms `title` and `publisher`. Implementers may include here whatever metadata they like.

##### Example of url :

- `https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C2.E2`

##### Headers

| Key          | Value                             |
| ------------ | --------------------------------- |
| Key          | Value                             |
| ------------ | --------------------------------- |
| Content-Type | Content-Type: application/ld+json |

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1-alpha1.json",
    "dtsVersion": "1-alpha",
    "@id": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula&ref=C2.E2",
    "document": "https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula{&ref,start,end,tree,mediaType}",
    "collection": "https://example.org/api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula{&page,nav}",
    "navigation": "https://example.org/api/dts/navigation/?resource=https://en.wikisource.org/wiki/Dracula{&ref,down,start,end,tree,page}",
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
    "ref": {
      "identifier": "C2.E2",
      "@type": "CitableUnit",
      "level": 2,
      "parent": "C2",
      "citeType": "Journal Entry",
      "dublinCore": {
        "title": {"lang": "en", "value": "7 May"},
        "publisher": ["Modern Library"],
      }
    },
    "member": [
      {
        "identifier": "C2.E2",
        "@type": "CitableUnit",
        "level": 2,
        "parent": "C2",
        "citeType": "Journal Entry",
        "dublinCore": {
          "title": {"lang": "en", "value": "7 May"},
          "publisher": ["Modern Library"],
        }
      },
    ],
}
```

## Document Endpoint

The Document endpoint is used to access the content of document, as opposed to metadata (which is found in collections).

### Query Parameters

The Document endpoint supports the following query parameters:

| Name | Type | Description                              | Methods | Constraints |
|------|------ | ------------------------------------|---------| ----------- |
| resource   | URI | The unique identifier for the `Resource` whose tree or subtree must be returned |  GET    | Required |
| ref | string | The string identifier of a single node in the `CitationTree` for the `Resource`, used as the root for the sub-tree to be reconstructed. | GET    | NOT used with `start` and `end` |
| start | string | The string identifier of a node in the `CitationTree` for the `Resource`, used as the starting point for a range that serves as the reference point for the query. This parameter is inclusive, so the starting point is considered part of the sub-tree to be returned. | GET | NOT used if a `ref` is specified, requires `end` as well |
| end |  string | The string identifier of a node in the `CitationTree` for the `Resource`, used as the ending point for a range that serves as the reference point for the query. This parameter is inclusive, so the supplied ending point is considered part of the specified range. | GET | NOT used if a `ref` is specified, requires `start` as well |
| tree | string | The string identifier for a `CitationTree` of the `Resource`. | GET | NOT used to query the default `CitationTree` |
| mediaType | string | The string identifier for the media-type the resource must be returned in | GET | |

For parameter combinations and potential errors, see [Link](#)

#### Usages

##### Query parameters combinations and requirements

- The `resource` query parameter **MUST** be provided.
- The `tree` query parameter **MUST** be omitted to address the default `CitationTree` of a `Resource`.
- The `tree` query parameter **MUST** be provided to address a `CitableUnit` within a different `CitationTree` from the default.
- The `ref` query parameter **CANNOT** be used in combination with `start` and `end`.
- The `start` query parameter **MUST** be used in combination with `end`.
- The `end` query parameter **MUST** be used in combination with `start`.

#### Errors

Some combination of query parameters and their values **MUST** produce 4xx HTTP Errors:

- A `400 Bad Request` error **SHOULD** be returned when the `resource` parameter is missing.
- A `404 Not Found` error **SHOULD** be returned when any combination of the parameters `resource`, `tree`, `ref`, `start` and `end` does not match a `Resource` and its `CitationTree`.
- A `404 Not Found` error **SHOULD** be returned when the requested value of the `mediaType` parameter is not available for the `Resource` identified by the parameter `resource` in the `Navigation` endpoint. 

#### Formats and limitations of the Endpoint

##### TEI as a major format for the Document endpoint

`Document` endpoint **SHOULD** return textual data in an XML format compliant with the Text Encoding Initiative (TEI) guidelines (`application/tei+xml` response format). The XML **MUST** be well formed and **SHOULD** be valid. XML/TEI **SHOULD** be the default response media-type.

`Document` endpoint **MAY** not implement TEI media-type.

`Document` endpoints **MAY** return requested data in as many other formats as the content provider wishes.

##### XML/TEI encoding and sub-tree representation

The root node of the XML response **MUST** be the `<TEI>` root element of the namespace `http://www.tei-c.org/ns/1.0`. So a response would normally look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <!-- XML of the document requested here -->
</TEI>
```

If the data to be returned is not a complete document, but a chunk using either the `ref` parameter, or the parameters `start`/`end`, the chunk of the document **SHOULD** be embedded in the `<wrapper>` element of the DTS Namespace (`https://w3id.org/api/dts#`) like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:wrapper xmlns:dts="https://w3id.org/api/dts#">
    <!-- XML or string of the passage requested here -->
  </dts:wrapper>
</TEI>
```

There is no limitation as to what can be contained by `<dts:wrapper>` or if its siblings can be provided as long as they are well formed and valid. 

The only limiting factor is that `<dts:wrapper>` **MUST** contain the requested textual data. This permits an implementation to return contextual material elsewhere within the root `TEI` node alongside the requested fragment.

The `<dts:wrapper>` **MAY** provides the attributes `ref`, `start` and `end` that contains `xpath` that identifies the elements identified by the parameters `ref`, `start` and `end`. It is recommended to provide these if the excerpt contains content from other chunks excluded by the sub-tree identified by the parameters `ref`, `start` and `end`.

##### Batch Requests

The `Document` endpoint **does not support** batch requests. 

For the sake of simplicity and usability the DTS guidelines do not allow for batch operations, i.e. fetching or altering more than one passage of text in the same request. Implementations should encourage client software and other consumers of their API to use multiple asynchronous requests instead.

### Response

#### Header

A `Document` endpoint **SHOULD** provide the following entries in their HTTP response header:

- `Link` **SHOULD** contain a URI that links back to the `Collection` endpoint for the requested `Resource`, e.g. as `Link: </api/dts/collection/?id=https://en.wikisource.org/wiki/Dracula; rel="collection"`
- `Content-Type` **SHOULD** contain the media-type of the response, e.g. `Content-Type: application/tei+xml`

### Examples

#### Writing Convention for the Examples

In the examples, HTML comments such as `<!-- ... -->` are used to shorten the amount of information represented, and indicate that more information may be found at this point in the Response body.

#### Example 1: Retrieve a subtree using the `ref` parameter

Retrieve the passage `C1` of the document labeled by the identifier `https://en.wikisource.org/wiki/Dracula`.

##### Request URL

- `https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1`

##### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | <https://example.org//api/dts/collection/?resource=https://en.wikisource.org/wiki/Dracula>; rel="collection" |

##### Successful response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <teiHeader>
      <!-- ... -->
   </teiHeader>
   <text>
      <body>
         <head>DRACULA</head>
         <dts:wrapper xmlns:dts="https://w3id.org/api/dts#">
            <div type="chapter" n="1">
               <head>CHAPTER I</head>
               <head>JONATHAN HARKER'S JOURNAL</head>
               <note>(Kept in shorthand.)</note>
               <div type="entry">
                  <p n="1"><date>3 May</date>. <placeName>Bistritz</placeName>.—Left <placeName>Munich</placeName> at <date>8:35 p. m., on 1st May</date>, arriving at Vienna early next morning; should have arrived at 6:46, but train was an hour late. Buda-Pesth seems a wonderful place, from the glimpse which I got of it from the train and the little I could walk through the streets. I feared to go very far from the station, as we arrived late and would start as near the correct time as possible. The impression I had was that we were leaving the West and entering the East; the most western of splendid bridges over the Danube, which is here of noble width and depth, took us among the traditions of Turkish rule.</p>
                  <p n="2">We left in pretty good time, and came after nightfall to Klausenburgh. Here I stopped for the night at the Hotel Royale. I had for dinner, or rather supper, a chicken done up some way with red pepper, which was very good but thirsty. (Mem., get recipe for Mina.) I asked the waiter, and he said it was called "paprika hendl," and that, as it was a national dish, I should be able to get it anywhere along the Carpathians. I found my smattering of German very useful here; indeed, I don't know how I should be able to get on without it.</p>
                  <!--- ... -->
               </div>
               <div type="entry">
                  <p n="1"><date>4 May</date>. I found that my landlord had got a letter from the Count, directing him to secure the best place on the coach for me; but on making inquiries as to details he seemed somewhat reticent, and pretended that he could not understand my German. This could not be true, because up to then he had understood it perfectly; at least, he answered my questions exactly as if he did. He and his wife, the old lady who had received me, looked at each other in a frightened sort of way. He mumbled out that the money had been sent in a letter, and that was all he knew. When I asked him if he knew Count Dracula, and could tell me anything of his castle, both he and his wife crossed themselves, and, saying that they knew nothing at all, simply refused to speak further. It was so near the time of starting that I had no time to ask any one else, for it was all very mysterious and not by any means comforting.</p>
                  <p n="2">Just before I was leaving, the old lady came up to my room and said in a very hysterical way: </p>
                  <!--- ... -->
               </div>
               <!--- ... -->
            </div>
         </dts:wrapper>
      </body>
   </text>
</TEI>
```

#### Example 2: Retrieve a subtree using `start` and `end`

Retrieve the `CitableUnit` `C1.E1,P1` to the `CitableUnit` `C1.E1,P1` of the `Resource` identified by `https://en.wikisource.org/wiki/Dracula`.

##### Request url

- `https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula&start=C1.E1,P1&end=C1.E1,P2`


##### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | <https://example.org//api/dts/collection/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="collection" |

##### Successful response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <teiHeader>
      <!-- ... -->
   </teiHeader>
   <text>
      <body>
         <head>DRACULA</head>
         <dts:wrapper xmlns:dts="https://w3id.org/api/dts#">
            <div type="chapter" n="1">
               <div type="entry">
                  <p n="1"><date>3 May</date>. <placeName>Bistritz</placeName>.—Left <placeName>Munich</placeName> at <date>8:35 p. m., on 1st May</date>, arriving at Vienna early next morning; should have arrived at 6:46, but train was an hour late. Buda-Pesth seems a wonderful place, from the glimpse which I got of it from the train and the little I could walk through the streets. I feared to go very far from the station, as we arrived late and would start as near the correct time as possible. The impression I had was that we were leaving the West and entering the East; the most western of splendid bridges over the Danube, which is here of noble width and depth, took us among the traditions of Turkish rule.</p>
                  <p n="2">We left in pretty good time, and came after nightfall to Klausenburgh. Here I stopped for the night at the Hotel Royale. I had for dinner, or rather supper, a chicken done up some way with red pepper, which was very good but thirsty. (Mem., get recipe for Mina.) I asked the waiter, and he said it was called "paprika hendl," and that, as it was a national dish, I should be able to get it anywhere along the Carpathians. I found my smattering of German very useful here; indeed, I don't know how I should be able to get on without it.</p>
               </div>
            </div>
         </dts:wrapper>
      </body>
   </text>
</TEI>
```

#### Example 3: Retrieve a full `Resource`'s tree

Retrieve the full content of a `Resource` labeled by the identifier `https://en.wikisource.org/wiki/Dracula`

##### Request URL

- `https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula`

##### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | <https://example.org/api/dts/collection/?id=https://en.wikisource.org/wiki/Dracula>; rel="collection" |

##### Successful response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <teiHeader>
      <!-- ... -->
   </teiHeader>
   <text>
      <body>
         <head>DRACULA</head>
            <div type="chapter" n="1">
               <head>CHAPTER I</head>
               <head>JONATHAN HARKER'S JOURNAL</head>
               <note>(Kept in shorthand.)</note>
               <div type="entry">
                  <p n="1"><date>3 May</date>. <placeName>Bistritz</placeName>.—Left <placeName>Munich</placeName> at <date>8:35 p. m., on 1st May</date>, arriving at Vienna early next morning; should have arrived at 6:46, but train was an hour late. Buda-Pesth seems a wonderful place, from the glimpse which I got of it from the train and the little I could walk through the streets. I feared to go very far from the station, as we arrived late and would start as near the correct time as possible. The impression I had was that we were leaving the West and entering the East; the most western of splendid bridges over the Danube, which is here of noble width and depth, took us among the traditions of Turkish rule.</p>
                  <p n="2">We left in pretty good time, and came after nightfall to Klausenburgh. Here I stopped for the night at the Hotel Royale. I had for dinner, or rather supper, a chicken done up some way with red pepper, which was very good but thirsty. (Mem., get recipe for Mina.) I asked the waiter, and he said it was called "paprika hendl," and that, as it was a national dish, I should be able to get it anywhere along the Carpathians. I found my smattering of German very useful here; indeed, I don't know how I should be able to get on without it.</p>
                  <!--- ... -->
               </div>
               <div type="entry">
                  <p n="1"><date>4 May</date>. I found that my landlord had got a letter from the Count, directing him to secure the best place on the coach for me; but on making inquiries as to details he seemed somewhat reticent, and pretended that he could not understand my German. This could not be true, because up to then he had understood it perfectly; at least, he answered my questions exactly as if he did. He and his wife, the old lady who had received me, looked at each other in a frightened sort of way. He mumbled out that the money had been sent in a letter, and that was all he knew. When I asked him if he knew Count Dracula, and could tell me anything of his castle, both he and his wife crossed themselves, and, saying that they knew nothing at all, simply refused to speak further. It was so near the time of starting that I had no time to ask any one else, for it was all very mysterious and not by any means comforting.</p>
                  <p n="2">Just before I was leaving, the old lady came up to my room and said in a very hysterical way: </p>
                  <!--- ... -->
               </div>
               <!--- ... -->
            </div>
            <div type="chapter" n="2">
               <!--- ... -->
            </div>
            <!--- ... -->
      </body>
   </text>
</TEI>
```

#### Example 4: Retrieve part of a `Resource` as `text/html`

Retrieve the subtree of the `CitableUnit` identified by `C1.E1,P1` of the `Resource` labeled by the identifier `https://en.wikisource.org/wiki/Dracula` as HTML content. 

##### Request URL

- `https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula&mediaType=text/html`

##### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: text/html |
| Link | <https://example.org/api/dts/collection/?id=https://en.wikisource.org/wiki/Dracula>; rel="collection" |

##### Successful response body

```xml
<!DOCTYPE html>
<html>
   <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Dracula, Chapter 1, Entry 1, Paragraph 1</title>
   </head>
   <body>
      <p>
      <i>3 May. Bistritz</i>.—Left Munich at 8:35 <span class="smallcaps" style="font-variant:small-caps;">p. m.</span>, on 1st May, arriving at Vienna early next morning; should have arrived at 6:46, but train was an hour late. Buda-Pesth seems a wonderful place, from the glimpse which I got of it from the train and the little I could walk through the streets. I feared to go very far from the station, as we arrived late and would start as near the correct time as possible. The impression I had was that we were leaving the West and entering the East; the most western of splendid bridges over the Danube, which is here of noble width and depth, took us among the traditions of Turkish rule.
      </p>
   </body>
</html>
```


#### Example 5: Retrieve part of a `Resource` as `text/html` of a non-default `CitationTree`

Retrieve the subtree of the `CitableUnit` identified by `5` of the `Resource` labeled by the identifier `https://en.wikisource.org/wiki/Dracula` using its `CitationTree` identified `pages` as HTML content. 

##### Request URL

- `https://example.org/api/dts/document/?resource=https://en.wikisource.org/wiki/Dracula&mediaType=text/html&tree=pages&ref=5`

##### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: text/html |
| Link | <https://example.org/api/dts/collection/?id=https://en.wikisource.org/wiki/Dracula>; rel="collection" |

##### Successful response body

```xml
<!DOCTYPE html>
<html>
   <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Dracula, Page 5</title>
   </head>
   <body>
      <p>simply refused to speak further. It was so near the time of starting that I had no time to ask any one else, for it was all very mysterious and not by any means comforting.</p>
      <p>Just before I was leaving, the old lady came up to my room and said in a very hysterical way:</p>
      <p>"Must you go? Oh! young Herr, must you go?" She was in such an excited state that she seemed to have lost her grip of what German she knew, and mixed it all up with some other language which I did not know at all. I was just able to follow her by asking many questions. When I told her that I must go at once, and that I was engaged on important business, she asked again:</p>
      <p>"Do you know what day it is?" I answered that it was the fourth of May. She shook her head as she said again:</p>
      <!-- ... -->
   </body>
</html>
```
