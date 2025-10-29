# Cookbook

## What is the cookbook ?

The cookbook is a set of recipes to help you deal with common questions about how you might want to express some things using DTS. These are not *enforced* standard practices but recommendations.

## Handling Versioning of `Resource` in DTS

### Purpose

This recipe explains one natural way to represent versioning in Distributed Text Services (DTS) Resources. Versioning is **completely optional** and should be used only when necessary for your project, because it introduces extra effort and maintenance for the data provider.

We leverage two features:

1. The ability of a `Resource` to have its own members (`member`).  
2. Dublin Core terms: `isVersionOf` (pointing to previous version[s]), `hasVersion` (listing current version) and `identifier` (identifying current version).

### Design Concept

The main idea is:

- The **root Resource** represents the latest version and is described as a standard DTS `Resource`.  
- Older versions are included as `member` resources.  
- Links between the resources are expressed via `hasVersion` and `isVersionOf`.  
- The current version is also marked using the `identifier` property, so clients know which member represents the current or canonical version.

This way, the root resource is always aligned with the current version, while historical versions remain discoverable.

### Example (JSON-LD)

```javascript
{
  "@context": "https://distributed-text-services.github.io/specifications/context/1.0rc1.json",
  "dtsVersion": "1.0rc1",
  "@id": "https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
  "@type": "Resource",
  "title": "Bucolica",
  "totalParents": 1,
  "totalChildren": 2,
  "collection": "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica(&nav}",
  "document": "/api/dts/document?resource=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica{&ref,start,end,tree,mediaType}",
  "navigation": "/api/dts/navigation?resource=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica{&ref,start,end,tree}",
  "dublinCore": {
    "hasVersion": [
      "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_2fe456",
      "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_9a18bc"
    ],
    "identifier": "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_9a18bc"
  },
  "citationTrees": [
    {
      /* ... */
    }
  ],
  "member": [
    {
      "@id": "https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_9a18bc",
      "@type": "Resource",
      "title": "Bucolica",
      "totalParents": 1,
      "totalChildren": 0,
      "collection": "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_9a18bc(&nav}",
      "document": "/api/dts/document?resource=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_9a18bc{&ref,start,end,tree,mediaType}",
      "navigation": "/api/dts/navigation?resource=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_9a18bc{&ref,start,end,tree}",
      "dublinCore": {
        "isVersionOf": [
          "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica"
        ]
      },
      "citationTrees": [
        {
          /* ... */
        }
      ]
    },
    {
      "@id": "https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_2fe456",
      "@type": "Resource",
      "title": "Bucolica",
      "totalParents": 1,
      "totalChildren": 0,
      "collection": "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_2fe456(&nav}",
      "document": "/api/dts/document?resource=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_2fe456{&ref,start,end,tree,mediaType}",
      "navigation": "/api/dts/navigation?resource=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica_2fe456{&ref,start,end,tree}",
      "dublinCore": {
        "isVersionOf": [
          "/api/dts/collection/?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica"
        ]
      },
      "citationTrees": [
        {
          /* ... */
        }
      ]
    }
  ]
}
```

## IIIF

### Link manifests to a catalog entry

To link a manifest to a catalog entry, we recommend using the Dublin Core Terms property "dct:source" with some human readable information on top of the link:

```json
{
    "@context": {
        "@vocab": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
        "sc": "http://iiif.io/api/presentation/2#"
    },
    "@id": "/a/collection/uri",
    "@type" : "Resource",
    "title" : "A title",
    "totalItems": 0,
    "totalParents": 1,
    "totalChildren": 0,
    "passage": "/api/dts/document?id=/a/collection/uri",
    "references": "/api/dts/navigation?id=/a/collection/uri",
    "citeDepth": 2,
    "dublinCore": {
        "source": [
            {
                "@id": "https://a/manifest/uri",
                "@type": "sc:Manifest",
                "title": "A manifest title"
            }
        ]
    }
}
```

### Link manifests to a passage

The operation is really close to the [previous one](#link-manifests-to-a-catalog-entry) as `dublinCore` is also available in the [Navigation endpoint](./navigation-endpoint.html)

```json
{
    "@context": {
        "@vocab": "https://distributed-text-services.github.io/specifications/context/1.0.0draft-2.json",
        "sc": "http://iiif.io/api/presentation/2#"
    },
    "@id":"/api/dts/navigation/?id=/a/text/id&level=2",
    "citeDepth" : 2,
    "level": 2,
    "member": [
      {"ref": "ref 1"},
      {
        "ref": "ref 2",
        "dublinCore": {
            "source": [
                {
                    "@id": "https://a/manifest/uri",
                    "@type": "sc:Manifest",
                    "title": "A manifest title"
                }
            ]
        }
      },
    ],
    "passage": "/dts/api/document/?id=/a/text/id{&ref}{&start}{&end}"
}
```

