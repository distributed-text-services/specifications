Document Endpoint
=================

The Document endpoint is used to access the content of document, as opposed to metadata (which is found in collections).

## Query Parameters

The Document endpoint supports the following query parameters:

| Name | Type | Description                              | Methods | Constraints |
|------|------ | ------------------------------------|---------| ----------- |
| resource   | URI | The unique identifier for the `Resource` whose tree or subtree must be returned |  GET    | Required |
| ref | string | The string identifier of a single node in the `CitationTree` for the `Resource`, used as the root for the sub-tree to be reconstructed. | GET    | NOT used with `?start` and `?end` |
| start | string | The string identifier of a node in the `CitationTree` for the `Resource`, used as the starting point for a range that serves as the reference point for the query. This parameter is inclusive, so the starting point is considered part of the sub-tree to be returned. | GET | NOT used if a `?ref` is specified, requires `?end` as well |
| end |  string | The string identifier of a node in the `CitationTree` for the `Resource`, used as the ending point for a range that serves as the reference point for the query. This parameter is inclusive, so the supplied ending point is considered part of the specified range. | GET | NOT used if a `?ref` is specified, requires `?start` as well |
| tree | string | The string identifier for a `CitationTree` of the `Resource`. | GET | NOT used to query the default `CitationTree` |
| mediaType | string | The string identifier for the media-type the resource must be returned in | GET | |

