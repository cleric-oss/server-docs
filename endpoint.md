# Endpoint Adapter API

## Interface types

```ts
// static class
interface Endpoint {
  // segments with a : at the start are path parameters
  // invalid paths throw an error
  get(path: string): EmptyEndpoint;
  post(path: string): EmptyEndpoint;
  delete(path: string): EmptyEndpoint;
}

// endpoints which are being created
interface EmptyEndpoint {
  // not parsed, not validated, and not responded
  parse(fn: (body: string) => Object): UnvalidatedEndpoint;
  validate<T>(
    // JSON.parse returns an Object
    // params are the segments with a colon
    // ie path `/c/:character/` would have the parameter `character`
    fn: (obj: Object, params: Record<string, string>) => T
  ): UnrespondedEndpoint<T>;
  respond(
    fn: (
      req: Readonly<{
        headers: Headers;
        // JSON.parse returns an Object
        body: Object;
        params: Record<string, string>;
      }>,
      res: NetworkResponse
    ) => NetworkResponse
  ): RespondableEndpoint;
}

interface UnvalidatedEndpoint {
  // parsed but not validated or responded
  validate<T>(
    // parsed value could be anything
    fn: (obj: any, params: Record<string, string>) => T
  ): UnrespondedEndpoint<T>;
  respond(
    fn: (
      req: {
        headers: Headers;
        // parsed value could be anything
        body: any;
        params: Record<string, string>;
      },
      res: NetworkResponse
    ) => NetworkResponse
  ): RespondableEndpoint;
}

interface UnrespondedEndpoint<T> {
  // parsed and validated
  respond(
    fn: (
      req: {
        headers: Headers;
        // validated value is now T
        body: T;
      },
      res: NetworkResponse
    ) => NetworkResponse
  ): RespondableEndpoint;
}

interface RespondableEndpoint {
  // change how the response body is returned to the client
  stringify(fn: (body: any) => string): void;
}

// responses sent to client
interface NetworkResponse {
  head: Headers;
  body: any;
}
```

## Static Endpoint Methods

### Get

Creates a new get request:

```ts
Endpoint.get(path: string): EmptyEndpoint;
```

### Post

Creates a new post request:

```ts
Endpoint.post(path: string): EmptyEndpoint;
```

### Delete

Creates a new delete request:

```ts
Endpoint.delete(path: string): EmptyEndpoint;
```

## Endpoint Methods

### Parse

Default parse method is `JSON.parse`.  
Define a function to convert the body to an object.

```ts
parse(fn: (body: string) => Object): UnvalidatedEndpoint;
```

### Validate

By default: no validation method.  
Define a function to ensure the body is valid. If the body is malformed, throw an error. The message will be returned tho the client like so:

```json
{
  "status": "error",
  "message": "<Error.message>"
}
```

The return value of this function is passed to the `respond` function, and the return types are also copied over. If a validation method is ommited, the body will be returned after parsing with a type of `Object` (if no custom parse method is defined), or `any` (if a custom parse method is defined)

```ts
validate(fn: (obj: Object) => any): UnrespondedEndpoint<ReturnType<typeof fn>>;
validate(fn: (obj: any) => any): UnrespondedEndpoint<ReturnType<typeof fn>>;
```

### Respond

Required.  
Define a function to be called whenever responding to the client. The function should return a `NetworkResponse` object. If no parse or validation functions are defined, the body is of type `Object`. If a parse function is defined but no validation method defined, the body is of type `any`. If a validation method is defined, the body is of the type returned by the validation function.

```ts
respond(
  fn: (
    req: {
      headers: Headers;
      body: any;
    },
    res: NetworkResponse
  ) => NetworkResponse
): void;
```

### Stringify

Default stringify method is `JSON.stringify`.  
Define a function to convert the body to a string.

```ts
stringify(fn: (body: any) => string): void;
```

## NetworkResponse Properties

### head

The headers to be sent to the client. Defaults to `new Headers()`.

```ts
head: Headers = new Headers();
```

### body

The body to be sent to the client. Defaults to `{}`.
Can be of any type.

```ts
body: any = {};
```
