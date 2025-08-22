## MOVE Operation

### 9.5 Move Resource (HTTP MOVE)

The move resource operation is bound to the HTTP MOVE method as defined in WebDAV [[RFC 4918](https://datatracker.ietf.org/doc/html/rfc4918)]. This method relocates the resource from the request-URI to the URI specified in the Destination header.

Request:
- Method: MOVE
- Headers:
  - Destination: The absolute URI of the target location.
  - Overwrite: "true" to allow overwriting an existing resource at the destination; "false" otherwise (default is "false").
  - Authorization: As required for authentication.
  - If-Match or If-None-Match: Optional for conditional moves based on ETags.

Example Request (moving a file):
```
MOVE /alice/notes/shoppinglist.txt HTTP/1.1
Host: example.com
Authorization: Bearer <token>
Destination: /alice/documents/shoppinglist.txt
Overwrite: false
```

Response:
- Success: 201 Created (if destination was created) or 204 No Content (if overwritten).
- Headers: Location (new URI if applicable), ETag (updated if changed).

Example Success Response:
```
HTTP/1.1 201 Created
Location: /alice/documents/shoppinglist.txt
ETag: "def789012"
Content-Length: 0
```

Status Codes Mapping:
| Abstract Response | HTTP Status Code | Description |
|-------------------|------------------|-------------|
| Moved            | 201 Created     | New resource created at destination. |
| Moved            | 204 No Content  | Overwritten at destination. |
| Bad Request      | 400 Bad Request | Invalid request syntax. |
| Not Permitted    | 403 Forbidden   | Unauthorized. |
| Conflict         | 412 Precondition Failed | Overwrite not allowed or precondition failed. |
| Not Found        | 404 Not Found   | Source not found. |
| Unknown Error    | 500 Internal Server Error | Server error. |

