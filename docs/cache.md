# Hub file cache REST API

The Hub  provides a REST API for caching client-uploaded files.
This allows to use only cached file descriptors for form processing and avoid using multipart data.
Also this feature improves UX because user can upload files separately and it seems to be faster for him than batch upload 

#### POST /api/files/cache

Uploads new file to cache

##### Request:
- **file**: multipart representation of uploaded file
 
An example request
```
POST /api/files/cache HTTP/1.1
Host: localhost:8080
Content-Type: multipart/form-data
```

##### Response:
- **201 Created**, a success response, file was added to cache, file descriptor is returned as a result:

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{ 
  "uuid": "89828e1e-c834-42a2-86f1-893209f63ab5", 
  "fileName": "my_file.pdf", 
  "mimeType": "application/pdf", 
  "size": 123123, 
  "expiredOn": "2020-01-01T12:00:00Z"
)

```

- **400 Bad Request**, error during file upload


#### DELETE /api/files/cache/{uuid}

Removes file with _uuid_ from cache (deletes from server)

##### Request:
- **uuid**: UUID of file in cache
 
An example request
```
POST /api/files/cache/9828e1e-c834-42a2-86f1-893209f63ab5 HTTP/1.1
Host: localhost:8080
Content-Type: application/json;charset=UTF-8
```

##### Response:
- **200 OK**, a success response, file was removed from cache

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
```

- **400 Bad Request**, error during file deleting