# Workflow engine

Each workflow execution is assigned an **Execution Context** and a **unique ID** (UUID). The UUID identifies a Customer throughout the whole workflow execution. 
The Execution context stores the current state of a workflow execution as a set of context attributes.

### Workflow Execution

- execution termination
- route checkpoint
- route validation
- exchange validation
- failed exchange

### Executor life-cycle

Each workflow execution is handled by a separate executor. 
- executor persistence
- DSL executor context discarded after execution terminated
- executor expiration set using `hub.executor.expiration`, the default is 8 hours

### DSL
Each DO process is defined by a workflow using a custom DSL (Domain Specific Language).
 
The workflow DSL provides the following commands:

#### route
Executes a route to be rendered by a Hub Client. A route is specified by a _name_ and _uri_. _Export_ can be used for passing data to a corresponding route view.
When a route result is submitted (via Hub Client), the result payload is validated using the _payload specification_ and stored as context attributes under the _namespace_. 
```
route('name') {
    uri '/uri'
    export data
    validate { payload specification }
    namespace namespace
    checkpoint()
    terminal(result)
}
```
- **name** for registration and logging purposes
- **uri** for client-side routing
- **export** data passed to a Hub Client, can be any serializable object, e.g. string, map, list, context attribute via its key
- **validate** specifies the route result payload validation
- **namespace** for exporting route result as context attribute
- **checkpoint()** marks a route as a checkpoint, i.e. disables going back from this route
- **terminal()** marks a route as terminal and checkpoint, a result can be specified or omitted

See [route examples](/docs/example/routedsl.md) for more details.


#### path
Executes a registered path (sub-workflow) specified by a _name_.
```
path 'name'

```

#### exchange
Makes an external (API) call using a connector (specified by a _type_) with a connector-specific _configuration_. 
To handle a connector failures, a _timeout_, retries_ and _fallback_ can be specified.
A randomized exponential backoff strategy is used for connector retries. 
An exchange is executed asynchronously when marked with _async()_. 
The connector result is validated using a _payload specification_ and stored as a context attribute under the _namespace_.
```
exchange('name') {
    connector('type') {
        configuration
    }
    async()
    timeout 30
    retry 3
    retryBackoff 5
    validate { payload specification }
    namespace namespace
}

```
- **name** for registration and logging purposes
- **connector** for making an external call, a connector is specified by a _type_ and a custom configuration, see [Connectors](docs/connectors.md)
- **async()** marks an exchange to be executed asynchronously
- **timeout** a number of seconds before an exchange timeouts and fails, the default timeout is 30s
- **retry** a number of retries in case of a connector failure, the default number of retries is 3
- **retryBackoff** a number of seconds before first retry, the default retryBackoff is 5s
- **validate** specifies an exchange result validation
- **namespace** for exporting exchange result as context attribute


#### match
Executes a _workflow_ when an _expression_ is true. The expression can contain context attributes.
```
match (expression) {
    workflow
}

``` 

#### switch / case
The switch statement matches expression with cases and executes the matching case. It's a fallthrough switch-case. You can share the same code for multiple matches or 
 use the _break_ command. It uses different kinds of matching like, collection case, regular expression case, closure case and equals case.
```
switch (expression) {
    case "bar":
        route "Bar"
    
    case ~/fo*/:
        route "Foo"
        break

    case [4, 5, 6, 'inList']:
        route "Matched
        break
        
    default:
        route "Default"
}

``` 

#### loop-until
Executes a _workflow_ until an _expression_ is true.
```
loop {
    workflow
} until (expression)
```

### Context attributes
The Context attributes define a data model for the corresponding workflow definition. The attributes are **hierarchical** and are used for sharing data between 
different workflow commands and making flow control decisions.
 
Each attribute can be accessed by its key, e.g. _client.address.city_

There are several way the Context attributes can be updated:
- a route result is stored using the route namespace 
- an exchange result is stored using the exchange namespace
- setting an attribute directly using an object, route or exchange result
 
  
```
config.logo << 'http://logo.png'
products << [product1: "Product1", product2: "Product2"]
client.basic << route('basic info')
credit << exchange('credit check')
```

### Payload validation 
The _validate_ block is used for route and exchange results validation.
It specifies a result (fields) structure and data constrains. 
A field is defined by a _name_, data constrains (validators) and nested fields. The default validator for a field is mandatory, i.e. a field is mandatory if listed.

You can use the following validators
- **string()** a field must be a string
- **number()** a field must be a number
- **truefalse()** a field must be a boolean
- **file()** a field must be a file descriptor
- **file mimeType** a field must be a file descriptor and match the mime type, e.g. `application/pdf`, `image`, etc.
- **values value1, value2, ...** a list of values, a field data must be one of the values
- **regex ~/pattern/** a regular expression to match a field data, the expression can be a string or a groovy regex pattern syntax

A validate example below
```
validate {
    firstname
    lastname
    address {
            city { values "Prague", "Paris" }
            zip { regex ~/^[0-9]{5}(?:-[0-9]{4})?$/ }
        }
}
``` 
See [validate examples](/docs/example/validate.md) for more details.

## Workflow validation
- check for invalid DSL commands
- validate DSL command specification (uri is mandatory for route)
- validate connector configurations (http connector must have url...)
- validate context attribute usage, i.e. if an attribute was set before being used
 