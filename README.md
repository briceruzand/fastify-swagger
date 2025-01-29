# @fastify/swagger

[![NPM version](https://img.shields.io/npm/v/@fastify/swagger.svg?style=flat)](https://www.npmjs.com/package/@fastify/swagger)
[![CI](https://github.com/fastify/fastify-swagger/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/fastify/fastify-swagger/actions/workflows/ci.yml)
[![neostandard javascript style](https://img.shields.io/badge/code_style-neostandard-brightgreen?style=flat)](https://github.com/neostandard/neostandard)

A Fastify plugin for serving [Swagger (OpenAPI v2)](https://swagger.io/specification/v2/) or [OpenAPI v3](https://swagger.io/specification) schemas, which are automatically generated from your route schemas, or an existing Swagger/OpenAPI schema.

If you are looking for a plugin to generate routes from an existing OpenAPI schema, check out [fastify-openapi-glue](https://github.com/seriousme/fastify-openapi-glue).

The following plugins serve Swagger/OpenAPI front-ends based on the swagger definitions generated by this plugin:

- [@fastify/swagger-ui](https://github.com/fastify/fastify-swagger-ui)
- [@scalar/fastify-api-reference](https://github.com/scalar/scalar/tree/main/packages/fastify-api-reference)

See [the migration guide](MIGRATION.md) for migrating from `@fastify/swagger` version `<=7.x` to version `>=8.x`.

<a name="install"></a>
## Install

```
npm i @fastify/swagger
```

### Compatibility

| Plugin version | Fastify version |
| -------------- | --------------- |
| `^9.x`         | `^5.x`          |
| `^8.x`         | `^4.x`          |
| `^7.x`         | `^4.x`          |
| `^6.x`         | `^3.x`          |
| `^3.x`         | `^2.x`          |
| `^1.x`         | `^1.x`          |


Please note that if a Fastify version is out of support, then so are the corresponding versions of this plugin
in the table above.
See [Fastify's LTS policy](https://github.com/fastify/fastify/blob/main/docs/Reference/LTS.md) for more details.



<a name="usage"></a>
## Usage

Add it with `register`, pass it options, call the `swagger` API, and you are done! Below is an example of configuring the OpenAPI v3 specification with Fastify Swagger:

```js
const fastify = require('fastify')()

await fastify.register(require('@fastify/swagger'), {
  openapi: {
    openapi: '3.0.0',
    info: {
      title: 'Test swagger',
      description: 'Testing the Fastify swagger API',
      version: '0.1.0'
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server'
      }
    ],
    tags: [
      { name: 'user', description: 'User related end-points' },
      { name: 'code', description: 'Code related end-points' }
    ],
    components: {
      securitySchemes: {
        apiKey: {
          type: 'apiKey',
          name: 'apiKey',
          in: 'header'
        }
      }
    },
    externalDocs: {
      url: 'https://swagger.io',
      description: 'Find more info here'
    }
  }
})

fastify.put('/some-route/:id', {
  schema: {
    description: 'post some data',
    tags: ['user', 'code'],
    summary: 'qwerty',
    security: [{ apiKey: [] }],
    params: {
      type: 'object',
      properties: {
        id: {
          type: 'string',
          description: 'user id'
        }
      }
    },
    body: {
      type: 'object',
      properties: {
        hello: { type: 'string' },
        obj: {
          type: 'object',
          properties: {
            some: { type: 'string' }
          }
        }
      }
    },
    response: {
      201: {
        description: 'Successful response',
        type: 'object',
        properties: {
          hello: { type: 'string' }
        }
      },
      default: {
        description: 'Default response',
        type: 'object',
        properties: {
          foo: { type: 'string' }
        }
      }
    }
  }
}, (req, reply) => { })

await fastify.ready()
fastify.swagger()
```

<a name="usage.fastify.autoload"></a>
### With `@fastify/autoload`

Register `@fastify/swagger` before routes are loaded with `@fastify/autoload`:

```js
const fastify = require('fastify')()
const fastify = fastify()
await fastify.register(require('@fastify/swagger'))
fastify.register(require("@fastify/autoload"), {
  dir: path.join(__dirname, 'routes')
})
await fastify.ready()
fastify.swagger()
```

<a name="api"></a>
## API

<a name="register.options"></a>
### Register options

<a name="register.options.modes"></a>
#### Modes
`@fastify/swagger` supports `dynamic` and `static` registration modes:

<a name="register.options.mode.dynamic"></a>
##### Dynamic
`dynamic` is the default mode, which auto-generates API schemas from route schemas:
```js
// All of the below parameters are optional but are included for demonstration purposes
{
  // swagger 2.0 options
  swagger: {
    info: {
      title: String,
      description: String,
      version: String
    },
    externalDocs: Object,
    host: String,
    schemes: [ String ],
    consumes: [ String ],
    produces: [ String ],
    tags: [ Object ],
    securityDefinitions: Object
  },
  // openapi 3.0.3 options
  // openapi: {
  //   info: {
  //     title: String,
  //     description: String,
  //     version: String,
  //   },
  //   externalDocs: Object,
  //   servers: [ Object ],
  //   components: Object,
  //   security: [ Object ],
  //   tags: [ Object ]
  // }
}
```

All properties in the [Swagger (OpenAPI v2)](https://swagger.io/specification/v2/) and [OpenAPI v3](https://swagger.io/specification/) specifications can be used.
`@fastify/swagger` generates API schemas adhering to the Swagger specification by default.
Providing an `openapi` option generates OpenAPI compliant API schemas instead.

Examples of using `@fastify/swagger` in `dynamic` mode:
- [Using the `swagger` option](examples/dynamic-swagger.js)
- [Using the `openapi` option](examples/dynamic-openapi.js)

<a name="register.options.mode.static"></a>
##### Static
 `static` mode must be configured explicitly. It serves an existing Swagger or OpenAPI schema passed to `specification.path`:

```js
{
  mode: 'static',
  specification: {
    path: './examples/example-static-specification.yaml',
    postProcessor: function(swaggerObject) {
      return swaggerObject
    },
    baseDir: '/path/to/external/spec/files/location',
  },
}
```

The `specification.postProcessor` parameter is optional and allows modifying the Swagger object on the fly, e.g., based on the environment.
It accepts `swaggerObject` - a JavaScript object parsed from a `yaml` or `json` file and should return a Swagger schema object.

`specification.baseDir` allows specifying the directory where all spec files that are included in the main one using `$ref` will be located.
By default, it is the directory of the main spec file. The value should be an absolute path **without** a trailing slash.

An example of using `@fastify/swagger` with `static` mode enabled can be found [here](examples/static-json-file.js).

#### Options

 | Option           | Default   | Description                                                                                                                   |
 | ---------------- | --------  | ----------------------------------------------------------------------------------------------------------------------------- |
 | hiddenTag        | X-HIDDEN  | Tag to control hiding of routes.                                                                                              |
 | hideUntagged     | false     | If `true` remove routes without tags from resulting Swagger/OpenAPI schema file.                                              |
 | openapi          | {}        | [OpenAPI configuration](https://swagger.io/specification/#oasObject).                                                         |
 | stripBasePath    | true      | Strips base path from routes in docs.                                                                                         |
 | swagger          | {}        | [Swagger configuration](https://swagger.io/specification/v2/#swaggerObject).                                                  |
 | transform        | null      | Transform method for the route's schema and url. [documentation](#register.options.transform).                                |
 | transformObject  | null      | Transform method for the swagger or openapi object before it is rendered. [documentation](#register.options.transformObject). |
 | refResolver      | {}        | Option to manage the `$ref`s of the application's schemas. Read the [`$ref` documentation](#register.options.refResolver)    |
 | exposeHeadRoutes | false     | Include HEAD routes in the definitions                                                                                        |
 | decorator        | 'swagger' | Overrides the Fastify decorator. [documentation](#register.options.decorator).                                                |

<a name="register.options.transform"></a>
#### Transform

Pass a synchronous `transform` function to modify the route's URL and schema.
`openapiObject` and `swaggerObject` are also available.

Some possible uses of this are:
- Adding the `hide` flag to the schema based on URL and schema logic
- Altering the route URL to suit the API spec
- Transforming different schemas (e.g., [Joi](https://github.com/hapijs/joi)) to standard JSON schemas
- Hiding routes based on version constraints

This option is available in `dynamic` mode only.

Examples of all the possible uses mentioned:

```js
const convert = require('joi-to-json')

await fastify.register(require('@fastify/swagger'), {
  swagger: { ... },
  transform: ({ schema, url, route, swaggerObject }) => {
    const {
      params,
      body,
      querystring,
      headers,
      response,
      ...transformedSchema
    } = schema
    let transformedUrl = url

    // Transform the schema as you wish with your own custom logic.
    // In this example convert is from 'joi-to-json' lib and converts a Joi based schema to json schema
    if (params) transformedSchema.params = convert(params)
    if (body) transformedSchema.body = convert(body)
    if (querystring) transformedSchema.querystring = convert(querystring)
    if (headers) transformedSchema.headers = convert(headers)
    if (response) transformedSchema.response = convert(response)

    // can add the hide tag if needed
    if (url.startsWith('/internal')) transformedSchema.hide = true

    // can transform the url
    if (url.startsWith('/latest_version/endpoint')) transformedUrl = url.replace('latest_version', 'v3')

    // can add the hide tag for routes that do not match the swaggerObject version
    if (route?.constraints?.version !== swaggerObject.swagger) transformedSchema.hide = true

    return { schema: transformedSchema, url: transformedUrl }
  }
})
```

The transform function can also be attached to a specific endpoint:

```js
fastify.get("/", {
  schema: { ... },
  config: {
    swaggerTransform: ({ schema, url, route, swaggerObject }) => { ... }
  }
})
```

If both global and local transform functions are available for an endpoint, the endpoint-specific transform function is used.

The local transform function is useful for adding information to a specific endpoint, applying different transformations, or ignoring the global transform function.

The global transform function can be disabled by passing `false` instead of a function.

<a name="register.options.transformObject"></a>
#### Transform Object

By passing a synchronous `transformObject` function you can modify the resulting `swaggerObject` or `openapiObject` before it is rendered.

```js
await fastify.register(require('@fastify/swagger'), {
  swagger: { ... },
  transformObject ({ swaggerObject }) => {
    swaggerObject.info.title = 'Transformed';
    return swaggerObject;
  }
})
```

<a name="register.options.refResolver"></a>
#### Managing your `$ref`s

In `dynamic` mode, this plugin resolves all `$ref`s in the application's schemas, creating a new in-line schema that references itself.
This ensures the generated documentation is valid, preventing Swagger UI from failing to fetch schemas from the server or network.

By default, this option resolves all `$ref`s, renaming them to `def-${counter}`, while view models keep the original `$id` naming using the [`title` parameter](https://swagger.io/docs/specification/2-0/basic-structure/#metadata).

This logic can be customized by passing a `refResolver` option to the plugin:

```js
await fastify.register(require('@fastify/swagger'), {
  swagger: { ... },
  ...
  refResolver: {
    buildLocalReference (json, baseUri, fragment, i) {
      return json.$id || `my-fragment-${i}`
    }
  }
}
```

For details on `buildLocalReference` arguments, see the [documentation](https://github.com/Eomm/json-schema-resolver#usage-resolve-one-schema-against-external-schemas).

<a name="register.options.decorator"></a>
#### Decorator

The default decorate function (`fastify.swagger()`) can be overridden by passing a string to the `decorator` option. This allows creating multiple documents by registering `@fastify/swagger` multiple times with different `transform` functions:

```js
// Create an internal Swagger doc
await fastify.register(require('@fastify/swagger'), {
  swagger: { ... },
  transform: ({ schema, url, route, swaggerObject }) => {
    const {
      params,
      body,
      querystring,
      headers,
      response,
      ...transformedSchema
    } = schema
    let transformedUrl = URL

    // Hide external URLs
    if (url.startsWith('/external')) transformedSchema.hide = true

    return { schema: transformedSchema, url: transformedUrl }
  },
  decorator: 'internalSwagger'
})

// Create an external Swagger doc
await fastify.register(require('@fastify/swagger'), {
  swagger: { ... },
  transform: ({ schema, url, route, swaggerObject }) => {
    const {
      params,
      body,
      querystring,
      headers,
      response,
      ...transformedSchema
    } = schema
    let transformedUrl = URL

    // Hide internal URLs
    if (url.startsWith('/internal')) transformedSchema.hide = true

    return { schema: transformedSchema, url: transformedUrl }
  },
  decorator: 'externalSwagger'
})
```

Then call those decorators individually to retrieve them:
```
fastify.internalSwagger()
fastify.externalSwagger()
```

<a name="route.options"></a>
### Route options

`HEAD` routes can be included in the definitions by adding `exposeHeadRoute` in the route config:

```js
  fastify.get('/with-head', {
    schema: {
      operationId: 'with-head',
      response: {
        200: {
          description: 'Expected Response',
          type: 'object',
          properties: {
            foo: { type: 'string' }
          }
        }
      }
    },
    config: {
      swagger: {
        exposeHeadRoute: true,
      }
    }
  }, () => {})
```

<a name="route.response.options"></a>
#### Response Options

<a name="route.response.description"></a>
##### Response description and response body description
`description` is required by the Swagger specification. If not provided, the plugin defaults to `'Default Response'`.
If a `description` is supplied, it will be used for both the response and response body schema:

```js
fastify.get('/description', {
  schema: {
    response: {
      200: {
        description: 'response and schema description',
        type: 'string'
      }
    }
  }
}, () => {})
```

Generates this in a Swagger (OpenAPI v2) schema's `paths`:

```json
{
  "/description": {
    "get": {
      "responses": {
        "200": {
          "description": "response and schema description",
          "schema": {
            "description": "response and schema description",
            "type": "string"
          }
        }
      }
    }
  }
}
```

And this in an OpenAPI v3 schema's `paths`:

```json
{
  "/description": {
    "get": {
      "responses": {
        "200": {
          "description": "response and schema description",
          "content": {
            "application/json": {
              "schema": {
                "description": "response and schema description",
                "type": "string"
              }
            }
          }
        }
      }
    }
  }
}
```

To provide different descriptions for the response and response body, use the `x-response-description` field alongside `description`:

```js
fastify.get('/responseDescription', {
  schema: {
    response: {
      200: {
        'x-response-description': 'response description',
        description: 'schema description',
        type: 'string'
      }
    }
  }
}, () => {})
```

If a `$ref` is provided in the response schema without a description, the reference's description will be used as a fallback. Currently, `$ref` is resolved by matching with `$id` only, not through complex paths.

<a name="route.response.2xx"></a>
##### Status code 2xx
Fastify supports `2xx` and `3xx` status codes, but Swagger (OpenAPI v2) does not.
`@fastify/swagger` transforms `2xx` into `200`, omitting it if `200` is already declared.
OpenAPI v3 supports [`2xx` syntax](https://swagger.io/specification/#http-codes) so is unaffected.

Example:

```js
{
  response: {
    '2xx': {
      description: '2xx',
      type: 'object'
    }
  }
}

// will become
{
  response: {
    200: {
      schema: {
        description: '2xx',
        type: 'object'
      }
    }
  }
}
```
<a name="route.response.headers"></a>
##### Response headers
Response headers can be decorated with the following example.
Specify the `type` property when decorating response headers to prevent schema modification by Fastify.

```js
{
  response: {
    200: {
      type: 'object',
      headers: {
        'X-Foo': {
          type: 'string'
        }
      }
    }
  }
}
```

<a name="route.response.empty_body"></a>
##### Different content types responses
> 🛈 Note: Supported [only by OpenAPI v3](https://swagger.io/docs/specification/describing-responses/), not Swagger (OpenAPI v2).

Different content types are supported by `@fastify/swagger` and `@fastify`.
Use `content` for the response to prevent Fastify from failing to compile the schema:

```js
{
  response: {
    200: {
      description: 'Description and all status-code based properties are working',
      content: {
        'application/json': {
          schema: {
            name: { type: 'string' },
            image: { type: 'string' },
            address: { type: 'string' }
          }
        },
        'application/vnd.v1+json': {
          schema: {
            fullName: { type: 'string' },
            phone: { type: 'string' }
          }
        }
      }
    }
  }
}
```
##### Empty Body Responses
Empty body responses are supported by `@fastify/swagger`.
Specify `type: 'null'` for the response to prevent Fastify from failing to compile the schema:

```js
{
  response: {
    204: {
      type: 'null',
      description: 'No Content'
    },
    503: {
      type: 'null',
      description: 'Service Unavailable'
    }
  }
}
```

<a name="route.openapi"></a>
#### OpenAPI Parameter Options

> 🛈 Note: OpenAPI's terminology differs from Fastify's. OpenAPI uses "parameter" to refer to parts of a request that in [Fastify's validation documentation](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/) are called "querystring", "params", and "headers".

OpenAPI extends the [JSON schema specification](https://json-schema.org/specification.html) with options like `collectionFormat` for encoding array parameters.

These encoding options only affect how Swagger UI presents documentation and generates `curl` commands.
Depending on the schema options, you may need to change Fastify's default query string parser to produce a JavaScript object conforming to the schema.
The default parser conforms to `collectionFormat: "multi"`.
For `collectionFormat: "csv"`, replace the default parser with one that parses CSV values into arrays. This applies to other request parts that OpenAPI calls "parameters" and are not encoded as JSON.

Different serialization `style` and `explode` can also be applied as specified [here](https://swagger.io/docs/specification/serialization/#query).

`@fastify/swagger` supports these options as shown in this example:

```js
// Need to add a collectionFormat keyword to ajv in fastify instance
const fastify = Fastify({
  ajv: {
    customOptions: {
      keywords: ['collectionFormat']
    }
  }
})

fastify.route({
  method: 'GET',
  url: '/',
  schema: {
    querystring: {
      type: 'object',
      required: ['fields'],
      additionalProperties: false,
      properties: {
        fields: {
          type: 'array',
          items: {
            type: 'string'
          },
          minItems: 1,
          //
          // Note that this is an OpenAPI version 2 configuration option. The
          // options changed in version 3.
          //
          // Put `collectionFormat` on the same property which you are defining
          // as an array of values. (i.e. `collectionFormat` should be a sibling
          // of the `type: "array"` specification.)
          collectionFormat: 'multi'
        }
      },
     // OpenAPI 3 serialization options
     explode: false,
     style: "deepObject"
    }
  },
  handler (request, reply) {
    reply.send(request.query.fields)
  }
})
```

There is a complete runnable example [here](examples/collection-format.js).

<a name="route.complex-serialization"></a>
#### Complex serialization in query and cookie, eg. JSON

> 🛈 Note: Supported [only by OpenAPI v3](https://swagger.io/docs/specification/describing-parameters/#schema-vs-content), not Swagger (OpenAPI v2).

```
http://localhost/?filter={"foo":"baz","bar":"qux"}
```

> 🛈 Note: Change Fastify's default query string parser to produce a JavaScript object conforming to the schema. See [example](examples/json-in-querystring.js).

```js
fastify.route({
  method: 'GET',
  url: '/',
  schema: {
    querystring: {
      type: 'object',
      required: ['filter'],
      additionalProperties: false,
      properties: {
        filter: {
          type: 'object',
          required: ['foo'],
          properties: {
            foo: { type: 'string' },
            bar: { type: 'string' }
          },
          'x-consume': 'application/json'
        }
      }
    }
  },
  handler (request, reply) {
    reply.send(request.query.filter)
  }
})
```

Generates this in the OpenAPI v3 schema's `paths`:

```json
{
  "/": {
    "get": {
      "parameters": [
        {
          "in": "query",
          "name": "filter",
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "required": [
                  "foo"
                ],
                "properties": {
                  "foo": {
                    "type": "string"
                  },
                  "bar": {
                    "type": "string"
                  }
                }
              }
            }
          }
        }
      ]
    }
  }
}
```

##### Route parameters

Route parameters in Fastify are called params.
These are values included in the URL of the requests, for example:

```js
fastify.route({
  method: 'GET',
  url: '/:id',
  schema: {
    params: {
      type: 'object',
      properties: {
        id: {
          type: 'string',
          description: 'user id'
        }
      }
    }
  },
  handler (request, reply) {
    reply.send(request.params.id)
  }
})
```

Generates this in the Swagger (OpenAPI v2) schema's `paths`:

```json
{
  "/{id}": {
    "get": {
      "parameters": [
        {
          "type": "string",
          "description": "user id",
          "required": true,
          "in": "path",
          "name": "id"
        }
      ],
      "responses": {
        "200": {
          "description": "Default Response"
        }
      }
    }
  }
}
```

Generates this in the OpenAPI v3 schema's `paths`:

```json
{
  "/{id}": {
    "get": {
      "parameters": [
        {
          "schema": {
            "type": "string"
          },
          "in": "path",
          "name": "id",
          "required": true,
          "description": "user id"
        }
      ],
      "responses": {
        "200": {
          "description": "Default Response"
        }
      }
    }
  }
}
```

When `params` is not present in the schema, or a schema is not provided, parameters are automatically generated:

```js
fastify.route({
  method: 'POST',
  url: '/:id',
  handler (request, reply) {
    reply.send(request.params.id)
  }
})
```

Generates this in the Swagger (OpenAPI v2) schema's `paths`:

```json
{
  "/{id}": {
    "get": {
      "parameters": [
        {
          "type": "string",
          "required": true,
          "in": "path",
          "name": "id"
        }
      ],
      "responses": {
        "200": {
          "description": "Default Response"
        }
      }
    }
  }
}
```

Generates this in the OpenAPI v3 schema's `paths`:

```json
{
  "/{id}": {
    "get": {
      "parameters": [
        {
          "schema": {
            "type": "string"
          },
          "in": "path",
          "name": "id",
          "required": true
        }
      ],
      "responses": {
        "200": {
          "description": "Default Response"
        }
      }
    }
  }
}
```

<a name="route.links"></a>
#### Links

> 🛈 Note: Supported [only by OpenAPI v3](https://swagger.io/docs/specification/links), not Swagger (OpenAPI v2).

Add OpenAPI v3 Links by adding a `links` property to the top-level options of a route. See:

```js
fastify.get('/user/:id', {
  schema: {
    params: {
      type: 'object',
      properties: {
        id: {
          type: 'string',
          description: 'the user identifier, as userId'
        }
      },
      required: ['id']
    },
    response: {
      200: {
        type: 'object',
        properties: {
          uuid: {
            type: 'string',
            format: 'uuid'
          }
        }
      }
    }
  },
  links: {
    // The status code must match the one in the response
    200: {
      address: {
        // See the OpenAPI documentation
        operationId: 'getUserAddress',
        parameters: {
          id: '$request.path.id'
        }
      }
    }
  }
}, () => {})

fastify.get('/user/:id/address', {
  schema: {
    operationId: 'getUserAddress',
    params: {
      type: 'object',
      properties: {
        id: {
          type: 'string',
          description: 'the user identifier, as userId'
        }
      },
      required: ['id']
    },
    response: {
      200: {
        type: 'string'
      }
    }
  }
}, () => {})
```

<a name="route.hide"></a>
#### Hide a route
There are two ways to hide a route from the Swagger UI:
- Pass `{ hide: true }` to the schema object inside the route declaration.
- Use the tag declared in `hiddenTag` options property inside the route declaration. Default is `X-HIDDEN`.

<a name="function.options"></a>
### Swagger function options

Registering `@fastify/swagger` decorates the fastify instance with `fastify.swagger()`, which returns a JSON object representing the API.
If `{ yaml: true }` is passed to `fastify.swagger()` it returns a YAML string.

<a name="integration"></a>
### Integration

This plugin can be integrated with `@fastify/helmet` with minimal effort:
```js
.register(helmet, instance => {
  return {
    contentSecurityPolicy: {
      directives: {
        ...helmet.contentSecurityPolicy.getDefaultDirectives(),
        "form-action": ["'self'"],
        "img-src": ["'self'", "data:", "validator.swagger.io"],
        "script-src": ["'self'"].concat(instance.swaggerCSP.script),
        "style-src": ["'self'", "https:"].concat(
          instance.swaggerCSP.style
        ),
      }
    }
  }
})
```

<a name="schema.examplesField"></a>
### Add examples to the schema

[OpenAPI](https://swagger.io/specification/#example-object) and [JSON Schema](https://json-schema.org/draft/2020-12/json-schema-validation.html#rfc.section.9.5) have different examples field formats.

Array with examples from JSON Schema converted to OpenAPI `example` or `examples` field automatically with generated names (example1, example2...):

```js
fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    querystring: {
      type: 'object',
      required: ['filter'],
      properties: {
        filter: {
          type: 'object',
          required: ['foo'],
          properties: {
            foo: { type: 'string' },
            bar: { type: 'string' }
          },
          examples: [
            { foo: 'bar', bar: 'baz' },
            { foo: 'foo', bar: 'bar' }
          ]
        }
      },
      examples: [
        { filter: { foo: 'bar', bar: 'baz' } }
      ]
    }
  },
  handler (request, reply) {
    reply.send(request.query.filter)
  }
})
```

Generates this in the OpenAPI v3 schema's `paths`:

```json
"/": {
  "post": {
    "requestBody": {
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "required": ["filter"],
            "properties": {
              "filter": {
                "type": "object",
                "required": ["foo"],
                "properties": {
                  "foo": { "type": "string" },
                  "bar": { "type": "string" }
                },
                "example": { "foo": "bar", "bar": "baz" }
              }
            }
          },
          "examples": {
            "example1": {
              "value": { "filter": { "foo": "bar", "bar": "baz" } }
            },
            "example2": {
              "value": { "filter": { "foo": "foo", "bar": "bar" } }
            }
          }
        }
      },
      "required": true
    },
    "responses": { "200": { "description": "Default Response" } }
  }
}
```

Use the `x-examples` field to set names or add descriptions to schema examples in [OpenAPI format](https://swagger.io/specification/#example-object):

```js
// Need to add a new allowed keyword to ajv in fastify instance
const fastify = Fastify({
  ajv: {
    plugins: [
      function (ajv) {
        ajv.addKeyword({ keyword: 'x-examples' })
      }
    ]
  }
})

fastify.route({
  method: 'POST',
  url: '/feed-animals',
  schema: {
    body: {
      type: 'object',
      required: ['animals'],
      properties: {
        animals: {
          type: 'array',
          items: {
            type: 'string'
          },
          minItems: 1,
        }
      },
      "x-examples": {
        Cats: {
          summary: "Feed cats",
          description:
            "A longer **description** of the options with cats",
          value: {
            animals: ["Tom", "Garfield", "Felix"]
          }
        },
        Dogs: {
          summary: "Feed dogs",
          value: {
            animals: ["Spike", "Odie", "Snoopy"]
          }
        }
      }
    }
  },
  handler (request, reply) {
    reply.send(request.body.animals)
  }
})
```

<a name="usage"></a>
## `$id` and `$ref` usage

### How it works under the hood

`@fastify/static` serves `swagger-ui` static files, then calls `/docs/json` to get the Swagger file and render it.

#### How to work with $refs

The `/docs/json` endpoint in dynamic mode produces a single `swagger.json` file, resolving all of the references.

## Acknowledgments

This project is kindly sponsored by:
- [nearForm](https://nearform.com)
- [LetzDoIt](https://www.letzdoitapp.com/)

<a name="license"></a>
## License

Licensed under [MIT](./LICENSE).
