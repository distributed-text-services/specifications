# Collections Endpoint

The collections endpoint is used for navigating collections. A collection contains metadata for the collection itself and an array of members.  Each member is either a collection or the metadata for a document.

DTS does not specify URLs. Clients should discover URLs using navigation and link relations since URLs may differ among implementations.

### Hydra Representation and Hierarchy

DTS does not specify any particular hierarchy of collections. A collection might provide all documents in a flat collection or a collection hierarchy organized by geography, time, or any other convenient logical grouping.

## Scheme

Everything that is not marked as Optional is mandatory.

JSON wide attributes :

- `@context` must set the default vocabulary to Hydra and provide DCT, TEI and DTS namespace prefixes

Item properties :

- `title` is a single string.   Additional descriptions may be placed in `dts:dublincore` using `dc:title`, e.g. for internationalization.
- `@id` is the identifier of the object (TODO: add language recommending the use of URIs for ids)
- `@type` is either `Collection` or `Resource`
- `totalItems` is the number of children contained by the object
- (Required on Resource) `dts:citeDepth` declare the maximum depth of a readable resource.
- (Optional) `description` is a string that describes the object. Additional descriptions may be placed in `dts:dublincore` using `dc:description`, e.g. for internationalization.
- (Optional) `member` contains members of the collection
- (Optional) `dts:dublincore` contains Dublin Core Terms metadata
- (Optional) `dts:extensions` contains any supplementary information provided by other ontologies/domains
- (Optional) `dts:references` contains links to the Navigation API route for the object (TODO: mandatory in children of `member`?)
- (Optional) `dts:passage` contains a link to the Documents API for the object
- (Optional) `dts:download` contains a link or a list of links to a downloadable format of the object (TODO: decide on link or map of type:URL)
- (Optional) `dts:citeStructure` holds a declared citation tree, see [Sub-collection readable](#sub-collection-readable)

## URI 

### Query Parameters

The collections endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a collection or document. |  GET    |
| page | page of the current collection's members |  GET    |
| nav  | whether members of the collection are its `children` (default)  or `parents` | GET |

### URI Template

Here is a template of the URI for Collections API. The route itself (`/dts/api/collections/`) is up to the implementer.

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  },
  "@type": "IriTemplate",
  "template": "/dts/api/collections/?id={collection_id}&page={page}",
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
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": 2,
    "title": "Collection Générale de l'École Nationale des Chartes",
    "dts:dublincore": {
        "dc:publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "dc:title": [
            {"@language": "fr", "@value": "Collection Générale de l'École Nationale des Chartes"}
        ]
    },
    "member": [
        {
             "@id" : "cartulaires",
             "title" : "Cartulaires",
             "description": "Collection de cartulaires d'Île-de-France et de ses environs",
             "@type" : "Collection",
             "totalItems" : 10
        },
        {
             "@id" : "lasciva_roma",
             "title" : "Lasciva Roma",
             "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
             "@type" : "Collection",
             "totalItems" : 1
        },
        {
             "@id" : "lettres_de_poilus",
             "title" : "Correspondance des poilus",
             "description": "Collection de lettres de poilus entre 1917 et 1918",
             "@type" : "Collection",
             "totalItems" : 10000
        }
    ]
}
```


### Child collection containing a single work

The example is a child of the parent root collection. It contains a single textual work as a member collection.

#### Example of url : 

- `/api/dts/collections/?id=lasciva_roma`

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
    "@id": "lasciva_roma",
    "@type": "Collection",
    "totalItems": 3,
    "title" : "Lasciva Roma",
    "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
    "dts:dublincore": {
        "dc:creator": [
            "Thibault Clérice", "http://orcid.org/0000-0003-1852-9204"
        ],
        "dc:title" : [
            {"@language": "la", "@value": "Lasciva Roma"},
        ],
        "dc:description": [
            {
                "@language": "en",
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
                    {"@language": "en", "@value": "Anonymous"}
                ],
                "dc:language": ["la", "en"],
                "dc:description": [
                    { "@language": "en", "@value": "Anonymous lascivious Poems" }
                ],
            },
            "@type" : "Collection",
            "totalItems": 1
        }
    ]
}
```

### Child collection representing a single work

The example is a child collection. It represent a single textual work and its members are textual Resources that are individual expressions of that work. These Resources are therefore Readable Collections.


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
    },
    "@id": "urn:cts:latinLit:phi1103.phi001",
    "@type": "Collection",
    "title" : "Priapeia",
    "dts:dublincore": {
        "dc:type": ["http://chs.harvard.edu/xmlns/cts#work"],
        "dc:creator": [
            {"@language": "en", "@value": "Anonymous"}
        ],
        "dc:language": ["la", "en"],
        "dc:title": [{"@language": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@language": "en",
            "@value": "Anonymous lascivious Poems "
        }]
    },
    "totalItems" : 1,
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "@type": "Resource",
            "title" : "Priapeia",
            "description": "Priapeia based on the edition of Aemilius Baehrens",
            "totalItems": 0,
            "dts:dublincore": {
                "dc:title": [{"@language": "la", "@value": "Priapeia"}],
                "dc:description": [{
                   "@language": "en",
                   "@value": "Anonymous lascivious Poems "
                }],
                "dc:type": [
                    "http://chs.harvard.edu/xmlns/cts#edition",
                    "dc:Text"
                ],
                "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
                "dc:dateCopyrighted": 1879,
                "dc:creator": [
                    {"@language": "en", "@value": "Anonymous"}
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

### Child Readable Collection (i.e. a textual Resource) 
This example is a child Readable Collection, i.e. a textual Resource which is composed of passages of readable text. The response includes fields which identify the urls for the other 2 DTS api endpoints for further exploration of this Collection: dts:references for retrieval of passage references and dts:passage for retrieval of the entire collection of text passages (i.e the full document itself).

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
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalItems": 0,
    "dts:dublincore": {
        "dc:title": [{"@language": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@language": "en",
            "@value": "Anonymous lascivious Poems "
        }],
        "dc:type": [
            "http://chs.harvard.edu/xmlns/cts#edition",
            "dc:Text"
        ],
        "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
        "dc:dateCopyrighted": 1879,
        "dc:creator": [
            {"@language": "en", "@value": "Anonymous"}
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
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "dc": "http://purl.org/dc/elements/1.1/"
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


### Paginated Child Collection

This is an example of a paginated request for a Child Collection's members.

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
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": 2,
    "title": "Collection Générale de l'École Nationale des Chartes",
    "dts:dublincore": {
        "dc:publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "dc:title": [
            {"@language": "fr", "@value" : "Collection Générale de l'École Nationale des Chartes"}
        ]
    },
    "@id" : "lettres_de_poilus",
    "@type" : "Collection",
    "totalItems" : 10000,
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

### Parent Collection Query

This is an example of a query for the parents of a Collection.

#### Example of url : 

- `/api/dts/collections/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1&nav=parents`

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
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalItems": 0,
    "dts:dublincore": {
        "dc:title": [{"@language": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@language": "en",
            "@value": "Anonymous lascivious Poems "
        }],
        "dc:type": [
            "http://chs.harvard.edu/xmlns/cts#edition"
        ],
        "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
        "dc:dateCopyrighted": 1879,
        "dc:creator": [
            {"@language": "en", "@value": "Anonymous"}
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
                    {"@language": "en", "@value": "Anonymous"}
                ],
                "dc:language": ["la", "en"],
                "dc:description": [
                    { "@language": "en", "@value": "Anonymous lascivious Poems" }
                ],
            },
            "@type" : "Collection",
            "totalItems": 1
        }
    ]
}
```
