Changelogs
==========

# 1-draft2 to 1-alpha

## Hydra

We do not adhere anymore **strictly** to Hydra, but draw inspiration from it. The stale status of Hydra puyshed us to move away from complete conformance.

## Versioning

Version is now displayed in a `dtsVersion`

## Conformance clarification

- Added a conformance level 0 and a conformance level 1. Conformance level 1 has a single change compared to level 0: the ability to use ranges in Navigation and Document.
- The API now requires implementation of all endpoints. 

## Navigation endpoint changes

- Added a new introduction to the Navigation documentation
- Defined what a citation tree is
- Redefined all classes of object found in the Navigation endpoint (fixes [Issue #184](https://github.com/distributed-text-services/specifications/issues/184) and [Issue #194](https://github.com/distributed-text-services/specifications/issues/194))
- Interactive pagination (user defined page size) has been dropped

### Specification changes

- Reworked the traversal system of the Navigation endpoint in order to clarify the impact of the `down` parameter
  - Clarified that document traversal is in document order and defined it
  - Provided the ability to retrieve the data to create a table of contents (fixes [Issue #226](https://github.com/distributed-text-services/specifications/issues/226))
  - Described the behavior of each combination of query parameters `down`, `start`, `end`, and `ref` (fixes [Issue #91](https://github.com/distributed-text-services/specifications/issues/91) and [Issue #194](https://github.com/distributed-text-services/specifications/issues/194)) 
  - Provided the ability to retrieve siblings of a `CitableUnit`
- Added a `tree` query parameter for handling multiple citation trees (fixes [Issue #142](https://github.com/distributed-text-services/specifications/issues/142), [Issue #223](https://github.com/distributed-text-services/specifications/issues/223), [Issue #202](https://github.com/distributed-text-services/specifications/issues/202))
  - Moved `maxCiteDepth` to be a property of `CitationTree` rather than `Resource`
  - Encapsulated `citeStructure` inside a `citationTrees` property with respective changes to the relevant classes
- Listed a set of query parameter combinations that would return HTTP errors
- Clarified a way to request only the metadata of a `CitableUnit` without its descendants (fixes [Issue #198](https://github.com/distributed-text-services/specifications/issues/198))

#### Renamed query parameters, value and properties

- Changed "max" for `?down` to `-1` when querying for the deepest point in a citation tree 
- Changed the `?id` query parameter name to `?resource`
- Renamed the `id` property of `CitableUnit`s to `identifier`

#### Dropped functionalities

- Dropped the `groupBy` query parameter

## Document Endpoint changes

- Clarified error codes and condition of errors generations.
- Removed URI templates as per Hydra definition.
- Document endpoints does not require TEI anymore, but still recommend providing XML TEI output.

### Specifications changes

- Added attributes to the `<dts:wrapper>` element to allow for identifying specific nodes within the wrapped TEI (Fixes https://github.com/distributed-text-services/specifications/issues/133)
- Removed the requirement for `Link` and `Media-Type` HTTP Response Headers
  - Implementation stil **should** provide such capacity.
- Added implementation of multiple trees through the `?tree` parameter ( (fixes https://github.com/distributed-text-services/specifications/issues/142, https://github.com/distributed-text-services/specifications/issues/223, https://github.com/distributed-text-services/specifications/issues/202)

#### Renamed query parameters, value and properties

- Renamed `?id` query parameter identifying a `Resource` to `?resource`
- Renamed the `?format` query parameter for content-negociation to `?mediaType` (Implements parts of https://github.com/distributed-text-services/specifications/issues/225)
- Renamed the `<dts:fragment>` XML node for XML/TEI responses to `<dts:wrapper>`.

## Collection Endpoints changes

- Interactive pagination (user defined page size) has been dropped
