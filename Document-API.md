# Distributed Text Services : Document API

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