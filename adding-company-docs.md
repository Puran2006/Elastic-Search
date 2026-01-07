# Adding documents

## Adding a company

```json
PUT /company/_doc/1
{
  "name": "My Company Inc.",
  "join_field": "company"
}
```

## Adding a department

```json
PUT /company/_doc/2?routing=1
{
  "name": "Development",
  "join_field": {
    "name": "department",
    "parent": 1
  }
}
```

## Adding an employee

```json
PUT /company/_doc/3?routing=1
{
  "name": "Bo Andersen",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}
```

## Adding some more test data
```json
PUT /company/_doc/4
{
  "name": "Another Company, Inc.",
  "join_field": "company"
}
```

```json
PUT /company/_doc/5?routing=4
{
  "name": "Marketing",
  "join_field": {
    "name": "department",
    "parent": 4
  }
}
```

```json
PUT /company/_doc/6?routing=4
{
  "name": "John Doe",
  "join_field": {
    "name": "employee",
    "parent": 5
  }
}
```
