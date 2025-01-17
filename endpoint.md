# Endpoint API

```ts
// new endpoint
endpoint()
// path or function to match it
// if nothing matches: invoke /404
// if /404 does not exist, return error code 400 and quit
  .get("/api/blog")
// parse the body (try JSON.parse if error pt by default)
// if this fails the error code is 400
  .parse(JSON.parse)
// req contains request information (will define later)
// res contains functions for creating responses
// each response contains the info relevant 
// functions for adding more info
// or replacing/modifying new data
  .respond((req: { headers; body }, res) =>
// returning the request converts the body and headers and other info etc etc and then responds to the client
    res
      .headers({
        contentType: "application/json",
      })
      .body({
        ...body,
      })
  );
```

okay i cba to write it but basically:  
call `endpoint()` to make an endpoint  
then call `.get` `.post` or `.delete` with the path as an arg
then if u want call `.parse` and pass in a function u can  
it takes in a string as an argument and returns whatever  
and the output is then passed to the `.respond` part  
which takes in a request with headers and body and info  
and a res with utility functions for requests  (`.headers,` .body, `.status`)
