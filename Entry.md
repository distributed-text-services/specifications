# General Information

## Vocabulary

- *Collection* : an identifiable aggregation of digital resources which has meaning in a specific research context. Resources may be virtual or real.

- *Readable Collection* : a specific type of *Collection* which represents an aggregation of readable text passages (e.g. a structured text)

- *Reference* : Identifier of a citable node inside a *Readable Collection* (aka text)

## Media Types

The Collection and Navigation endpoints of the DTS API output `application/ld+json` adhering to Hydra conventions [http://www.hydra-cg.com](http://www.hydra-cg.com). The `@id` of any resource can be any URI, which may be a URL or a URI.

The Document endpoint of the DTS API __must__ be able to return `application/tei+xml`.  Implementers of the API may return additional media-types for document content, including, HTML, PDF, syntax trees, or image formats.

## HTTP Status Codes

Standard conventions for HTTP Status Codes are used.  No custom status codes are allowed.  The Document endpoint may report error codes in XML. See the individual endpoint specifications for detailed information.

## Base API Endpoint

A client needs to know the URL of the base DTS API endpoint, which is determined by the server. (In this example, the path of the base endpoint is `/dts/api`, but a server can choose a different relative URL.) 

If the client does a `GET` on the base API endpoint,, the following response is returned:

```
{
  "@context": "/dts/api/contexts/EntryPoint.jsonld",
  "@id": "/dts/api/",
  "@type": "EntryPoint",
  "collections": "/dts/api/collections/",
  "documents": "/dts/api/documents/",
  "navigation" : "/dts/api/navigation"
}
```

This response provides the base path for each of the 3 specified endpoints: collection, navigation and documents. All other resources can be discovered via navigation of individual endpoint responses.  Base paths to individual endpoints URLs shown in response payloads may vary from these base endpoint paths.
