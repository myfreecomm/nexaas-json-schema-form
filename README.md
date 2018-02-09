# Nexaas JSON Schema Form

This is a "subschema" for building HTML forms out of a [JSON Schema](http://json-schema.org/), while there is not an [JSON Schema UI vocabilary](https://github.com/json-schema-org/json-schema-vocabularies/issues/2) defined yet.

We must use it for implementing JSON Schema forms on our apps.

The JSON Schema Form is only an extension of JSON Schema. So, the form for omitted properties must be implemented based on JSON schema.

## JSON Schema extensions

We defined some extensions on top of vanilla JSON Schema:

### format

Some extra possible values for [format](http://json-schema.org/latest/json-schema-validation.html#rfc.section.7) as [annotations](http://json-schema.org/latest/json-schema-validation.html#annotations):

- currency

## Examples

Given the following JSON Schema:

``` json
{
  "title": "Payment",
  "type": "object",
  "properties": {
    "amount": {
      "title": "Valor",
      "type": "number",
      "default": "7.11"
    },
    "date": {
      "title": "Data do Pagamento",
      "type": "string",
      "format": "date"
    },
    "due_date": {
      "title": "Data de Vencimento",
      "type": "string",
      "format": "date",
      "default": "2017-12-18"
    },
    "payer_document": {
      "title": "Documento do Pagador",
      "type": "string"
    },
    "payer_document_type": {
      "title": "CPF ou CNPJ?",
      "type": "string",
      "enum": [
        "cpf",
        "cnpj"
      ]
    }
  },
  "required": [
    "date",
    "amount",
    "due_date",
    "payer_document",
    "payer_document_type"
  ]
}
```

And this JSON Schema Form:

``` json
{
  "postal_code": {
    "mask": "99999-999",
    "placeholder": "99999-999"
  },
  "payee_document_type": {
    "tag": "select",
    "options": {
      "cpf": "CPF",
      "cnpj": "CNPJ"
    }
  },
  "payee_document": {
    "dependency": {
      "key": "payee_document_type",
      "enum": {
        "cnpj": {
          "mask": "99.999.999/9999-99"
        },
        "cpf": {
          "mask": "999.999.999-99"
        }
      }
    }
  },
}
```

They are separated schemas and is recommended that your API implements like this:

``` json
{
  "schema": {
    /* your json schema goes here */
  },
  "form": {
    /* your json form schema goes here */
  }
}
```

## Keywords

### tag

Define the form element to be used for the property representation. If not defined, must use `"text"` as default. Possible values are:

- text
- select
- radio

### mask

Define a mask to be applied to the form. It must follows [jQuery-Mask-Plugin](http://igorescobar.github.io/jQuery-Mask-Plugin/) pattern or similar.

### placeholder

Define a placeholder to be applied to the form.

### options

Define the options for a select though an object `"value": "label"`. If not defined, must implement value and label as the property's `enum` keyword.

### dependency

Defines a dependency for change one property's form dynamically based on other property value.

#### key

The name of the property that, when the value changes, the attributes must be applied.

#### enum

An object `"value": {attributes}` where `attributes` is an `"attribute": "val"` to be applied to the property, when dependency value is equal to `"value"`. On our example, when `"payee_document_type"` value is `"cpf"`, the `"payee_document"` mask will be `"99.999.999/9999-99"`.

## References

- http://schemaform.io
- https://github.com/mozilla-services/react-jsonschema-form
- http://jsonforms.io

## Extra info

### enum + options vs oneOf

An alternative for the `enum` + `options` for valitate and present the possibles values for a property is the use of `oneOf` keyword:

``` json
/** Using enum + options **/

{
  "schema": {
    "payer_document_type": {
      "title": "CPF ou CNPJ?",
      "type": "string",
      "enum": [
        "cpf",
        "cnpj"
      ]
    }
  },
  "form": {
    "payee_document_type": {
      "tag": "select",
      "options": {
        "cpf": "CPF",
        "cnpj": "CNPJ"
      }
    }
  }
}

/** Using oneOf **/

{
  "schema": {
    "payer_document_type": {
      "title": "CPF ou CNPJ?",
      "type": "string",
      "oneOf": [
        {
          "const": "cpf",
          "title": "CPF"
        },
        {
          "const": "cnpj",
          "title": "CNPJ"
        }
      ]
    }
  },
    "form": {
    "payee_document_type": {
      "tag": "select"
    }
  }
}
```

We prefer to use the enum + options for keep separated the validation from the UI.

More info:

- https://github.com/json-schema-org/json-schema-spec/issues/57#issuecomment-247861695
- http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.7.3
