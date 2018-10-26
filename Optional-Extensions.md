# Optional: Content Creation and Editing Extensions

## About

This page provides optional extensions to the DTS specification. The core specification is focused on consumption (retrieval) of documents and metadata. These extensions provide additional specifications for creation, updating, and deletion of documents and their metadata. An implementation need not include these extensions if it will not be allowing these other kinds of interaction. If an implementation does allow for more than just consumption of data, however, it should do so using the relevant extensions.

These extensions take two forms. First, some expose additional HTTP methods for the core DTS endpoints. Second, these extensions add additional endpoints for kinds of data not included in the core DTS specification.

## Extensions and the Base API Endpoint

If the client does a `GET` on the base API endpoint, the response should include a path for any of the optional endpoints the implementation supports. For example, if all of the optional endpoints are supported, the response would read something like this:

```
{
  "@context": "/dts/api/contexts/EntryPoint.jsonld",
  "@id": "/dts/api/",
  "@type": "EntryPoint",
  "collections": "/dts/api/collections/",
  "documents": "/dts/api/documents/",
  "navigation" : "/dts/api/navigation"
  "docinfo" : "/dts/api/docinfo"
  "media" : "/dts/api/media"
  "annotation" : "/dts/api/annotation"
}
```
(In this example, the path of the base endpoint is `/dts/api`, but a server can choose a different relative URL.)

## Optional Methods for the Collection Endpoint

In addition to retrieving metadata on collections via the GET method, implementations may also support creation and management of collections through the methods POST, PUT, and DELETE.

This documentation assumes that you have read the core specification for the collections endpoint.

### URI

#### Extended Collection Query Parameters

Note that the core `page` and `nav` parameters are not accepted for any of the extended request methods. One additional parameter, `parent`, must be accepted if the `POST` method is supported.

Assuming that the implementation wants to maintain control over who can create and modify server data, the new `token` parameter also allows for token-based authentication (as with OAuth 2.0) if desired. It is up to the implementation to decide how such tokens should be generated and processed.

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a collection or document. Spaces and other problematic characters should be url encoded. |  GET, PUT, DELETE |
| page | page of the current collections members  |  GET    |
| nav  | whether members of the collection are its `children` (default)  or `parents` | GET |
| parent | identifier for the collection under which a new collection or document is to be created. Spaces and other problematic characters in the identifier should be url encoded. | POST |
| token | authentication token for access control | POST, PUT, DELETE |

#### Extended URI Collection Template

