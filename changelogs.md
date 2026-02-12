Changelogs
==========

# 1-alpha to 1.0

- 2026-02-04
  - Fixed ([Issue 271](https://github.com/distributed-text-services/specifications/issues/271)) to have extensions and dublinCore be nested JSON LD properties
  - Clarification on the fact that Citation Trees without a reference represent the same document.
- 2025-07-11 : 1.0 Release Candidate 1 !
- 2025-04-17
  - Fixed formatting issues.
  - Removed ambiguity around the section "Handling Requests with No Matching `CitableUnit`s at the Requested Level(s)" ([Issue 268](https://github.com/distributed-text-services/specifications/issues/268))
  - Clarified the behaviour of `?down=0` while no `start/end`/`ref` are provided (=400 Bad Request Error) ([Issue 269](https://github.com/distributed-text-services/specifications/issues/269))
  - Allow `CitableUnit` to have `@id` for Linked Data usages ([Issue 274](https://github.com/distributed-text-services/specifications/issues/274)).
  - Clarify that headers of the Entry Endpoint SHOULD be JSON+LD ([Issue 272](https://github.com/distributed-text-services/specifications/issues/272)).
  - Clarify that `MetadataObject` MUST have their vocabularies defined, either by the main `@context` property if they are reusing base DTS vocabulary (including Dublin Core Terms) or by their own `@context` property.
- 2024-08-08
  - Made `citeType` required for `CiteStructure` objects.
  - Removed `maxCiteDepth` everywhere, including in the example.
- 2024-08-06
  - Removed `maxCiteDepth` from Collection objects at the Collection endpoint now that we have a `CitationTree` object at the Resource level.
  - Harmonized the properties for `Resource` objects between the Collection and Navigation endpoints. This involves adding required `document` and `navigation` properties, as well as allowing additional optional properties. Also removed the `document` and `navigation` properties from the `Navigation` object since these URI templates are now part of the `Resource` object.
  - Added `view` properties for paginated responses to the Collection and Navigation endpoints. Added a `Pagination` object type to provide the pagination links.
- 2024-05-24
  - Removed `totalItems` from the Collection Endpoints (See https://github.com/distributed-text-services/specifications/issues/248, problem pointed out by @kbrueckmann)
  - `passage` property (URI template) moved to `document` for consistency between Collection and Navigation endpoint (See https://github.com/distributed-text-services/specifications/issues/249, problem pointed out by @philippepons)
    - Harmonized the examples to have the resource already populated, as per Collection URI Templates
  - `collection` property (URI template) added to `Collection` and `Resource` object (See https://github.com/distributed-text-services/specifications/issues/250, problem pointed out by @philippepons)
  - Fixed a typo in the Entry endpoint table, where it referenced Resources.

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
