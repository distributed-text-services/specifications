# Distributed Text Services API - Collections Endpoint

The collections endpoint is used for navigating collections. A collection contains metadata for the collection itself and an array of members.  Each member is either a collection or the metadata for a document.

DTS does not specify URLs. Clients should discover URLs using navigation and link relations since URLs may differ among implementations.

### Hydra Representation and Hierarchy

DTS does not specify any particular hierarchy of collections. A collection might provide all documents in a flat collection or a collection hierarchy organized by geography, time, or any other convenient logical grouping. 

## Scheme

Everything that is not marked as Optional is mandatory.

JSON wide attributes :

- `@context` must set the default vocabulary to Hydra and provide DCT, TEI and DTS namespace prefixes

Item properties :

- `title` is a single string.   Additional descriptions may be placed in `dts:dublincore` using `dct:title`, e.g. for internationalization.
- `@id` is the identifier of the object (TODO: add language recommending the use of URIs for ids)
- `@type` is either `Collection` or `Resource`
- `totalItems` is the number of children contained by the object
- (Required on Resource) `dts:citeDepth` declare the maximum depth of a readable resource.
- (Optional) `description` is a string that describes the object. Additional descriptions may be placed in `dts:dublincore` using `dc:description`, e.g. for internationalization.
- (Optional) `member` contains members of the collection
- (Optional) `dts:dublincore` contains Dublin Core Terms metadata
- (Optional) `dts:extensions` contains any supplementary information provided by other ontologies/domains
- (Optional) `dts:references` contains links to the Navigation API route for the object (TODO: mandatory in children of `member`?)
- (Optional) `dts:passage` contains a link to the Passage API for the object
- (Optional) `dts:download` contains a link or a list of links to a downloadable format of the object (TODO: decide on link or map of type:URL)
- (Optional) `dts:citeStructure` holds a declared citation tree, see [Sub-collection readable](#sub-collection-readable)

## URI 

### Query Parameters

The collections endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a collection or document. |  GET    |
| page | page of the current collections members |  GET    |
| nav  | whether members of the collection are its `children` (default)  or `parents` | GET |

### URI Template

Here is a template of the URI for Collection API. The route itself (`/dts/api/collection/`) is up to the implementer.

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
  },
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

#### Example of url : 

- `/api/dts/collections/`
- `/api/dts/collections/?id=general`

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
    "@id": "general",
    "@type": "Collection",
    "totalItems": "2",
    "title": "Collection Générale de l'École Nationale des Chartes",
    "dts:dublincore": {
        "dc:publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "dc:title": [
            {"fr" : "Collection Générale de l'École Nationale des Chartes"}
        ]
    },
    "member": [
        {
             "@id" : "cartulaires",
             "title" : "Cartulaires",
             "description": "Collection de cartulaires d'Île-de-France et de ses environs",
             "@type" : "Collection",
             "totalItems" : "10"
        },
        {
             "@id" : "lasciva_roma",
             "title" : "Lasciva Roma",
             "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
             "@type" : "Collection",
             "totalItems" : "1"
        },
        {
             "@id" : "lettres_de_poilus",
             "title" : "Correspondance des poilus",
             "description": "Collection de lettres de poilus entre 1917 et 1918",
             "@type" : "Collection",
             "totalItems" : "10000"
        }
    ]
}
```


### Sub-collection, not readable but small

### Example of url : 

- `/api/dts/collections/?id=lasciva_roma`

### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "tei": "http://www.tei-c.org/ns/1.0"
    },
    "@id": "lasciva_roma",
    "@type": "Collection",
    "totalItems": "2",
    "title" : "Lasciva Roma",
    "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
    "dts:dublincore": {
        "dc:creator": [
            "Thibault Clérice", "http://orcid.org/0000-0003-1852-9204"
        ],
        "dc:title" : [
            {"@lang": "la", "@value": "Lasciva Roma"},
        ],
        "dc:description": [
            {
                "@lang": "en",
                "@value": "Collection of primary sources of interest in the studies of Ancient World's sexuality"
            }
        ],
    },
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001",
            "title" : "Priapeia",
            "dts:dublincore": {
                "dc:type": [
                    "http://chs.harvard.edu/xmlns/cts#work"
                ],
                "dc:creator": [
                    {"@lang": "en", "@value": "Anonymous"}
                ],
                "dc:language": ["la", "en"],
                "dc:description": [
                    { "@lang": "en", "@value": "Anonymous lascivious Poems" }
                ],
            },
            "@type" : "Collection",
            "totalItems": 1
        }
    ]
}
```

### Sub-collection, not readable but with readable members

#### Note

Although, this is optional, the expansion of `@type:Resource`'s metadata is advised to avoid multiple API calls. 

#### Example of url : 

