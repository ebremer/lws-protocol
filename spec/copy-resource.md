## COPY Operation

### 7.6 Copy Resource

The copy resource operation requests the duplication of an existing resource on the server, creating a new independent copy at the specified destination. This applies to both non-container and container resources. The copy is atomic and does not affect the original resource.

Inputs:
- Source identifier: The full identifier of the existing resource to copy.
- Destination identifier: The full target identifier for the new copy.
- Overwrite flag: Optional; specifies whether to overwrite if the destination exists.
- Depth: Optional for containers; "infinite" (default) for recursive copy of all members, or "0" for shallow copy (container only, without members).

Behavior:
- The server verifies the source exists and the client has read access to it, plus write access to the destination.
- The copy creates a new resource at the destination with identical content, metadata, and media type as the source. For containers, a recursive copy includes all members, preserving structure.
- The operation is atomic: success creates the full copy; failure leaves no partial copies and the destination unchanged.
- The original source remains unchanged. The server MAY assign new identifiers or metadata (e.g., new ETags) to the copy.
- Constraints may include quota checks at destination or restrictions on copying certain resource types.

Possible Responses:
- Copied: Success; response includes the new destination identifier and metadata.
- Bad Request: Malformed request.
- Not Permitted: Lacking permissions.
- Conflict: Destination exists without overwrite.
- Not Found: Source not found.
- Unknown Error: Server error.
