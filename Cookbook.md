# Cookbook

## What is the cookbook ?

The cookbook is a set of recipes to help you deal with common questions about how you might want to express some things using DTS. These are not *enforced* standard practices but recommendations. 

## IIIF

### Link manifests to a catalog entry

To link a manifest to a catalog entry, we recommend using the Dublin Core Terms property "dct:source" with some human readable information on top of the link:

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "sc": "http://iiif.io/api/presentation/2#"
    },
    "@id": "/a/collection/uri",
    "@type" : "Resource",
    "title" : "A title",
    "totalItems": 0,
    "dts:totalParents": 1,
    "dts:totalChildren": 0,
    "dts:passage": "/api/dts/documents?id=/a/collection/uri",
    "dts:references": "/api/dts/navigation?id=/a/collection/uri",
    "dts:citeDepth": 2, 
    "dts:dublincore": {
        "dc:source": [
            {
                "@id": "https://a/manifest/uri",
                "@type": "sc:Manifest",
                "dc:title": "A manifest title"
            }
        ]
    }
}
``` 

### Link manifests to a passage

The operation is really close to the [previous one](#link-manifests-to-a-catalog-entry) as `dts:dublincore` is also available in the [Navigation endpoint](./navigation-endpoint.html)

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "sc": "http://iiif.io/api/presentation/2#"
    },
    "@id":"/api/dts/navigation/?id=/a/text/id&level=2",
    "dts:citeDepth" : 2,
    "dts:level": 2,
    "member": [
      {"dts:ref": "ref 1"},
      {
        "dts:ref": "ref 2",
        "dts:dublincore": {
            "dc:source": [
                {
                    "@id": "https://a/manifest/uri",
                    "@type": "sc:Manifest",
                    "dc:title": "A manifest title"
                }
            ]
        }
      },
    ],
    "dts:passage": "/dts/api/documents/?id=/a/text/id{&ref}{&start}{&end}"
}
``` 

