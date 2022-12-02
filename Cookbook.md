# Cookbook

## What is the cookbook ?

The cookbook is a set of recipes to help you deal with common questions about how you might want to express some things using DTS. These are not *enforced* standard practices but recommendations.

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

