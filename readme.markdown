# TypeScript OpenAPI DSL

> Build OpenAPI3 contracts declaratively.

## Install

```sh
npm i ts-openapi-dsl
```

## Usage

### Example

```ts
spec.paths = paths({
  getUser: operation({
    method: 'get',
    path: '/users/:id',
    responses: {
      200: json({
        schema: array(string()),
      }),
    },
  }),
});

// same as
spec.paths = {
  '/users/:id': {
    'get': {
      operationId: 'getUser',
      responses: {
        '200': {
          content: {
            'application/json': {
              schema: {
                type: 'array',
                items: {
                  type: 'string',
                },
              },
            },
          },
        },
      },
    },
  },
};
```

### Schema helpers

Schema helpers can be used to remove _some_ boilerplate around JSON schema
definitions. Every helper returns a JSON schema object.

```ts
import { types as t } from 'ts-openapi-dsl';

const User = t.object('An object that represents a user', {
  id: t.integer('The ID of the user'),
  name: t.string('The name of the user'),
  createdAt: t.dateTime(),
  foo: {
    type: 'string',
    description: 'You can use raw JSON schema as well...',
  },
});
```

Every helper accepts an `extra` parameter that can be used to decorate the schema with additional properties.

#### `boolean(description?: string, extra?: Schema): Schema`

Defines a boolean schema, essentially:

```ts
const type = {
  type: 'boolean',
  description,
  ...extra,
};
```

#### `integer(description?: string, extra?: Schema): Schema`

Defines an integer schema, essentially:

```ts
const type = {
  type: 'integer',
  description,
  ...extra,
};
```

#### `string(description?: string, extra?: Schema): Schema`

Defines a string schema, essentially:

```ts
const type = {
  type: 'string',
  description,
  ...extra,
};
```

#### `pattern(pattern: RegExp, description?: string, extra?: Schema): Schema`

Defines a string schema with a regex pattern.

#### `date(description?: string, extra?: Schema): Schema`

Defines a string schema with `date` format.

#### `dateTime(description?: string, extra?: Schema): Schema`

Defines a string schema with `date-time` format.

#### `binary(description?: string, extra?: Schema): Schema`

Defines a string schema with `binary` format.

#### `email(description?: string, extra?: Schema): Schema`

Defines a string schema with `email` format.

#### `constant(value: string, description?: string, extra?: Schema): Schema`

Defines a string schema with an `enum` that only accept a single value, essentially:

```ts
const type = {
  type: 'string',
  description,
  enum: [value],
  ...extra,
};
```

#### `choice(values: Record<string, string>, extra?: Schema): Schema`

Defines a string schema with an `enum` that only accepts predefined values. Keys of `values` are the allowed values, and the corresponding values in the object are the descriptions.

```ts
const type = choice({
  FOO: 'Description of FOO',
  BAR: 'Description of BAR',
});

// same as
cons type = {
  type: 'string',
  enum: ['FOO', 'BAR'],
  description: '* `FOO` - Description of FOO\n* `BAR` - Description of BAR',
};
```

#### `array(items: Schema, description?: string, extra?: Schema): Schema`

Defines an array schema with the specified `items`, essentially:

```ts
const type = {
  type: 'array',
  description,
  items,
  ...extra,
};
```

#### `object(properties: Record<string, Schema>, description?: string, extra?: Schema): Schema`

Defines an object schema with the specified properties.

Every helpers has an `optional` variant, eg. `string.optional(...)`, which signals to its parent object to make it an optional property.

Objects are strict by default (ie. `additionalProperties` is set to `false`).

```ts
const type = object({
  foo: string(),
  bar: string.optional(),
});

// same as
const type = {
  type: 'object',
  additionalProperties: false,
  required: ['foo'],
  properties: {
    foo: {
      type: 'string',
    },
    bar: {
      type: 'string',
    },
  },
};
```

#### `required(schema: Schema): Schema`

Turns the given schema into a required schema (ie. in a wrapper `object`, it'll
be a `required` property).

```ts
const wrapped = string.optional();

const wrapper = object({
  foo: wrapped,
  bar: required(wrapped),
});

// same as
const wrapper = {
  type: 'object',
  additionalProperties: false,
  required: ['bar'],
  properties: {
    foo: { type: 'string' },
    bar: { type: 'string' },
  },
};
```

#### `optional(schema: Schema): Schema`

Turns the given schema into an optional schema (ie. in a wrapper `object`, it
won't be a `required` property).

```ts
const wrapped = string.required();

const wrapper = object({
  foo: wrapped,
  bar: optional(wrapped),
});

// same as
const wrapper = {
  type: 'object',
  additionalProperties: false,
  required: ['foo'],
  properties: {
    foo: { type: 'string' },
    bar: { type: 'string' },
  },
};
```

### OpenAPI helpers

#### `response(props: Response): ResponseObject`

Defines a response object with a single media type:

```ts
const res = response({
  description: 'Response description',
  headers,
  links,
  type: 'application/json',
  schema,
  examples,
  encoding,
});

// same as
const res = {
  description: 'Response description',
  headers,
  links,
  content: {
    'application/json': {
      schema,
      examples,
      encoding,
    },
  },
};
```

#### `json(props: Response): ResponseObject`

Shorthand for creating `application/json` responses.

```ts
const res = json({
  description: 'Response description',
  headers,
  links,
  schema,
  examples,
  encoding,
});

// same as
const res = {
  description: 'Response description',
  headers,
  links,
  content: {
    'application/json': {
      schema,
      examples,
      encoding,
    },
  },
};
```

#### `operation(op: Operation): OperationObject`

Defines an operation with `path` and `method`.

#### `paths(ops: Record<string, Operation>): PathsObject`

Creates `paths` based on the provided `ops`.

## License

MIT
