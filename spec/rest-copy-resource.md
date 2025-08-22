## COPY Operation

### 9.6 Copy Resource (HTTP COPY)

Bound to HTTP COPY method per WebDAV [[RFC 4918](https://datatracker.ietf.org/doc/html/rfc4918)]. Uses Destination header for target.

Request:
- Method: COPY
- Headers:
  - Destination: Target URI.
  - Overwrite: "true" or "false".
  - Depth: "infinite" or "0" for containers.
  - Authorization: As needed.

Example Request (copying a file):
```
COPY /alice/notes/shoppinglist.txt HTTP/1.1
Host: example.com
Authorization: Bearer <token>
Destination: /alice/backup/shoppinglist.txt
Overwrite: false
```

Response:
- Success: 201 Created or 204 No Content.

Example Success Response:
```
HTTP/1.1 201 Created
Location: /alice/backup/shoppinglist.txt
ETag: "ghi123456"
Content-Length: 0
```

Status Codes Mapping:
| Abstract Response | HTTP Status Code | Description |
|-------------------|------------------|-------------|
| Copied           | 201 Created     | New copy created. |
| Copied           | 204 No Content  | Overwritten. |
| Bad Request      | 400 Bad Request | Invalid syntax. |
| Not Permitted    | 403 Forbidden   | Unauthorized. |
| Conflict         | 412 Precondition Failed | Conflict on overwrite. |
| Not Found        | 404 Not Found   | Source missing. |
| Unknown Error    | 500 Internal Server Error | Error occurred. |
