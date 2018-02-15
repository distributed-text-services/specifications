# Distributed Text Services API

Status: nascent.  Not yet ready for implementation.

## Media Types

The media type for DTS is JSON-LD, using Hydra conventions [http://www.hydra-cg.com](http://www.hydra-cg.com). The `@id` of any resource can be any URI, which may be a URL or a URI. Document content can be in any media type, including TEI, HTML, PDF, syntax trees, or image formats.

## HTTP Status Codes

Standard conventions for HTTP Status Codes are used.  No custom status codes are allowed.

## Entry Point

A client needs to know the URL of the entry point, which is determined by the server. In this example, the entry point is `/dts/api`, but a server can choose a different relative URL. All other resources can be discovered via navigation or queries from the entry point.  URLs shown in payloads in this document may differ from the URLs provided by a given server, and should not be relied on by clients.

If the client does a `GET` on the entry point URL, the following document is returned.

```
{
  "@context": "/dts/api/contexts/EntryPoint.jsonld",
  "@id": "/dts/api/",
  "@type": "EntryPoint",
  "collections": "/dts/api/collections/",
  "documents": "/dts/api/documents/",
  "navigation" : "/dts/api/navigation",
  "search" : "/dts/api/search"  
}
```

Note: This document does not address which services are optional.  That will be determined by the group.

## Collections

The collections entry point is used for navigating collections. A collection contains metadata for the collection itself and an array of members.  Each member is either a collection or the metadata for a document.

The hierarchy of collections is not fixed.  One server might provide all documents in a flat collection or a collection hierarchy organized by geography, time, or any other convenient logical grouping.

### Hydra Representation and Hierarchy

DTS does not specify any particular hierarchy of collections. A collection might provide all documents in a flat collection or a collection hierarchy organized by geography, time, or any other convenient logical grouping.

A server that uses the CTS collection hierarchy might provide the following top level collection:

```
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@id": "urn:perseus:collections",
  "@type": "Collection",
  "totalItems": "2",
  "member": [
    {
      "@id": "urn:perseus:greekLit"
      "title" : "Ancient Greek"
    },
    {
      "@id": "urn:perseus:latinLit",
      "title": "Classical Latin"
    },
    ...
  ]
}]
```


### Query Parameters

The collections endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a collection or document. |  GET    |
| find | keyword / value search on collection or document metadata       |  GET    |
| q    | full-text search on collection or document metadata             |  GET    |

Note: the DTS group needs to decide which of these we need to support and whether some are optional.

Examples:

- `GET /dts/api/collections/?id=urn:perseus:greekLit` returns the metadata for the `urn:perseus:greekLit` collection.
- `GET /dts/api/collections/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5` returns the metadata for `urn:cts:greekLit:tlg0012.tlg001.opp-grc5`, an edition of Homer's Iliad.
- `GET /dts/api/collections/?find=publisher=Teubneri` returns a collection containing the metadata for items where `publisher=Tuebneri`.
- `GET /dts/api/collections/?q=Teubneri` returns a collection containing the metadata for items where some field contains Tuebneri

## Documents

The documents entry point is used to access the data for documents, as opposed to metadata (which is found in collections).  The representation of a document is up to the implementation.

### Query Parameters

The documents endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| q    | full-text search on document data             |  GET    |
| passage | passage identifier (used together with `id` | n/a    |
| collection | used with `q` to specify a collection to be searched | n/a |

Examples:

- `GET /dts/api/documents/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&passage=1.1` returns book 1, line 1 from Homer's Iliad.
- `GET /dts/api/documents/?id=urn:cts:greekLit:tlg0012.tlg001.opp-grc5&q=ἄειδε` returns a collection of units from Homer's Iliad that contain the string `ἄειδε`
- `GET /dts/api/documents/?q=ἄειδε` returns a collection of units from any document that contain the string `ἄειδε`
- `GET /dts/api/documents/?collection=urn:cts:greekLit&q=ἄειδε` returns a collection of units from Greek Literature that contain the string `ἄειδε`

## Navigation

Navigation API replicates functionality of getValidRef in CTS. Its role is to provide a list of passages that are available for a given resource.

### Query Parameters

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| passage | passage identifier (used together with `id`) | GET    |
| level | Depth for passages we want to retrieve identifiers of  | GET    |

### Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |

