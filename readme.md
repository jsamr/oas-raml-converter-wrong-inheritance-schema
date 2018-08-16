# In OAS3, converter should never use `$ref` at schema root

`oas-raml-coverter version 1.1.35`

In the provided example, `Foo` `description` field is lost when converted to OAS3 because of the use of `$ref` which should be replaced by `allOf`.

The reason is explained in the [OAS3 specification document](https://swagger.io/docs/specification/using-ref/):

> Any sibling elements of a `$ref` are ignored. This is because `$ref` works by replacing itself and everything on its level with the definition it is pointing at.

To reproduce, just run

``` text
oas-raml-converter --from RAML --to OAS30 api.raml > api.oas.json
```

### Raml definition

``` yaml
#%RAML 1.0
title: Bug report
types:
  FooCore:
    description: This is FooCore
    properties:
      name: string
  Foo:
    description: This is Foo, not FooCore
    type: FooCore
```

### Expected output (oas3+json, irrelevant fields removed)

``` json
{
  "openapi": "3.0.0",
  "info": {
    "title": "Bug report",
    "version": "undefined"
  },
  "components": {
    "schemas": {
      "FooCore": {
        "description": "This is FooCore",
        "properties": {
          "name": {
            "type": "string"
          }
        },
        "required": [
          "name"
        ],
        "type": "object"
      },
      "Foo": {
        "description": "This is Foo, not FooCore",
        "allOf": [
          {
            "$ref": "#/components/schemas/FooCore"
          }
        ]
      }
    }
  }
}

```

### Observed output (oas3+json, irrelevant fields removed)

``` json
{
  "openapi": "3.0.0",
  "info": {
    "title": "Bug report",
    "version": "undefined"
  },
  "components": {
    "schemas": {
      "FooCore": {
        "description": "This is FooCore",
        "properties": {
          "name": {
            "type": "string"
          }
        },
        "required": [
          "name"
        ],
        "type": "object"
      },
      "Foo": {
        "description": "This is Foo, not FooCore",
        "$ref": "#/components/schemas/FooCore"
      }
    }
  }
}
```