# Endpoint Basics

## Vocabulary

- *Collection* : an identifiable aggregation of digital resources which has meaning in a specific research context. Resources may be virtual or real.

- *Readable Collection* : a specific type of *Collection* which represents an aggregation of readable text passages (e.g. a structured text)

- *Reference* : Identifier of a citable node inside a *Readable Collection* (aka text)

## Media Types

The Collection and Navigation endpoints of the DTS API output `application/ld+json` adhering to Hydra conventions [http://www.hydra-cg.com](http://www.hydra-cg.com). The `@id` of any resource can be any URI, which may be a URL or a URI.

The Document endpoint of the DTS API __must__ be able to return `application/tei+xml`.  Implementers of the API may return additional media-types for document content, including, HTML, PDF, syntax trees, or image formats.

## Depth of secondary properties

In properties such as `dts:extensions` and `dts:dublinCore`, the following rule apply regarding the maximum depth of metadata encoding :

- `collection["dts:extensions"]` and `collection["dts:dublinCore"]` are namespaced dictionaries, working with `collection["@graph"]`
- `collection["extensions or dublinCore"]["prefix:term"]` can be :
   - a literal that cannot be localized (such as int) : `collection["extensions or dublinCore"]["prefix:term"] : 5`
   - a URIRef : `collection["extensions or dublinCore"]["prefix:term"] : "A String"`
   - a list of literal, including localized ones : `collection["extensions or dublinCore"]["prefix:term"] : [{"@value": "Literal", "@lang": "eng"}]`
   - a list of URIRef: `collection["extensions or dublinCore"]["prefix:term"] : ["http://wikidata.org/object/Author"]`

## HTTP Status Codes

Standard conventions for HTTP Status Codes are used.  No custom status codes are allowed.  The Document endpoint may report error codes in XML. See the individual endpoint specifications for detailed information.

## Error messages

Parts of the APIs whose response format is LD+JSON should follow the [Hydra standard for error presentation](https://www.hydra-cg.com/spec/latest/core/#description-of-http-status-codes-and-errors) :

```json
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@type": "Status",
  "statusCode": 429,
  "title": "Too Many Requests",
  "description": "A maximum of 500 requests per hour and user is allowed.",
}
```

The **same** situation in the Document API will be encoded in XML :

```xml
<error statusCode="429" xmlns="https://w3id.org/dts/api#">
  <title>Too Many Requests</title>
  <description>A maximum of 500 requests per hour and user is allowed.</description>
</error>
```

## Base API Endpoint

A client needs to know the URL of the base DTS API endpoint, which is determined by the server. (In this example, the path of the base endpoint is `/dts/api`, but a server can choose a different relative URL.)

If the client does a `GET` on the base API endpoint,, the following response is returned:

```
{
  "@context": "/dts/api/contexts/EntryPoint.jsonld",
  "@id": "/dts/api/",
  "@type": "EntryPoint",
  "collection": "/dts/api/collection/",
  "document": "/dts/api/document/",
  "navigation" : "/dts/api/navigation"
}
```

This response provides the base path for each of the 3 specified endpoints: collection, navigation and document. All other resources can be discovered via navigation of individual endpoint responses.  Base paths to individual endpoints URLs shown in response payloads may vary from these base endpoint paths.
