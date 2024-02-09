# Conformance

This section defines the conformance requirements for DTS servers.

DTS does not define conformance requirements for clients.  Any client
that can use a DTS server successfully for its own purposes can be
called a DTS client.  Few clients will use all DTS features.

All DTS servers **MUST** implement all endpoints:

- Entry
- Collection
- Document
- Navigation

All DTS servers **MUST** implement all valid calls on these endpoints as
documented, with one exception:

A Level 0 DTS Server need not support the `start` and `end` parameters
for the Navigation endpoint.  It **SHOULD** raise an error if either of
these parameters are used in a request.

Note: Level 0 conformance allows minimal DTS implementations,
including implementations created by generating static files.

A Level 1 DTS Server supports all valid calls as documented in this
specification.
