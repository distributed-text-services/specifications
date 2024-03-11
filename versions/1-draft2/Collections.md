## Status of this Document

- This Version: 1-draft2

<span style="color : red;">This document reflects the status of the version 1-draft2, check API Documentation for the current correct version.</span>

# Collection Endpoint

The collection endpoint is used for navigating collections. A collection contains metadata for the collection itself and an array of members.  Each member is either a collection or the metadata for a document.

DTS does not specify URLs. Clients should discover URLs using navigation and link relations since URLs may differ among implementations.

### Hydra Representation and Hierarchy

DTS does not specify any particular hierarchy of collections. A collection might provide all documents in a flat collection or a collection hierarchy organized by geography, time, or any other convenient logical grouping.

## Scheme

Everything that is not marked as Optional is mandatory.

JSON wide attributes :

- `@context` must set the default vocabulary to Hydra and provide DCT, TEI and DTS namespace prefixes

Item properties :

- `title` is a single string.   Additional descriptions may be placed in `dublinCore` using `title`, e.g. for internationalization.
- `@id` is the identifier of the object (TODO: add language recommending the use of URIs for ids)
- `@type` is either `Collection` or `Resource`
- `totalItems` - total number of items that you can find in the members property (irrespective of pagination)
- `totalChildren` - total number of members that you will find if you do nav=children
- `totalParents` - total number of members that you will find if you do nav=parents
- (Required on Resource) `maxCiteDepth` declare the maximum depth of a readable resource.
- (Optional) `description` is a string that describes the object. Additional descriptions may be placed in `dublinCore` using `description`, e.g. for internationalization.
- (Optional) `member` contains members of the collection
- (Optional) `dublinCore` contains Dublin Core Terms metadata
- (Optional) `extensions` contains any supplementary information provided by other ontologies/domains
- (Optional) `references` contains links to the Navigation API route for the object (TODO: mandatory in children of `member`?)
- (Optional) `passage` contains a link to the Document API for the object
- (Optional) `download` contains a link or a list of links to a downloadable format of the object (TODO: decide on link or map of type:URL)
- (Optional) `citeStructure` holds a declared citation tree, see [Child Readable Collection](#child-readable-collection) below.

## URI

### Query Parameters

The collection endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a collection or document. |  GET    |
| page | page of the current collection's members |  GET    |
| nav  | whether members of the collection are its `children` (default)  or `parents` | GET |

### URI Template

Here is a template of the URI for Collection API. The route itself (`/dts/api/collection/`) is up to the implementer.

```json
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
  "@type": "IriTemplate",
  "template": "/dts/api/collection/?id={collection_id}&page={page}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "collection_id",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "page",
      "required": false
    }
  ]
}
```

## Examples

### Root collection

This is an example of a top-level Collection that groups texts into 3 categories.

#### Example of url :

- `/api/dts/collection/`
- `/api/dts/collection/?id=general`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "general",
    "@type": "Collection",
    "totalItems": 2,
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
             "totalItems" : 10,
             "totalParents": 1,
             "totalChildren": 10
        },
        {
             "@id" : "lasciva_roma",
             "title" : "Lasciva Roma",
             "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
             "@type" : "Collection",
             "totalItems" : 1,
             "totalParents": 1,
             "totalChildren": 1
        },
        {
             "@id" : "lettres_de_poilus",
             "title" : "Correspondance des poilus",
             "description": "Collection de lettres de poilus entre 1917 et 1918",
             "@type" : "Collection",
             "totalItems" : 10000,
             "totalParents": 1,
             "totalChildren": 10000
        }
    ]
}
```


### Child collection containing a single work

The example is a child of the parent root collection. It contains a single textual work as a member collection.

#### Example of url :

- `/api/dts/collection/?id=lasciva_roma`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "lasciva_roma",
    "@type": "Collection",
    "totalItems": 3,
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
            "totalItems": 1,
            "totalParents": 1,
            "totalChildren": 1
        }
    ]
}
```

### Child collection representing a single work

The example is a child collection. It represent a single textual work and its members are textual Resources that are individual expressions of that work. These Resources are therefore Readable Collections.


#### Note

Although, this is optional, the expansion of `@type:Resource`'s metadata is advised to avoid multiple API calls.

#### Example of url :

- `/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "urn:cts:latinLit:phi1103.phi001",
    "@type": "Collection",
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
    "totalItems" : 1,
    "totalParents": 1,
    "totalChildren": 1,
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "@type": "Resource",
            "title" : "Priapeia",
            "description": "Priapeia based on the edition of Aemilius Baehrens",
            "totalItems": 0,
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
            "passage": "/api/dts/document?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "references": "/api/dts/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
            "maxCiteDepth": 2,
            "citeStructure": [
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

### Child Readable Collection (i.e. a textual Resource)
This example is a child Readable Collection, i.e. a textual Resource which is composed of passages of readable text. The response includes fields which identify the urls for the other 2 DTS api endpoints for further exploration of this Collection: references for retrieval of passage references and passage for retrieval of the entire collection of text passages (i.e the full document itself).

#### Example of url :

- `/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response Example 1

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalItems": 0,
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
    "passage": "/api/dts/document?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "references": "/api/dts/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
    "maxCiteDepth": 2,
    "citeStructure": [
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
```

#### Response Example 2

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "@type" : "Resource",
    "title" : "Bucolica",
    "totalItems": 0,
    "totalParents": 1,
    "totalChildren": 0,
    "passage": "/api/dts/document?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "references": "/api/dts/navigation?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "download": "https://github.com/sjhuskey/Calpurnius_Siculus/blob/master/editio.xml",
    "maxCiteDepth": 2,
    "citeStructure": [
        {
            "citeType": "front"
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
```


### Paginated Child Collection

This is an example of a paginated request for a Child Collection's members.

#### Example of url :

- `/api/dts/collection/?id=lettres_de_poilus&page=19`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id" : "lettres_de_poilus",
    "@type" : "Collection",
    "totalItems" : 10000,
    "totalParents": 1,
    "totalChildren": 10000,
    "title": "Lettres de Poilus",
    "dublinCore": {
        "publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "title": [
            {"lang": "fr", "value" : "Lettres de Poilus"}
        ]
    },
    "member": ["member 190 up to 200"],
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

### Parent Collection Query

This is an example of a query for the parents of a Collection. Note that, in this context, `totalItems` == `totalParents`

#### Example of url :

- `/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1&nav=parents`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalItems": 1,
    "totalParents": 1,
    "totalChildren": 0,
    "dublinCore": {
        "title": [{"lang": "la", "value": "Priapeia"}],
        "description": [{
           "lang": "en",
            "value": "Anonymous lascivious Poems "
        }],
        "type": [
            "http://chs.harvard.edu/xmlns/cts#edition"
        ],
        "source": ["https://archive.org/details/poetaelatinimino12baeh2"],
        "dateCopyrighted": 1879,
        "creator": [
            {"lang": "en", "value": "Anonymous"}
        ],
        "contributor": ["Aemilius Baehrens"],
        "language": ["la", "en"]
    },
    "passage": "/api/dts/document?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "references": "/api/dts/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
    "maxCiteDepth": 2,
    "citeStructure": [
        {
            "citeType": "poem",
            "citeStructure": [
                {
                    "citeType": "line"
                }
            ]
        }
    ],
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
            "totalItems": 1,
            "totalParents": 1,
            "totalChildren": 1,
        }
    ]
}
```
