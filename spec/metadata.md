**** This section is a the general description of how the metadata will be handled

New Section: Resource Metadata

This section defines the model for associating metadata with LWS resources. The LWS metadata system is based on the principles of Web Linking [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288.html), which allows servers to describe the relationships between resources using typed links.

### Metadata Model

All metadata in LWS is expressed as a set of typed links originating from a resource (the link context). Each link consists of:

    A link target: A URI identifying the related resource.

    A relation type: A string that defines the nature of the relationship (e.g., acl, describedby, license).

    Optional target attributes: Additional key-value pairs that further describe the link or the target resource (e.g., type, hreflang, title).

### Discovering Metadata

    Clients discover a resource's metadata primarily through Link headers returned in response to GET or HEAD requests on the resource's URI.

    To manage response verbosity, servers SHOULD support the Prefer header [RFC 7240](https://www.rfc-editor.org/rfc/rfc7240.html). A client can request the inclusion of descriptive metadata links by sending a Prefer header with the include preference. For LWS, the following preference token is defined: http://www.w3.org/ns/lws#metadata.

    To provide finer-grained control over the response payload, the include preference MAY be parameterized with a fields parameter. The value of this parameter is a comma-separated, case-insensitive string of the link attribute names that the client wishes to receive (e.g., href, rel, type).

    A client MAY supply multiple include preferences in a single Prefer header by separating them with a comma. This allows for the retrieval of metadata from multiple vocabularies in a single request. Each include preference is processed independently by the server. Any parameters, such as fields, are scoped locally to the specific include preference they are attached to.

### The Linkset Resource

	For resources with extensive metadata, an LWS server SHOULD expose the complete set of links in a separate linkset resource, as defined in [RFC 9264](https://www.rfc-editor.org/rfc/rfc9264.html). A resource's linkset is discovered via a Link header with the relation type linkset. The linkset resource itself contains a serialized representation of all links. LWS servers MUST support the JSON-based application/linkset+json format.

### Managing Metadata

    Metadata is managed by modifying a resource's associated linkset resource using PUT or PATCH operations.

    Replacing Metadata (PUT): A client can replace the entire set of metadata links by sending a PUT request to the linkset URI with a complete linkset document in the body.

    Partially Updating Metadata (PATCH): A client can add, remove, or modify individual links by sending a PATCH request to the linkset URI. LWS servers SHOULD support a standard patch format, such as JSON Merge Patch [RFC 7396](https://www.rfc-editor.org/rfc/rfc7396.html) (application/merge-patch+json).


************* Add to existing CRUD Section

************** 9.1 Create Resource

New resources are created using either POST (for server-assigned names) or PUT (for client-specified URIs).  Clients MAY provide initial metadata for the new resource by including one or more Link headers in the POST or PUT request, following the syntax of [RFC 8288].

Example (Response to POST):
```
HTTP/1.1 201 Created
Location: /alice/notes/shoppinglist.txt
Content-Type: text/plain; charset=UTF-8
ETag: "def789012"
Link: <.meta>; rel="linkset"; type="application/linkset+json"
Link: </alice/notes/>; rel="up"
Content-Length: 0
```

On success, return 201 Created with the new URI in the Location header. It SHOULD also include Link headers for server-managed metadata, such as a link to the parent container (rel="up") and a link to its dedicated linkset resource (rel="linkset"). The body may be empty.

************** 9.2.1 Retrieving Metadata

Metadata associated with a resource is returned in Link headers in the response to a GET or HEAD request. As described in Section [Resource Metadata], clients can use the Prefer header not only to request the inclusion of metadata but also to specify which link attributes (fields) they wish to receive.

Example (GET a resource with specific metadata fields):
The client requests only the relation type (rel) and media type (type) for each associated link.

```
GET /alice/notes/shoppinglist.txt HTTP/1.1
Host: example.com
Authorization: Bearer <token>
Prefer: include="http://www.w3.org/ns/lws#metadata"; fields="rel,type"
```

Example (Response with reduced Link headers):
The Link header's target URI is always present. The fields parameter controls which of the other key=value attributes are included.
```
HTTP/1.1 200 OK
ETag: "abc123456"
Link: <.meta>; rel="linkset"; type="application/linkset+json"
Link: <.acl>; rel="acl"
Link: </alice/notes/>; rel="up"
Preference-Applied: include="http://www.w3.org/ns/lws#metadata"; fields="rel,type"

... (response body) ...
```

Example (GET a linkset resource with specific fields):
If the client then requests the linkset resource itself, it can apply the same preference to shape the JSON response.

```
GET /alice/notes/shoppinglist.txt.meta HTTP/1.1
Host: example.com
Authorization: Bearer <token>
Accept: application/linkset+json
Prefer: include="http://www.w3.org/ns/lws#metadata"; fields="href,rel,type"
```

Example (Response with reduced linkset representation):
The server returns a JSON document where each link object in the linkset array contains only the requested keys.

```
HTTP/1.1 200 OK
Content-Type: application/linkset+json
ETag: "meta-etag-111"
Preference-Applied: include="http://www.w3.org/ns/lws#metadata"; fields="href,rel,type"

{
  "linkset": [
    {
      "href": "/alice/notes/shoppinglist.txt.acl",
      "rel": "acl"
    },
    {
      "href": "/alice/notes/",
      "rel": "up",
      "type": "text/turtle"
    },
    {
      "href": "/descriptions/groceries.txt",
      "rel": "describedby",
      "type": "text/plain"
    }
  ]
}
```
In this response, the link for rel="acl" does not include a type attribute because it was not present on the server for that link, while the other links include type because it was requested and available. This allows clients to reduce bandwidth and processing load by fetching only the metadata attributes they require.

************** 9.3 Update Resource:

9.3.1 Update Resource Content ( HTTP PUT / PATCH )

Note: This section describes updating a resource's primary content. To update its metadata, see Section 9.3.2.

9.3.2 Update Resource Metadata (HTTP PUT / PATCH on Linkset)

A resource's metadata is updated by modifying its corresponding linkset resource, discovered via the Link header with rel="linkset".

Full Replacement (PUT): A PUT request to the linkset URI with a complete linkset document in the body replaces all metadata for the resource.

Partial Update (PATCH): A PATCH request to the linkset URI (e.g., with application/merge-patch+json) adds, removes, or modifies specific links.

		
************** 9.4 Delete Resource:

The DELETE method removes a resource or container.

Non-container: Deletes the resource and any associated metadata, such as its linkset.

Container: By default, will be rejected with 409 Conflict if not empty. Recursive deletion MAY be supported via the Depth: infinity header.