For parameter combinations and potential errors, see [Link](#)

### Usages

#### Query parameters combinations and requirements

- The `?resource` query parameter **must** be provided.
- The `?tree` query parameter **must** be omitted to address the default `CitationTree` of a `Resource`.
- The `?tree` query parameter **must** be provided to address a `CitableUnit` within a different `CitationTree` from the default.
- The `?ref` query parameter **cannot** be used in combination with `?start` and `?end`.
- The `?start` query parameter **must** be used in combination with `?end`.
- The `?end` query parameter **must** be used in combination with `?start`.

### Errors

Some combination of query parameters and their values **must** produce 4xx HTTP Errors:

- A `400 Bad Request` error **should** be returned when the `?resource` parameter is missing.
- A `404 Not Found` error **should** be returned when any combination of `?resource`, `?tree`, `?ref`, `?start` and `?end` does not match a `Resource` and its `CitationTree`.
- A `404 Not Found` error **should** be returned when the requested `?mediaType` is not available for the `Resource` identified by `?resource` in the `Navigation` endpoint. 

### Formats and limitations of the Endpoint

#### TEI as a major format for the Document endpoint

`Document` endpoint **should** return textual data in an XML format compliant with the Text Encoding Initiative (TEI) guidelines (`application/tei+xml` response format). The XML **must** be well formed and **should** be valid. XML/TEI **should** be the default response media-type.

`Document` endpoint **may** not implement TEI media-type.

`Document` endpoints **may** return requested data in as many other formats as the content provider wishes.

#### XML/TEI encoding and sub-tree representation

The root node of the XML response **must** be the `<TEI>` root element of the namespace `http://www.tei-c.org/ns/1.0`. So a response would normally look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <!-- XML of the document requested here -->
</TEI>
```

If the data to be returned is not a complete document, but a chunk using either `?ref`, or `?start`/`?end`, the chunk of the document **should** be embedded in the `<wrapper>` element of the DTS Namespace (`https://w3id.org/dts/api#`) like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:wrapper xmlns:dts="https://w3id.org/dts/api#">
    <!-- XML or string of the passage requested here -->
  </dts:wrapper>
</TEI>
```

There is no limitation as to what can be contained by `<dts:wrapper>` or if its siblings can be provided as long as they are well formed and valid. 

The only limiting factor is that `<dts:wrapper>` **must** contain the requested textual data. This permits an implementation to return contextual material elsewhere within the root `TEI` node alongside the requested fragment.

The `<dts:wrapper>` **may** provides the attributes `@ref`, `@start` and `@end` that contains `xpath` that identifies the elements identified by `?ref`, `?start` and `?end`. It is recommended to provide these if the excerpt contains content from other chunks excluded by the sub-tree identified by `?ref`, `?start` and `?end`.

#### Batch Requests

The `Document` endpoint **does not support** batch requests. 

For the sake of simplicity and usability the DTS guidelines do not allow for batch operations, i.e. fetching or altering more than one passage of text in the same request. Implementations should encourage client software and other consumers of their API to use multiple asynchronous requests instead.

## Response

### Header

A `Document` endpoint **should** provide the following entries in their HTTP response header:

- `Link` **should** contain a URI that links back to the `Collection` endpoint for the requested `Resource`, e.g. as `Link: </dts/api/collection/?id=https://en.wikisource.org/wiki/Dracula; rel="collection"`
- `Content-Type` **should** contain the media-type of the response, e.g. `Content-Type: application/tei+xml`

## Examples

### Writing Convention for the Examples

In the examples, HTML comments such as `<!-- ... -->` are used to shorten the amount of information represented, and indicate that more information may be found at this point in the Response body.

### Example 1: Retrieve a subtree using `?ref`

Retrieve the passage `C1` of the document labeled by the identifier `https://en.wikisource.org/wiki/Dracula`.

#### Request URL

- `https://example.org/dts/api/document/?resource=https://en.wikisource.org/wiki/Dracula&ref=C1`

#### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | <https://example.org//dts/api/collection/?resource=https://en.wikisource.org/wiki/Dracula>; rel="collection" |

#### Successful response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <teiHeader>
      <!-- ... -->
   </teiHeader>
   <text>
      <body>
         <head>DRACULA</head>
         <dts:wrapper xmlns:dts="https://w3id.org/dts/api#">
            <div type="chapter" n="1">
               <head>CHAPTER I</head>
               <head>JONATHAN HARKER'S JOURNAL</head>
               <note>(Kept in shorthand.)</note>
               <div type="entry">
                  <p n="1"><date>3 May</date>. <placeName>Bistritz</placeName>.—Left <placeName>Munich</placeName> at <date>8:35 p. m., on 1st May</date>, arriving at Vienna early next morning; should have arrived at 6:46, but train was an hour late. Buda-Pesth seems a wonderful place, from the glimpse which I got of it from the train and the little I could walk through the streets. I feared to go very far from the station, as we arrived late and would start as near the correct time as possible. The impression I had was that we were leaving the West and entering the East; the most western of splendid bridges over the Danube, which is here of noble width and depth, took us among the traditions of Turkish rule.</p>
                  <p n="2">We left in pretty good time, and came after nightfall to Klausenburgh. Here I stopped for the night at the Hotel Royale. I had for dinner, or rather supper, a chicken done up some way with red pepper, which was very good but thirsty. (Mem., get recipe for Mina.) I asked the waiter, and he said it was called "paprika hendl," and that, as it was a national dish, I should be able to get it anywhere along the Carpathians. I found my smattering of German very useful here; indeed, I don't know how I should be able to get on without it.</p>
                  <!--- ... -->
               </div>
               <div type="entry">
                  <p n="1"><date>4 May</date>. I found that my landlord had got a letter from the Count, directing him to secure the best place on the coach for me; but on making inquiries as to details he seemed somewhat reticent, and pretended that he could not understand my German. This could not be true, because up to then he had understood it perfectly; at least, he answered my questions exactly as if he did. He and his wife, the old lady who had received me, looked at each other in a frightened sort of way. He mumbled out that the money had been sent in a letter, and that was all he knew. When I asked him if he knew Count Dracula, and could tell me anything of his castle, both he and his wife crossed themselves, and, saying that they knew nothing at all, simply refused to speak further. It was so near the time of starting that I had no time to ask any one else, for it was all very mysterious and not by any means comforting.</p>
                  <p n="2">Just before I was leaving, the old lady came up to my room and said in a very hysterical way: </p>
                  <!--- ... -->
               </div>
               <!--- ... -->
            </div>
         </dts:wrapper>
      </body>
   </text>
</TEI>
```

### Example 2: Retrieve a subtree using `start` and `end`

Retrieve the `CitableUnit` `C1.E1,P1` to the `CitableUnit` `C1.E1,P1` of the `Resource` identified by `https://en.wikisource.org/wiki/Dracula`.

#### Request url

- `https://example.org/dts/api/document/?resource=https://en.wikisource.org/wiki/Dracula&start=C1.E1,P1&end=C1.E1,P2`


#### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | <https://example.org//dts/api/collection/?id=https://papyri.info/ddbdp/bgu;11;2029/source>; rel="collection" |

#### Successful response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <teiHeader>
      <!-- ... -->
   </teiHeader>
   <text>
      <body>
         <head>DRACULA</head>
         <dts:wrapper xmlns:dts="https://w3id.org/dts/api#">
            <div type="chapter" n="1">
               <div type="entry">
                  <p n="1"><date>3 May</date>. <placeName>Bistritz</placeName>.—Left <placeName>Munich</placeName> at <date>8:35 p. m., on 1st May</date>, arriving at Vienna early next morning; should have arrived at 6:46, but train was an hour late. Buda-Pesth seems a wonderful place, from the glimpse which I got of it from the train and the little I could walk through the streets. I feared to go very far from the station, as we arrived late and would start as near the correct time as possible. The impression I had was that we were leaving the West and entering the East; the most western of splendid bridges over the Danube, which is here of noble width and depth, took us among the traditions of Turkish rule.</p>
                  <p n="2">We left in pretty good time, and came after nightfall to Klausenburgh. Here I stopped for the night at the Hotel Royale. I had for dinner, or rather supper, a chicken done up some way with red pepper, which was very good but thirsty. (Mem., get recipe for Mina.) I asked the waiter, and he said it was called "paprika hendl," and that, as it was a national dish, I should be able to get it anywhere along the Carpathians. I found my smattering of German very useful here; indeed, I don't know how I should be able to get on without it.</p>
               </div>
            </div>
         </dts:wrapper>
      </body>
   </text>
</TEI>
```

### Example 3: Retrieve a full `Resource`'s tree

Retrieve the full content of a `Resource` labeled by the identifier `https://en.wikisource.org/wiki/Dracula`

#### Request URL

- `https://example.org/dts/api/document/?resource=https://en.wikisource.org/wiki/Dracula`

#### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | <https://example.org/dts/api/collection/?id=https://en.wikisource.org/wiki/Dracula>; rel="collection" |

#### Successful response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <teiHeader>
      <!-- ... -->
   </teiHeader>
   <text>
      <body>
         <head>DRACULA</head>
            <div type="chapter" n="1">
               <head>CHAPTER I</head>
               <head>JONATHAN HARKER'S JOURNAL</head>
               <note>(Kept in shorthand.)</note>
               <div type="entry">
                  <p n="1"><date>3 May</date>. <placeName>Bistritz</placeName>.—Left <placeName>Munich</placeName> at <date>8:35 p. m., on 1st May</date>, arriving at Vienna early next morning; should have arrived at 6:46, but train was an hour late. Buda-Pesth seems a wonderful place, from the glimpse which I got of it from the train and the little I could walk through the streets. I feared to go very far from the station, as we arrived late and would start as near the correct time as possible. The impression I had was that we were leaving the West and entering the East; the most western of splendid bridges over the Danube, which is here of noble width and depth, took us among the traditions of Turkish rule.</p>
                  <p n="2">We left in pretty good time, and came after nightfall to Klausenburgh. Here I stopped for the night at the Hotel Royale. I had for dinner, or rather supper, a chicken done up some way with red pepper, which was very good but thirsty. (Mem., get recipe for Mina.) I asked the waiter, and he said it was called "paprika hendl," and that, as it was a national dish, I should be able to get it anywhere along the Carpathians. I found my smattering of German very useful here; indeed, I don't know how I should be able to get on without it.</p>
                  <!--- ... -->
               </div>
               <div type="entry">
                  <p n="1"><date>4 May</date>. I found that my landlord had got a letter from the Count, directing him to secure the best place on the coach for me; but on making inquiries as to details he seemed somewhat reticent, and pretended that he could not understand my German. This could not be true, because up to then he had understood it perfectly; at least, he answered my questions exactly as if he did. He and his wife, the old lady who had received me, looked at each other in a frightened sort of way. He mumbled out that the money had been sent in a letter, and that was all he knew. When I asked him if he knew Count Dracula, and could tell me anything of his castle, both he and his wife crossed themselves, and, saying that they knew nothing at all, simply refused to speak further. It was so near the time of starting that I had no time to ask any one else, for it was all very mysterious and not by any means comforting.</p>
                  <p n="2">Just before I was leaving, the old lady came up to my room and said in a very hysterical way: </p>
                  <!--- ... -->
               </div>
               <!--- ... -->
            </div>
            <div type="chapter" n="2">
               <!--- ... -->
            </div>
            <!--- ... -->
      </body>
   </text>
</TEI>
```

### Example 4: Retrieve part of a `Resource` as `text/html`

Retrieve the subtree of the `CitableUnit` identified by `C1.E1,P1` of the `Resource` labeled by the identifier `https://en.wikisource.org/wiki/Dracula` as HTML content. 

#### Request URL

- `https://example.org/dts/api/document/?resource=https://en.wikisource.org/wiki/Dracula&mediaType=text/html`

#### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: text/html |
| Link | <https://example.org/dts/api/collection/?id=https://en.wikisource.org/wiki/Dracula>; rel="collection" |

#### Successful response body

```xml
<!DOCTYPE html>
<html>
   <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Dracula, Chapter 1, Entry 1, Paragraph 1</title>
   </head>
   <body>
      <p>
      <i>3 May. Bistritz</i>.—Left Munich at 8:35 <span class="smallcaps" style="font-variant:small-caps;">p. m.</span>, on 1st May, arriving at Vienna early next morning; should have arrived at 6:46, but train was an hour late. Buda-Pesth seems a wonderful place, from the glimpse which I got of it from the train and the little I could walk through the streets. I feared to go very far from the station, as we arrived late and would start as near the correct time as possible. The impression I had was that we were leaving the West and entering the East; the most western of splendid bridges over the Danube, which is here of noble width and depth, took us among the traditions of Turkish rule.
      </p>
   </body>
</html>
```


### Example 5: Retrieve part of a `Resource` as `text/html` of a non-default `CitationTree`

Retrieve the subtree of the `CitableUnit` identified by `5` of the `Resource` labeled by the identifier `https://en.wikisource.org/wiki/Dracula` using its `CitationTree` identified `pages` as HTML content. 

#### Request URL

- `https://example.org/dts/api/document/?resource=https://en.wikisource.org/wiki/Dracula&mediaType=text/html&tree=pages&ref=5`

#### Recommended response headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: text/html |
| Link | <https://example.org/dts/api/collection/?id=https://en.wikisource.org/wiki/Dracula>; rel="collection" |

#### Successful response body

```xml
<!DOCTYPE html>
<html>
   <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Dracula, Page 5</title>
   </head>
   <body>
      <p>simply refused to speak further. It was so near the time of starting that I had no time to ask any one else, for it was all very mysterious and not by any means comforting.</p>
      <p>Just before I was leaving, the old lady came up to my room and said in a very hysterical way:</p>
      <p>"Must you go? Oh! young Herr, must you go?" She was in such an excited state that she seemed to have lost her grip of what German she knew, and mixed it all up with some other language which I did not know at all. I was just able to follow her by asking many questions. When I told her that I must go at once, and that I was engaged on important business, she asked again:</p>
      <p>"Do you know what day it is?" I answered that it was the fourth of May. She shook her head as she said again:</p>
      <!-- ... -->
   </body>
</html>
```
