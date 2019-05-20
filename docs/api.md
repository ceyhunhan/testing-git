# Hub Backend REST API

The Hub Backend provides a REST API for Hub Clients and Backoffice.

## Hub Client API
The Hub Client API is used by Hub Clients to start, resume and get the current state of a workflow execution.

### Route resource
Route resource represents a route to be rendered by a Hub client.

It contains the following fields

- **uuid**: a route uuid, identifies a route for the purpose of posting route payload and resuming the workflow execution
- **uri**: a route URI, used for client-side routing and rendering a corresponding route view
- **terminal**: marks a terminal route, set if the corresponding execution terminated
- **backEnabled**: determines if it's possible to go back to a previous route
- **export**: a route data, used for setting up a route, e.g. a list of products
- **payload**: a payload used for resuming a previous route, set after going back 
 
#### POST /api/executor/{name}
Creates and starts a new executor for a workflow specified by a _name_. The _name_ can be omitted if the Hub instance has just a single workflow.

Optionally, the body can contain a JSON payload. The payload can be then accessed as `init` context attribute.  

##### Response:
- **201 Created**, a success response with _executor URI_ as a Location header, the body contains a JWT token used for accessing the executor
- **404 Not Found**, a workflow with a _name_ not found, or no default workflow 

An example of a successful response below
```
HTTP/1.1 201 Created
Content-Length: 80
Content-Type: application/json;charset=UTF-8
Location: http://localhost:8080/api/execution/511b0441-d6d0-4a8a-8003-22ef52740847

{
    "uuid": "511b0441-d6d0-4a8a-8003-22ef52740847",
    "uri": "/api/execution/511b0441-d6d0-4a8a-8003-22ef52740847",
    "token": "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI1MTFiMDQ0MS1kNmQwLTRhOGEtODAwMy0yMmVmNTI3NDA4NDciLCJzdWIiOiJleGVjdXRvciIsImlhdCI6MTU1NzI0NjE4OSwiZXhwIjoxNTU3Mjc0OTg5fQ.2GVAuboArO8k1G48CY1ojFdypO9zm9u2ZubCE7Qa-Co"
}
```

#### GET /api/execution/{uuid}
Query the current route using an executor with _uuid_. 

##### Response:
- **200 OK**, a success response with Route resource as a body
- **404 Not Found**, an executor with _uuid_ not found
- **400 Bad Request**, invalid executor _uuid_ or execution error
- **401 Unauthorized**, invalid access token

An example of a successful response below
```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI1MTFiMDQ0MS1kNmQwLTRhOGEtODAwMy0yMmVmNTI3NDA4NDciLCJzdWIiOiJleGVjdXRvciIsImlhdCI6MTU1NzI0NjE4OSwiZXhwIjoxNTU3Mjc0OTg5fQ.2GVAuboArO8k1G48CY1ojFdypO9zm9u2ZubCE7Qa-Co

{ 
  "uuid": "89828e1e-c834-42a2-86f1-893209f63ab5", 
  "uri": "/product", 
  "terminal": false, 
  "backEnabled": false, 
  "export": {"product1": "Product1", "product2": "Product2"}
)

```

#### POST /api/execution/{uuid}
Resumes a workflow execution using an executor with _uuid_.

##### Request:
- **uuid**: the route uuid to resume, i.e. the last route uuid
- **payload**: a JSON payload, i.e. a data user entered and file descriptors for uploaded files
- **files**: a list of uploaded files, each files is specified by _uuid_ and _documentType_ 
 
An example request
```
POST /api/executor/c973a8e7-eb24-4e55-980f-f2ea0fff680e HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI1MTFiMDQ0MS1kNmQwLTRhOGEtODAwMy0yMmVmNTI3NDA4NDciLCJzdWIiOiJleGVjdXRvciIsImlhdCI6MTU1NzI0NjE4OSwiZXhwIjoxNTU3Mjc0OTg5fQ.2GVAuboArO8k1G48CY1ojFdypO9zm9u2ZubCE7Qa-Co

{
    "uuid": "3a0d231f-12b8-47b3-a495-b9418db294b3",
    "payload": {
        "firstname": "Joe",
        "lastname": "Bloke",
        "id_card": {
            "uuid": "a3c6c1ec-ae02-47a6-8cac-8760521f2ed8", 
            "fileName": "my_cool_id_card_photo.png", 
            "mimeType": "application/png", 
            "size": 123123, 
            "expiredOn": "2020-01-01T12:00:00Z" 
        }
    },
    "files": [
        { "uuid":"a3c6c1ec-ae02-47a6-8cac-8760521f2ed8", "documentType":"id_card" }
    ]
}
```

##### Response:
- **200 OK**, a success response, the execution has been resumes, the body contains the next route as Route resource
- **202 Accepted**, a success response, the execution has been resumed
- **422 Unprocessable Entity**, a route validation failed, the body contains a list of validation errors 
- **400 Bad Request**, invalid executor _uuid_ or execution error
- **401 Unauthorized**, invalid access token
- **404 Not Found**, an executor with _uuid_ not found


An example of a successful route response below
```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI1MTFiMDQ0MS1kNmQwLTRhOGEtODAwMy0yMmVmNTI3NDA4NDciLCJzdWIiOiJleGVjdXRvciIsImlhdCI6MTU1NzI0NjE4OSwiZXhwIjoxNTU3Mjc0OTg5fQ.2GVAuboArO8k1G48CY1ojFdypO9zm9u2ZubCE7Qa-Co

{ 
  "uuid": "89828e1e-c834-42a2-86f1-893209f63ab5", 
  "uri": "/product", 
  "terminal": false, 
  "backEnabled": false, 
  "export": {"product1": "Product1", "product2": "Product2"}
)

```

An example of a route validation error response below
```
HTTP/1.1 422 UNPROCESSABLE ENTITY
Content-Type: application/json;charset=UTF-8

{
    "errors": [
        {
            "field": "mobile",
            "message": "Required"
        }
    ]
}
```

#### POST /api/execution/{uuid}/back
Goes back to the previous route using an executor with _uuid_.

##### Request:
- **uuid**: the route uuid to back from, i.e. the last route uuid

An example request
```
POST /api/executor/c973a8e7-eb24-4e55-980f-f2ea0fff680e/back HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI1MTFiMDQ0MS1kNmQwLTRhOGEtODAwMy0yMmVmNTI3NDA4NDciLCJzdWIiOiJleGVjdXRvciIsImlhdCI6MTU1NzI0NjE4OSwiZXhwIjoxNTU3Mjc0OTg5fQ.2GVAuboArO8k1G48CY1ojFdypO9zm9u2ZubCE7Qa-Co

{
    "uuid": "3a0d231f-12b8-47b3-a495-b9418db294b3"
}
```

##### Response:
- **200 OK**, a success response, the body contains the previous route as Route resource, including the previous route resume payload
- **404 Not Found**, an executor with _uuid_ not found
- **400 Bad Request**, invalid executor _uuid_ or execution error
- **401 Unauthorized**, invalid access token

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
    "uuid": "fc05e808-9521-4268-af2d-dcc68a6eb4c9",
    "uri": "/basic",
    "terminal": false,
    "backEnabled": false,
    "payload": {
        "firstname": "Joe",
        "lastname": "Bloke"
    }
}
```

### Security
The execution API endpoints are secured using JWT tokens.

A token is generated when a workflow execution starts. This token is then used for accessing the corresponding workflow execution, identified by _uuid_.

The token expiration is set to 8 hours and can be modified using `jwt.expiration` property.

When using the execution API endpoints, the token must be sent in the `Authorization` header.

```
Authorization: Bearer {token}
```  