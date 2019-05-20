### Validate payload Examples

- a single mandatory field
```
validate {
    mobile
}
```

- a mandatory field with regex validator
```
validate {
     mobile { regex ~/[0-9]{5}/ }
}
```

- a mandatory field with value validator
```
validate {
     product { values "product1", "product2", "product3"}
}
```

- mandatory nested fields
```
validate {
    firstname
    lastname
    address {
        street
        city
        state
        zip
    }
}
```
