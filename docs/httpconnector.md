### HTTP connector
A generic http connector enables making HTTP calls.

Http connector has the following configuration:
- _url_, a request URL (**mandatory**)
- _method_, a request HTTP method, specified as String or [HttpMethod](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpMethod.html) (default **GET**)
- _header_, a request HTTP header (key and value) 
- _contentType_, a request content type, specified as String or [MediaType](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/MediaType.html) 
- _basicAuth_, a HTTP basic authentication using user/password credentials
- _body_, a request body as a String 
- _jsonBody_, a json request body and content type, specified using one of the following ways 
  - _object_, e.g. a map or list
  - _attribute_, specified by an attribute key
  - _jsonBuilder_, a json builder closure, see [JsonBuilder](http://docs.groovy-lang.org/latest/html/gapi/groovy/json/JsonBuilder.html) 

A response is parsed based on the content type and available as an exchange result. There is a built-in support for Json and xml responses. 

A few HTTP connector examples below

A simple url config
```
connector('http') {
    url 'http://sms-portal.us-east-2.elasticbeanstalk.com/messages'
}

```

A jsonBody using _client_ attribute 
```
connector('http') {
    url "${middleware.url}/api/v1/client" 
    method POST
    jsonBody client
}

```

A jsonBody using _object_ 
```
connector('http') {
    url "${middleware.url}/api/v1/client" 
    method POST
    jsonBody firstname: 'Joe', lastname: 'Bloke'
}

```

A jsonBody using _jsonBuilder_ 
```
connector('http') {
    url "${middleware.url}/api/v1/client" 
    method POST
    jsonBody {
        firstname "$firstname"
        lastname "$lastname"
        address( city: "$city",
                 country: "$country",
                 zip: "$zip")
    }
}

```


A full config


```
connector('http') {
    url "http://sms-portal.us-east-2.elasticbeanstalk.com/messages"
    method "GET"
    header "header1", "Header 1"
    body "body"
    contentType APPLICATION_JSON_VALUE
    basicAuth "user", "password"
}

```
