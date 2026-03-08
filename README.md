# OpenAPI Schema Test Suite

This repository contains a set of JSON objects that implementers of OpenAPI Schema Object dialect validation libraries can use to test their validators.
The test suite repository exists to verify specified behavior defined by the OpenAPI Schema Object dialects and should not be confused with a style guide. It is not intended to demonstrate how specifications ought to be written. Tests may appear unusual or unintuitive, but they exist solely to exercise behavior prescribed by the specification.

It is meant to be language agnostic and should require only a JSON parser. The conversion of the JSON objects into tests within a specific language and test framework of choice is left to be done by the validator implementer.

## Coverage

This suite covers the following OpenAPI Schema Object dialects:

- **OAS 3.0**: Highly customized subset/superset of JSON Schema Draft 4
- **OAS 3.1**: [`https://spec.openapis.org/oas/3.1/dialect/2024-11-10`](https://spec.openapis.org/oas/3.1/dialect/2024-11-10) (also compatible with [`https://spec.openapis.org/oas/3.1/dialect/base`](https://spec.openapis.org/oas/3.1/dialect/base))
- **OAS 3.2**: [`https://spec.openapis.org/oas/3.2/schema/2025-11-23`](https://spec.openapis.org/oas/3.2/schema/2025-11-23)

## Introduction to the Test Suite Structure

The tests in this suite are contained in the `tests` directory at the root of this repository. Inside that directory is a subdirectory for each supported version of the OpenAPI specification:

- `tests/oas30/` — Tests for the OpenAPI 3.0 Schema Object dialect
- `tests/oas31/` — Tests for the OpenAPI 3.1 Schema Object dialect
- `tests/oas32/` — Tests for the OpenAPI 3.2 Schema Object dialect

Inside each version directory there are a number of `.json` files each containing a collection of related tests. The grouping is generally by keyword or feature under test.

Each `.json` file consists of a single JSON array of test cases. The format of each test case is described below and formally defined in [`test-schema.json`](test-schema.json).

### Subdirectories Within Each Version Directory

Each version directory may contain an `optional/` subdirectory:

- `optional/format/` — Contains tests for `format` validation when treated as assertions (not annotations). These tests apply only to validators that are configured to enable format assertion validation.

### Sample Test Case

Here is a single test case containing one or more tests:

```json
{
    "description": "The test case description",
    "schema": {
        "$schema": "https://spec.openapis.org/oas/3.1/dialect/2024-11-10",
        "type": "string"
    },
    "tests": [
        {
            "description": "a test with a valid instance",
            "data": "a string",
            "valid": true
        },
        {
            "description": "a test with an invalid instance",
            "data": 15,
            "valid": false
        }
    ]
}
```

### Terminology

| Term | Definition |
|------|------------|
| **test suite** | the entirety of the contents of this repository, containing tests for multiple different OpenAPI Schema Object dialect versions |
| **test case** | a single schema, along with a description and an array of _tests_ |
| **test** | within a _test case_, a single test example, containing a description, instance and a boolean indicating whether the instance is valid under the test case schema |
| **test runner** | a program, external to this repository and authored by a user of this suite, which implements the logic required to execute each of the tests in the suite |

## Key Behaviors Tested

### OAS 3.1 / OAS 3.2 (both based on JSON Schema 2020-12)

- **Type validation** (`type.json`): Basic type checks including `integer`, `number`, `string`, `boolean`, `null`, `object`, `array`, and arrays of types.
- **Nullable types** (`nullable.json`): Nullable schemas using `type: ["string", "null"]` (the OAS 3.1+ way, replacing the `nullable: true` keyword from OAS 3.0).
- **Properties** (`properties.json`): Object properties, `required`, `additionalProperties`, `minProperties`, `maxProperties`, `patternProperties`, `propertyNames`.
- **Items** (`items.json`): Array items, `prefixItems`, `minItems`, `maxItems`, `uniqueItems`, `contains`, `minContains`, `maxContains`.
- **allOf** (`allOf.json`): Schema composition with `allOf`.
- **anyOf** (`anyOf.json`): Schema composition with `anyOf`.
- **oneOf** (`oneOf.json`): Schema composition with `oneOf`.
- **not** (`not.json`): Schema negation with `not`.
- **$ref** (`ref.json`): Schema references using `$ref` and `$defs`.
- **discriminator** (`discriminator.json`): The OpenAPI `discriminator` keyword (annotation for polymorphic schema selection).
- **enum and const** (`enum.json`): Enumeration and constant value validation.
- **Numeric constraints** (`numeric.json`): `minimum`, `maximum`, `exclusiveMinimum`, `exclusiveMaximum`, `multipleOf`.
- **String constraints** (`string.json`): `minLength`, `maxLength`, `pattern`.
- **Format annotations** (`format.json`): `format` as annotations only (the default behavior in the OAS 3.1/3.2 base dialect).
- **Unevaluated keywords** (`unevaluated.json`): `unevaluatedProperties` and `unevaluatedItems`.

### Optional Tests

- **Format assertions** (`optional/format/format-assertion.json`): Tests for `format` validation when treated as assertions. Validators enabling format assertion support should pass these tests.

## Using the Suite to Test a Validator Implementation

To test a specific version, walk the filesystem tree for that version's subdirectory (`tests/oas31/` or `tests/oas32/`) and for each `.json` file found:

1. Parse the JSON array of test cases.
2. For each test case:
   - Load the schema from the `"schema"` property. The `"$schema"` field within the schema indicates the dialect.
   - For each test in the `"tests"` array:
     - Validate the value in `"data"` against the schema.
     - If the validation result matches the `"valid"` boolean, the test passes.
     - Otherwise, the test fails.

For tests in `optional/format/`, configure your validator to treat `format` as assertions before running them.

## Similar Projects

This test suite is inspired by and similar to the [JSON Schema Test Suite](https://github.com/json-schema-org/JSON-Schema-Test-Suite).
