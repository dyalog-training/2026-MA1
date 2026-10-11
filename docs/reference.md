# Reference card

What the exercises use, on one page.

## Generated clients

### Make a client

A client is made from a **configuration** namespace:

```apl
config←(baseUrl:'https://api.open-meteo.com')
meteo←⎕NEW OpenMeteo.Client config
```

| Configuration | Meaning |
|:--------------|:--------|
| `baseUrl` | Where the service is. Required, or `⎕NEW` signals an error. |
| `security` | Credentials, if the service needs them. For example, `security:(apiKey:'…')`. |
| `mock:1` | Return the HTTP request as text, instead of sending it. |

### Find your way around

Each tag in the specification becomes a namespace in the client, and each operation becomes a function in that namespace:

```apl
client.⎕NL ¯2           ⍝ the tags (and ∆, the client's own settings)
client.quotes.⎕NL ¯3    ⍝ the operations in one tag
)ed client.quotes.CreateQuote
```

The comment header at the top of each operation shows its HTTP method and path, and every parameter: its name, where it goes (path, query or body), whether it is required, and its type.

### Call an operation

Every operation takes **one namespace** of arguments, named as in the header. Use `()` for none:

```apl
r←client.products.ListProducts ()
r←client.quotes.GetQuote (quoteId:id)
r←client.quotes.ReplaceQuote (quoteId:id ⋄ quoteRequest:q)
```

A request body goes in the name that the header gives for it, such as `quoteRequest`.

### Read the result

The result `r` is an HttpCommand result:

| Name | Meaning |
|:-----|:--------|
| `r.rc` | 0 if a request was sent and a reply came back. Anything else means it never got that far: for example, 1106 is a host that can't be found. |
| `r.msg` | What went wrong, when `rc` is not 0 |
| `r.HttpStatus` | The status code of the reply, such as 200 or 404 |
| `r.HttpMessage` | The status text, such as `OK` |
| `r.Data` | The body of the reply, converted from JSON to APL |
| `r.Headers` | The reply's headers |

`r` displays as a one-line summary:

```{ .text .no-copy }
[rc: 0 | msg:  | HTTP Status: 200 "OK" | ≢Data: 1 (namespace)]
```

`r.Data.⎕NL ¯9` lists the namespaces inside the data, and `r.Data.⎕NL ¯2` lists the other values.

### Three ways a call can go wrong

| What happens | What you see | Example |
|:-------------|:-------------|:--------|
| The client refuses, and nothing is sent | An error is signalled, and there is no `r` | A required parameter or the key is missing |
| The server rejects it | A 4xx status, with the reason in `r.Data` | A misspelt value, the wrong key, an unknown id |
| It works, but not as you meant | Status 200, but data is missing | A misspelt parameter name: the client drops names it doesn't know |

Check the status, then check that the data is there.

## HTTP

### Methods

| Method | Meaning | In the exercises |
|:-------|:--------|:-----------------|
| `GET` | Fetch | `GET /v1/forecast`, `GET /products`, `GET /quotes/{id}` |
| `POST` | Create | `POST /quotes` |
| `PUT` | Replace the whole thing | `PUT /quotes/{id}` |
| `DELETE` | Remove | `DELETE /quotes/{id}` |

### Status codes

| Status | Meaning |
|:-------|:--------|
| 200 OK | Done, and here is the result |
| 201 Created | Done, and something new now exists |
| 204 No Content | Done, with nothing to say |
| 400 Bad Request | The request is wrong. The body usually says why. |
| 401 Unauthorized | I don't know who you are: a key is missing or wrong |
| 404 Not Found | There is nothing at that URL |
| 500 Internal Server Error | The server's code failed |
| 501 Not Implemented | The server doesn't do that yet |

## JSON and APL

| JSON | APL |
|:-----|:----|
| `{"id": "NB", "quantity": 20}` | `(id:'NB' ⋄ quantity:20)` |
| `[1, 2, 3]` | `1 2 3` |
| An array of objects | A vector of namespaces: `nb pen` |
| An array of one object | `,⊂nb` or `(nb ⋄)`. On its own, `nb` is a scalar, which becomes an object, not an array. |
| `true`, `false` | `⊂'true'`, `⊂'false'` when you write JSON |

To get a table from a vector of namespaces, pick the fields and mix them:

```apl
↑r.Data.(id name unitPricePence)
```

## Stark

### Serve

```apl
router←Stark.New ()
router.Info←(title: 'Catalogue' ⋄ version: '1.0.0')
'/products/{productId}/price' router.Get ('GetPrice' op)
router.Start 8080
```

| Name | Meaning |
|:-----|:--------|
| `path router.Get ('Fn' op)` | Send `GET` requests for `path` to the function `Fn`, described by the operation `op`. `router.Post`, `router.Put` and `router.Delete` work the same way. |
| `router.Start port` | Start serving. The spec is at `/openapi.json`. |
| `router.Stop` | Stop serving |
| `router.Debug←1` | Stop in the tracer when a handler fails. Set it, then restart the server. |
| `router.Spec` | Extra parts of the spec, such as `components` |
| `router.AddSecurity` | Require a credential, such as an API key |

### Describe an operation

```apl
quantity←(name: 'quantity' ⋄ in: 'query'
    schema: (type: 'integer' ⋄ minimum: 1 ⋄ default: 1))
op←(operationId: 'GetPrice'
    summary: 'Price a quantity of one product'
    tags: ('products' ⋄)
    parameters: productId quantity
    responses: (200 Price)(400 Error)(404 Error))
```

| Name | Meaning |
|:-----|:--------|
| `operationId` | The function's name in a generated client |
| `tags` | The namespace it goes in, in a generated client |
| `parameters` | One namespace for each parameter: `name`, `in` (`'path'` or `'query'`), `required`, `schema` |
| `responses` | Pairs of status code and schema |

### Write a handler

A handler takes the request namespace and returns the body of the response:

| Name | Meaning |
|:-----|:--------|
| `req.PathParams` | Parameters from the path, such as `productId` |
| `req.QueryParams` | Parameters from the query string, as text |
| `req.Payload` | The JSON body of a `POST` or `PUT`, converted to APL |
| `req.Fail 404` | Respond with that status, instead of 200 |

Query parameters arrive as text. To read one as a number, with a default when it's missing:

```apl
n←⊃⊃⌽⎕VFI req.QueryParams ⎕VGET ⊂'quantity' '1'
```

Stark doesn't check the contract for you: the handler checks its input, and picks the status.