The following template is returned at the base collection endpoint, instructing clients how to build valid URL requests. The route itself (`/dts/api/collection/`) is up to the implementer.
<!---TODO: Is there a way of indicating in the template which methods are supported for which parameters?--->

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  },
  "@type": "IriTemplate",
  "template": "/dts/api/collection/?id={collection_id}&page={page}&nav={nav}&parent={parent&token={token}}",
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
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "nav",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "parent",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "token",
      "required": false
    }    
  ]
}
```

### POST on the Collections Endpoint

The `POST` method of the Collection endpoint allows for creation of the metadata record for a new collection or new document within a collection. Only one top-level item (collections and/or documents) may be created in a single `POST` request, but this request may simultaneously create any number of children.

#### POST Query parameters

There are no required query parameters for a `POST` request to the Collection endpoint. The one optional parameter is `parent`. This should be the unique identifier of an existing collection. If a `parent` is supplied, then the new item will be created as a child member of that collection. Otherwise the new item will be created at the top level of the target repository.

The other parameter that may be accepted is `token` which carries the client's authentication token.

#### POST Request Body

The request body must be valid JSON-LD, following the scheme set out in the core specification for the Collection endpoint. The minimum required terms for each item to be created are:

| Term  | Description    |
| ----- |--------------- |
| `title`         | a single string                                |
| `@id`           | the unique identifier of the object            |
| `@type`         | either Collection or Resource                  |
| `totalItems`    | the number of children contained by the object |
| `dts:citeDepth` | the maximum depth of a readable resource (required only when creating a Resource item). |

If children of the new top-level item are to be created, their metadata should be included as a list under the `member` term.

The JSON object must also begin with the `@context` property, setting the default vocabulary to Hydra and providing DCT, TEI and DTS namespace prefixes. This only needs to be included once at the top level of the JSON object. If children are included under `member` the `@context` is not repeated for each child.

#### POST responses

##### Status codes

A successful POST request should return the status code `201(Created)`.

If a POST request is unsuccessful because of problems with the request content (i.e., parameters or request body), then the return status code should be `400(Bad Request)`.

If a POST request is unsuccessful because the specified item `@id` is already in use, then the return status code should be `409(Conflict)`. An implementation must never allow the creation of items with duplicate `@id` values.

##### Successful response content

The response headers after a successful POST request should include a `Location` value. This should be the URL where the newly created record can be retrieved via a GET request. The response should also include a `Content-type` header with a value of "application/ld+json".

The response body after a successful POST request should contain a JSON-LD object representing the newly created metadata. This should be identical to the response a client would get using a GET request to the `Location` indicated in the response header. This will also mean that in most successful requests the JSON-LD contained in the response body will be identical to the object sent by the client in the POST.

##### Unsuccessful response content

If a response returns a status of `400(Bad Request)` then the response body should contain a JSON object providing as much information as possible about which submitted data was missing or unacceptable. In particular, the response body should indicate whether:
- there were missing required query parameters
- any query parameters held unacceptable values
- the request body did not have properly formed JSON-LD
- the request body was properly formed but carried unacceptable values
The description should also include the URL of the documentation for the Collections endpoint.

If a response returns a status of `409(Conflict)` then the response body should contain a JSON object indicating that the requested `@id` value already exists and cannot be created.

<!-- TODO: Has there been much discussion already about error responses? -->


#### POST Example 1: Creating an Empty Top-Level Collection

In this example we will create a new, empty, top-level collection on a server where the Collection endpoint is at `/api/dts/collections`. The newly created collection will be identified by the string "general" and will have the title "Collection Générale de l'École Nationale des Chartes".

##### POST request URL

- `/api/dts/collections/?token=XXXXX`

##### POST request body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": 0,
    "title": "Collection Générale de l'École Nationale des Chartes",
}
```
##### successful POST response status

- 201(Created)

##### successful POST response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/collections?id=general |
| Content-Type  | application/ld+json             |

##### successful POST response body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": 0,
    "title": "Collection Générale de l'École Nationale des Chartes",
}
```


#### POST Example 2: Creating a New Child Document in an Existing Collection

In this example we will create a new Readable Collection (i.e., a textual Resource) inside an existing collection "lasciva_roma". Again, this presumes that the Collection endpoint is found at `/api/dts/collections`.

##### POST request URL

- `/api/dts/collections/?parent=lasciva_roma&token=XXXXX`

##### POST request body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type": "Resource",
    "totalItems": 0,
    "title" : "Priapeia",
    "dts:citeDepth" : 2
}
```

##### successful POST response status

- 201(Created)

##### successful POST response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/collections?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1 |
| Content-Type  | application/ld+json             |

