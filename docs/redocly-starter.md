# Redocly starter guide

Moved out of the README. This is generic Redocly starter documentation; `AGENTS.md`, `package.json`,
and `redocly.yaml` are authoritative for this repo.

## How to use this starter

Go to:

- https://app.cloud.redocly.com/
- https://app.cloud.redocly.com/org/dboeckli/overview

![Click use template button](https://user-images.githubusercontent.com/3975738/92927304-12e35d80-f446-11ea-9bd3-a0f8a69792d0.png)

## Working on your OpenAPI definition

### Install

1. Install [Node.js](https://nodejs.org/).
2. Clone this repo and run `npm ci` in the repo root (`npm install` only when changing dependencies).
3. Redocly CLI is installed as a dependency; a global install is not required.

### Usage

- `npm test` validates the definition with `redocly lint`.
- `npm run build` bundles the definition to `dist/bundle.yaml`.
- `npm start` is broken with Redocly CLI 2.x because `preview-docs` was removed. Render static docs
  with `npx redocly build-docs openapi/openapi.yaml -o out.html`.

### Schemas

#### Adding schemas

1. Navigate to the `openapi/components/schemas` folder.
2. Add a file named as you wish to name the schema.
3. Define the schema.
4. Refer to the schema using `$ref` (see example below).

##### Example schema

This is a very simple schema example:

```yaml
type: string
description: The resource ID. Defaults to UUID v4
maxLength: 50
example: 4f6cf35x-2c4y-483z-a0a9-158621f77a21
```

This is a more complex schema example:

```yaml
type: object
properties:
  id:
    description: The customer identifier string
    readOnly: true
    allOf:
      - $ref: ./ResourceId.yaml
  websiteId:
    description: The website's ID
    allOf:
      - $ref: ./ResourceId.yaml
  paymentToken:
    type: string
    writeOnly: true
    description: |
      A write-only payment token; if supplied, it will be converted into a
      payment instrument and be set as the `defaultPaymentInstrument`. The
      value of this property will override the `defaultPaymentInstrument`
      in the case that both are supplied. The token may only be used once
      before it is expired.
  defaultPaymentInstrument:
    $ref: ./PaymentInstrument.yaml
  createdTime:
    description: The customer created time
    allOf:
      - $ref: ./ServerTimestamp.yaml
  updatedTime:
    description: The customer updated time
    allOf:
      - $ref: ./ServerTimestamp.yaml
  tags:
    description: A list of customer's tags
    readOnly: true
    type: array
    items:
      $ref: ./Tags/Tag.yaml
  revision:
    description: >
      The number of times the customer data has been modified.

      The revision is useful when analyzing webhook data to determine if the
      change takes precedence over the current representation.
    type: integer
    readOnly: true
  _links:
    type: array
    description: The links related to resource
    readOnly: true
    minItems: 3
    items:
      anyOf:
        - $ref: ./Links/SelfLink.yaml
        - $ref: ./Links/NotesLink.yaml
        - $ref: ./Links/DefaultPaymentInstrumentLink.yaml
        - $ref: ./Links/LeadSourceLink.yaml
        - $ref: ./Links/WebsiteLink.yaml
  _embedded:
    type: array
    description: >-
      Any embedded objects available that are requested by the `expand`
      querystring parameter.
    readOnly: true
    minItems: 1
    items:
      anyOf:
        - $ref: ./Embeds/LeadSourceEmbed.yaml
```

If you have a JSON example, convert it to a JSON schema with Redocly's
[JSON to JSON schema tool](https://redocly.com/tools/json-to-json-schema/).

##### Using `$ref`

The schema definition itself can use `$ref` links to other schemas:

```yaml
defaultPaymentInstrument:
  $ref: ./PaymentInstrument.yaml
```

The value of `$ref` is the path to the other schema definition. Use `$ref`s to compose schemas from
existing schemas and to reference schemas from path definitions.

#### Editing schemas

1. Navigate to the `openapi/components/schemas` folder.
2. Open the file you wish to edit.
3. Edit it.

### Paths

#### Adding a path

1. Navigate to the `openapi/paths` folder.
2. Add a YAML file named like the URL endpoint, replacing `/` with `_` (or another separator) and
   keeping path parameters in curly braces such as `{example}`.
3. Add the path and a `$ref` to it in `openapi/openapi.yaml`.

Example addition to `openapi/openapi.yaml`:

```yaml
'/customers/{id}':
  $ref: './paths/customers_{id}.yaml'
```

Example path file `customers_{id}.yaml`:

```yaml
get:
  tags:
    - Customers
  summary: Retrieve a list of customers
  operationId: GetCustomerCollection
  description: |
    You can have a markdown description here.
  parameters:
    - $ref: ../components/parameters/collectionLimit.yaml
    - $ref: ../components/parameters/collectionOffset.yaml
    - $ref: ../components/parameters/collectionFilter.yaml
    - $ref: ../components/parameters/collectionQuery.yaml
    - $ref: ../components/parameters/collectionExpand.yaml
    - $ref: ../components/parameters/collectionFields.yaml
  responses:
    '200':
      description: A list of Customers was retrieved successfully
      headers:
        Rate-Limit-Limit:
          $ref: ../components/headers/Rate-Limit-Limit.yaml
        Rate-Limit-Remaining:
          $ref: ../components/headers/Rate-Limit-Remaining.yaml
        Rate-Limit-Reset:
          $ref: ../components/headers/Rate-Limit-Reset.yaml
        Pagination-Total:
          $ref: ../components/headers/Pagination-Total.yaml
        Pagination-Limit:
          $ref: ../components/headers/Pagination-Limit.yaml
        Pagination-Offset:
          $ref: ../components/headers/Pagination-Offset.yaml
      content:
        application/json:
          schema:
            type: array
            items:
              $ref: ../components/schemas/Customer.yaml
        text/csv:
          schema:
            type: array
            items:
              $ref: ../components/schemas/Customer.yaml
    '401':
      $ref: ../components/responses/AccessForbidden.yaml
  x-code-samples:
    - lang: PHP
      source:
        $ref: ../code_samples/PHP/customers/get.php
post:
  tags:
    - Customers
  summary: Create a customer (without an ID)
  operationId: PostCustomer
  description: Another markdown description here.
  requestBody:
    $ref: ../components/requestBodies/Customer.yaml
  responses:
    '201':
      $ref: ../components/responses/Customer.yaml
    '401':
      $ref: ../components/responses/AccessForbidden.yaml
    '409':
      $ref: ../components/responses/Conflict.yaml
    '422':
      $ref: ../components/responses/InvalidDataError.yaml
  x-code-samples:
    - lang: PHP
      source:
        $ref: ../code_samples/PHP/customers/post.php
```

This example uses `$ref`s to different component types, including schemas and code samples.

### Code samples

Automated code sample generation is configured in `redocly.yaml`. Add manual code samples as follows:

1. Navigate to the `openapi/code_samples` folder.
2. Navigate to the `<language>` (for example PHP) subfolder.
3. Navigate to the path folder and add the reference to the code sample.

Add languages by creating new folders at the appropriate path level. More details are in the
`code_samples` folder README (if present).
