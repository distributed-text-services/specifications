# Collection Endpoint

The collection endpoint is used for navigating collections. A collection contains metadata for the collection itself and an array of members.  Each member is either a collection or the metadata for a Resource.

DTS does not specify any particular hierarchy of collections. A collection might provide all `Resource`s in a flat collection or a collection hierarchy organized by geography, time, or any other convenient logical grouping. The same Collection or Resource may exist in more than one collection.

## Scheme for Collection API Responses

Everything that is not marked as Optional is mandatory.

JSON wide attributes :

- `@context` provides DCT, TEI and DTS namespace prefixes

Item properties :

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `title` | string | Y | A name for the Collection or Resource. Additional names may be placed in `dublinCore` using `title`, e.g. for internationalization. |
| `@id` | URI | Y | The identifier of the `Collection` or `Resource`. |
| `@type` | `Collection` or `Resource` | Y | The type |
| `totalItems` | int | Y | Total number of parent or child items, depending on the navigation direction specified in the `nav` parameter. |
| `totalChildren` | int | Y | Total number of child `Collection`s or `Resource`s. |
| `totalParents` | int | Y | Total number of parent `Collection`s or `Resource`s. |
| `maxCiteDepth` | int | Y (if `@type` is `Resource`) | The maximum depth of the Citation Tree. |
| `description` | string | N | Short description of the `Collection` or `Resource`. Additional descriptions may be placed in `dublinCore` using `description`, e.g. for internationalization. |
| `member` | array | N | Contains members of the collection. |
| `dublinCore` | object | N | Contains metadata following the Dublin Core Terms scheme. |
| `extensions` | object | N | Contains any supplementary metadata following other schemes. |
| `navigation` | URI Template | Y (if `@type` is `Resource`) | Link to the Navigation API endpoint for the `Resource`. |
| `document` | URI Template | Y (if `@type` is `Resource`) | Link to the Document API endpoint for the `Resource`. |
| `download` | URI or object | N | A link or a key: value list of media type: link to downloadable versions of the `Resource` |
| `citeStructure` | array | N | A list of Citation Structures, outlining the types of citation in the `Resource`s citation tree. |

## URI for Collection Endpoint Request

### Query Parameters

The collection endpoint supports the following query parameters:

| Name | Type | Description | Methods | Constraints |
|------|----- | ----------- | ------- | ----------- |
| id | URI | Identifier for a collection or document. | GET | |
| page| int | Page of the current collection's members | GET | |
| nav | string | Determines whether the content of the `member` property represents parent or child items. | GET | `children` (default) or `parents` |

## Examples

### Root collection

This is an example of a top-level Collection that groups texts into 3 categories.

#### Example Request

- `/api/dts/collection/`
- `/api/dts/collection/?id=general`

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

#### Example Request

- `/api/dts/collection/?id=lasciva_roma`

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

#### Example Request

- `/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001`

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
            "document": "/api/dts/document?resource=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1{&ref,start,end,tree,mediaType}",
            "navigation": "/api/dts/navigation?resource=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1{7ref,start,end,tree}",
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

### Child Readable Collection (i.e. a textual Resource)
This example is a child Readable Collection, i.e. a textual Resource which is composed of passages of readable text. The response includes fields which identify the urls for the other 2 DTS api endpoints for further exploration of this Collection: references for retrieval of passage references and passage for retrieval of the entire collection of text passages (i.e the full document itself).

#### Example Request

- `/api/dts/collection/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1`

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


### Paginated Child Collection

This is an example of a paginated request for a Child Collection's members.

#### Example Request

- `/api/dts/collection/?id=lettres_de_poilus&page=19`

#### Example Response

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

### Parent Collection Query

This is an example of a query for the parents of a Collection. Note that, in this context, `totalItems` == `totalParents`

#### Example Request

The example comes from Papyri.info and concerns a document that has been published in three different corpora. The record `https://papyri.info/ddbdp/p.louvre;1;4` thus belongs to the BGU collection, the Chrest.Wilck. collection, and the P.Louvre collection.

- `/api/dts/collection/?id=https://papyri.info/ddbdp/p.louvre;1;4&nav=parents`

#### Example Response

```json
{
    "@context": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
    "@id": "https://papyri.info/ddbdp/p.louvre;1;4",
    "@type" : "Resource",
    "title" : "Housekeeping Book from Temple at Soknopaios (with Festive Calendar)",
    "totalItems": 3,
    "totalParents": 3,
    "totalChildren": 0,
    "dublinCore": {
        "title": [{"lang": "de", "value": "Haushaltsbuch des Soknopaios  - Heiligtums (mit Festkalender)"}],
        "type": [
            "http://lawd.info/ontology/WrittenWork"
        ],
        "language": ["grc", "de"]
    },
    "document": "/api/dts/document?resource=https://papyri.info/ddbdp/p.louvre;1;4{&ref,start,end,tree,mediaType}",
    "navigation": "/api/dts/navigation?id=https://papyri.info/ddbdp/p.louvre;1;4{&ref,start,end,tree}",
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
            "totalItems": 1,
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
            "totalItems": 1,
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
            "totalItems": 1,
            "totalParents": 1,
            "totalChildren": 93,
        }
    ]
}
```
