# API Definitions

This defines the format in which data should be returned by endpoints, and the format in which data is stored in the official cleric client. Whenever a url contains a `:` at the start of the segment, it is a path parameter. Additionally, all URLs should be prefixed when called or defined to prevent collision with urls on servers where a client is bundled. Whenever a `*` is included in the URL it should match to `.*` in a regex, and is not passed to the path parameter.  
For example: `/c/:character-id/` would match `/api/c/lorem/`, passing the path parameter `{"character-id": "lorem"}` to the endpoint resolver. 

Client requests should always include the following headers:

- `cleric-client-id`: The unique ID of the client
- `cleric-client-version`: The version of the client
- `cleric-client-features`: The features supported by the client as a list of values seperated by a comma.
- `cleric-client-auth`: The auth key for the client. If no auth key is avaliable to the client, the value should be `null`.

See [client.md](./client.md) for more information.

## ToC

- [Character](./character.md)