##### successful POST response body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type": "Resource",
    "totalItems": 0,
    "title" : "Priapeia",
    "dts:citeDepth" : 2
}
```


### PUT on the Collections Endpoint
The `PUT` method of the Collection endpoint allows for modification of the metadata for an existing collection or document. Only one item (collection or resource) may be modified in a single `PUT` request.

The request body must be valid JSON-LD. The simplest way to ensure this is by first retrieving a JSON-LD object (using `GET`) that represents the collection or document to be modified. The client may then modify that JSON-LD object and return it in the body of the `PUT` request.

#### PUT Query Parameters

The one required parameter for a `PUT` request is `id`. This should be the unique identifier of the collection or resource that the client wants to modify.

The other parameter that may be accepted is `token` which carries the client's authentication token.

#### PUT Request Body

The request body must be valid JSON-LD, following the scheme set out in the core specification for the Collection endpoint. There are, however, no minimum necessary terms to include in a PUT request (as there are in POST requests). The client may opt to include in the JSON-LD object submitted only those terms that are being modified. Terms left out of the submitted object are not deleted but simply left unchanged on the server. The submitted JSON-LD object still must, however, include the `@context` property, setting the default vocabulary to Hydra and providing DCT, TEI and DTS namespace prefixes.

If a client wishes to delete the value of a term, the client must include that term in the submitted JSON object but assign it an empty string ("") as its value. This API specification does not provide for complete removal of the term itself from a record.

#### PUT Responses

##### Status codes

If the specified item is updated successfully, the response status should be `200(OK)`.

If no item exists with the `@id` specified in the request URL, the response status should be `404(Not Found)`. If there is some other problem with the request data, the response status should be `400(Bad Request)`.

##### Successful response content

The response headers after a successful POST request should include a `Location` value. This should be the URL where the newly created record can be retrieved via a GET request. The response should also include a `Content-type` header with a value of "application/ld+json".

In a successful PUT request, the response body should be a JSON-LD object representing the edited item. This object should always include the `@context` and `@id` properties. Aside from those, however, this object should only include those term/value pairs that were modified in the transaction. This allows the client to quickly recognize whether the correct information was updated on the server.

##### Unsuccessful response content

If a response returns a status of `400(Bad Request)` then the response body should contain a JSON object providing as much information as possible about which submitted data was missing or unacceptable. In particular, the response body should indicate whether:

- there were missing required query parameters
- any query parameters held unacceptable values
- the request body did not have properly formed JSON-LD
- the request body was properly formed but carried unacceptable values

The description should also include the URL of the documentation for the Collections endpoint.

If a response returns a status of `404(Not Found)` then the response body  should contain a JSON object indicating that the requested `@id` value does not exist and so the requested item cannot be modified.

#### PUT Example 1: Editing an Existing Collection

In this example we will use a `PUT` request to modify the metadata for the collection with the `id` value "general." We will shorten the `title` value to read just "Collection Générale".

##### PUT request URL

- `/api/dts/collections/?id=general&token=XXXXX`

##### PUT request body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "title": "Collection Générale",
}
```
##### Successful PUT response status

- `200(OK)`

##### Successful PUT response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/collections?id=general |
| Content-Type  | application/ld+json             |

##### successful PUT response body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "title": "Collection Générale",
}
```

### DELETE on the Collections Endpoint  
The `DELETE` method of the Collection endpoint allows for removal of a collection or resource. Only one item (collection or resource) may be removed in a single `DELETE` request.

<!-- TODO: Should we allow deletion of non-empty collections, i.e. deletion of collection and children simultaneously? -->

#### DELETE Query Parameters

The only query parameter that *must* be accepted for a `DELETE` request is the unique `id` of the item to be removed.

The other parameter that *may* be accepted is `token` which carries the client's authentication token.

#### DELETE Request Body

Delete requests do not include a body, so the request body should be empty.

#### DELETE Responses

##### Status codes

With a `DELETE` request the server should provide different responses based on whether or not the operation has already concluded when the response is sent. If the deletion is done synchronously, and is finished at response time, the returned status code should be `200(OK)`. If the request simply began an asynchronous deletion process, then the response should return `202(Accept)`.

If a response returns a status of `404(Not Found)` then the response body should contain a JSON object indicating that the requested `@id` value does not exist and so the requested item cannot be modified.

##### Successful response content

Unlike with other methods, the response headers after a successful DELETE request should *not* include a `Location` value. The response should, though, include a `Content-type` header with the value "application/ld+json".

In a successful DELETE request, the response body should be a JSON-LD object representing the record that was deleted. This object should always include the `@context` and `@id` properties along with all other terms that carried associated values at the time of deletion.

##### Unsuccessful response content

#### DELETE Example 1: Deleting an Existing Collection

##### DELETE request URL

##### DELETE request body

##### Successful DELETE response status

##### Successful DELETE response headers

##### Successful DELETE response body

## Document Endpoint: Optional Methods

### Extended URI

### POST on the Document Endpoint  

### PUT on the Document Endpoint  

### DELETE on the Document Endpoint  

## Navigation Endpoint: Optional Methods

<!---TODO: Is there any use-case for other methods on Navigation?--->

## Optional Docinfo Endpoint

### Docinfo URI

### GET on the Docinfo Endpoint

### POST on the Docinfo Endpoint  

### PUT on the Docinfo Endpoint  

### DELETE on the Docinfo Endpoint  

## Optional Media Endpoint

### Media URI

### GET on the Media Endpoint

### POST on the Media Endpoint  

### PUT on the Media Endpoint  

### DELETE on the Media Endpoint  

## Optional Annotation Endpoint

### Annotation URI

### GET on the Annotation Endpoint

### POST on the Annotation Endpoint  

### PUT on the Annotation Endpoint  

### DELETE on the Annotation Endpoint  
