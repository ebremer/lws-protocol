## MOVE Operation

### 7.5 Move Resource

The move resource operation requests the relocation or renaming of an existing resource on the server. This can apply to both non-container (data) resources and container resources. The operation is atomic, ensuring that the resource is fully moved or the operation fails without partial changes.

Inputs:
- Source identifier: The full identifier (e.g., URI or path) of the existing resource to be moved.
- Destination identifier: The full target identifier where the resource should be moved to.
- Overwrite flag: An optional indicator (e.g., boolean or header) specifying whether to overwrite an existing resource at the destination if one exists. If not provided, the server MUST fail if the destination already exists.

Behavior:
- The server verifies the existence of the source resource and checks if the client has permission to move it (e.g., read/write access at source and write access at destination).
- If the destination is within the same container or hierarchy, this may act as a rename. If across different containers, it relocates the resource while updating container memberships accordingly.
- For containers, the move MUST be recursive: all member resources are moved with the container, preserving the hierarchy.
- The server MUST ensure atomicity — if any part of the move fails (e.g., due to conflicts or permissions), the source resource remains unchanged, and no changes are made at the destination.
- Upon success, the source identifier becomes invalid (resource is removed from source), and the resource is accessible at the new destination with the same content, metadata, and media type. The server MAY update any internal links or references, but clients are responsible for handling external references.
- The server MAY enforce constraints, such as preventing moves that would create cycles in the hierarchy or exceed quotas at the destination.

Possible Responses:
- Moved: The operation succeeded. The response includes the new destination identifier and possibly updated metadata (e.g., a new ETag).
- Bad Request: The request was malformed (e.g., invalid source or destination identifiers).
- Not Permitted: The client lacks authorization to move the resource or write to the destination.
- Conflict: The destination already exists, and overwrite was not allowed or failed.
- Not Found: The source resource does not exist.
- Unknown Error: An internal server error prevented the move.
