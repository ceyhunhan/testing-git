### Route DSL Examples

- a terminal route 
```
route("Finish") {
    uri "/finish"
    terminal()
}
```

- a terminal route with a checkpoint
```
route("Rejected") {
    uri "/rejected"
    checkpoint()
    terminal()
}
```

- a route with result validation 
```
route("Basic Info") {
    uri "/basic"
    namespace client
    validate {
        firstname
        lastname
        address {
            street
            city { values "Prague", "Paris" }
            zip { regex ~/^[0-9]{5}(?:-[0-9]{4})?$/ }
        }
    }
}
```

- a route exporting a static map 
```
route("Select Product") {
    uri "/product"
    namespace product
    export  product1: "Product 1", 
            product2: "Product 2", 
            product3: "Product 3"
    validate {
        values "product1", "product2", "product3"
    }
}
```

- a route exporting an attribute
```
delivery.address << [street: "Dejvicka 18", city: "Prague", zip: 12345]

route("Delivery address") {
    uri "/delivery"
    export address: delivery.address
}
```

- a route registration
```
register {
    route("Delivery address") {
        uri "/delivery"
        export delivery.address
    }
}

route("Delivery address") {
    export client.address
}
```