- `/api/dts/collections/?id=urn:cts:latinLit:phi1103.phi001`

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
        "tei": "http://www.tei-c.org/ns/1.0",
    },
    "@id": "urn:cts:latinLit:phi1103.phi001",
    "@type": "Collection",
    "title" : "Priapeia",
    "dts:dublincore": {
        "dc:type": ["http://chs.harvard.edu/xmlns/cts#work"],
        "dc:creator": [
            {"@lang": "en", "@value": "Anonymous"}
        ],
        "dc:language": ["la", "en"],
        "dc:title": [{"@lang": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@lang": "en",
            "@value": "Anonymous lascivious Poems "
        }]
    },
    "totalItems" : "1",
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "@type": "Resource",
            "title" : "Priapeia",
            "description": "Priapeia based on the edition of Aemilius Baehrens",
            "totalItems": 0,
            "dts:dublincore": {
                "dc:title": [{"@lang": "la", "@value": "Priapeia"}],
                "dc:description": [{
                   "@lang": "en",
                   "@value": "Anonymous lascivious Poems "
                }],
                "dc:type": [
                    "http://chs.harvard.edu/xmlns/cts#edition",
                    "dc:Text"
                ],
                "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
                "dct:dateCopyrighted": 1879,
                "dc:creator": [
                    {"@lang": "en", "@value": "Anonymous"}
                ],
                "dc:contributor": ["Aemilius Baehrens"],
                "dc:language": ["la", "en"]
            },
            "dts:passage": "/api/dts/documents?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "dts:references": "/api/dts/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "dts:download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
            "dts:citeDepth": 2,
            "dts:citeStructure": [
                {
                    "dts:citeType": "poem",
                    "dts:citeStructure": [
                        {
                            "dts:citeType": "line"
                        }
                    ]
                }
            ]
        }
    ]
}
```

### Sub-collection readable

#### Example of url : 

- `/api/dts/collections/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response Example 1

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dct": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "dc": "http://purl.org/dc/elements/1.1/",
        "tei": "http://www.tei-c.org/ns/1.0",
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalItems": 0,
    "dts:dublincore": {
        "dc:title": [{"@lang": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@lang": "en",
            "@value": "Anonymous lascivious Poems "
        }],
        "dc:type": [
            "http://chs.harvard.edu/xmlns/cts#edition",
            "dc:Text"
        ],
        "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
        "dct:dateCopyrighted": 1879,
        "dc:creator": [
            {"@lang": "en", "@value": "Anonymous"}
        ],
        "dc:contributor": ["Aemilius Baehrens"],
        "dc:language": ["la", "en"]
    },
    "dts:passage": "/api/dts/documents?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "dts:references": "/api/dts/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "dts:download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
    "dts:citeDepth": 2, 
    "dts:citeStructure": [
        {
            "dts:citeType": "poem",
            "dts:citeStructure": [
                {
                    "dts:citeType": "line"
                }
            ]
        }
    ]
}
```

#### Response Example 2

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dct": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "dc": "http://purl.org/dc/elements/1.1/",
        "tei": "http://www.tei-c.org/ns/1.0",
    },
    "@id": "https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "@type" : "Resource",
    "title" : "Bucolica",
    "totalItems": 0,
    "dts:passage": "/api/dts/documents?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "dts:references": "/api/dts/navigation?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "dts:download": "https://github.com/sjhuskey/Calpurnius_Siculus/blob/master/editio.xml",
    "dts:citeDepth": 2, 
    "dts:citeStructure": [
        {
            "dts:citeType": "front"
        },
        {
            "dts:citeType": "poem",
            "dts:citeStructure": [
                {
                    "dts:citeType": "line"
                }
            ]
        }
    ]
}
```


### Paginated sub-collection

#### Example of url : 

- `/api/dts/collections/?id=lettres_de_poilus&page=19`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dct": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "dc": "http://purl.org/dc/elements/1.1/",
        "tei": "http://www.tei-c.org/ns/1.0",
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": "2",
    "title": "Collection Générale de l'École Nationale des Chartes",
    "dts:dublincore": {
        "dc:publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "dc:title": [
            {"fr" : "Collection Générale de l'École Nationale des Chartes"}
        ]
    },
    "@id" : "lettres_de_poilus",
    "@type" : "Collection",
    "totalItems" : "10000",
    "member": ["member 190 up to 200"],
    "view": {
        "@id": "/api/dts/collections/?id=lettres_de_poilus&page=19",
        "@type": "PartialCollectionView",
        "first": "/api/dts/collections/?id=lettres_de_poilus&page=1",
        "previous": "/api/dts/collections/?id=lettres_de_poilus&page=18",
        "next": "/api/dts/collections/?id=lettres_de_poilus&page=20",
        "last": "/api/dts/collections/?id=lettres_de_poilus&page=500"
    }
}
```

### Querying parents

#### Example of url : 

- `/api/dts/collections/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1&nav=parent`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dct": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "dc": "http://purl.org/dc/elements/1.1/",
        "tei": "http://www.tei-c.org/ns/1.0",
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalItems": 0,
    "dts:dublincore": {
        "dc:title": [{"@lang": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@lang": "en",
            "@value": "Anonymous lascivious Poems "
        }],
        "dc:type": [
            "http://chs.harvard.edu/xmlns/cts#edition",
            "dc:Text"
        ],
        "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
        "dct:dateCopyrighted": 1879,
        "dc:creator": [
            {"@lang": "en", "@value": "Anonymous"}
        ],
        "dc:contributor": ["Aemilius Baehrens"],
        "dc:language": ["la", "en"]
    },
    "dts:passage": "/api/dts/documents?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "dts:references": "/api/dts/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "dts:download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
    "dts:citeDepth": 2, 
    "dts:citeStructure": [
        {
            "dts:citeType": "poem",
            "dts:citeStructure": [
                {
                    "dts:citeType": "line"
                }
            ]
        }
    ],
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001",
            "title" : "Priapeia",
            "dts:dublincore": {
                "dc:type": [
                    "http://chs.harvard.edu/xmlns/cts#work"
                ],
                "dc:creator": [
                    {"@lang": "en", "@value": "Anonymous"}
                ],
                "dc:language": ["la", "en"],
                "dc:description": [
                    { "@lang": "en", "@value": "Anonymous lascivious Poems" }
                ],
            },
            "@type" : "Collection",
            "totalItems": 1
        }
    ]
}
```
