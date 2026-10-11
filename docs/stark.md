# Stark walkthrough

[Stark](https://github.com/Dyalog-labs/Stark) is a routing layer on top of Jarvis. You register routes (an HTTP method and a path) that point at your APL functions. Stark generates an OpenAPI spec from your routes and serves it at `/openapi.json`.

This walkthrough builds one endpoint, contract first. You need Stark loaded (see [Setup](setup.md#load-stark)).

## Step 1: Agree the contract

Decide what the endpoint promises before you write any code.

```{ .text .no-copy }
GET /products/{productId}/price?quantity=20
```

| Parameter | In | Meaning |
|:----------|:---|:--------|
| `productId` | path, required | A product id, for example `NB` |
| `quantity` | query, optional | A whole number, 1 or more. The default is 1. |

| Status | Body | When |
|:-------|:-----|:-----|
| 200 | `{ productId, quantity, totalPence }` | |
| 400 | `{ message }` | `quantity` is not a whole number, 1 or more |
| 404 | `{ message }` | There is no such product |

The data is the catalogue from Exercise 2:

```apl
Catalogue←(
    (id: 'NB'  ⋄ name: 'Notebook' ⋄ unitPricePence: 400)
    (id: 'PEN' ⋄ name: 'Pen'      ⋄ unitPricePence: 200)
)
```

## Step 2: Describe the parameters

Use one namespace for each parameter:

```apl
productId←(name: 'productId' ⋄ in: 'path' ⋄ required: ⊂'true'
    description: 'A product id, e.g. NB'
    schema: (type: 'string'))

quantity←(name: 'quantity' ⋄ in: 'query'
    description: 'How many to price'
    schema: (type: 'integer' ⋄ minimum: 1 ⋄ default: 1))
```

A parameter says where it goes and what type it is. `⊂'true'` is how `⎕JSON` writes `true`. Path parameters are always required. Query parameters are optional unless you say so.

## Step 3: Describe the responses

Describe what comes back, as JSON Schema:

```apl
Price←(type: 'object' ⋄ properties: (
    productId:  (type: 'string')
    quantity:   (type: 'integer')
    totalPence: (type: 'integer')
))
Error←(type: 'object' ⋄ properties: (message: (type: 'string')))
```

Then put the whole operation together:

```apl
op←(operationId: 'GetPrice'
    summary: 'Price a quantity of one product'
    tags: ('products' ⋄)
    parameters: productId quantity
    responses: (200 Price)(400 Error)(404 Error))
```

`operationId` names the function in a generated client. A client generated from this spec has `client.products.GetPrice`.

## Step 4: Publish the contract

```apl
∇ r←GetPrice req     ⍝ not written yet
  req.Fail 501
  r←(message: 'Not implemented yet')
∇

router←Stark.New ()
router.Info←(title: 'Catalogue' ⋄ version: '1.0.0')
'/products/{productId}/price' router.Get ('GetPrice' op)
router.Start 8080
```

The spec is live before the code is. Open <http://localhost:8080/openapi.json>: it is already complete. A call returns 501 Not Implemented, but anyone can generate a client now.

!!! warning "Stop the server when you finish"
    The server keeps running in the background while you work in the session. It listens on your network as well as on `localhost`, so stop it when you finish:

    ```apl
    router.Stop
    ```

    `router.Start 8080` starts it again with the same routes. If port 8080 is already in use, start it on another port, such as 8081, and use that port in the URLs.

## Step 5: Write the function

```apl
∇ r←GetPrice req;i;n
  i←Catalogue.id⍳⊂req.PathParams.productId
  n←⊃⊃⌽⎕VFI req.QueryParams ⎕VGET ⊂'quantity' '1'
  :If i>≢Catalogue
      req.Fail 404 ⋄ r←(message: 'No such product')
  :ElseIf (n<1)∨n≠⌊n
      req.Fail 400 ⋄ r←(message: 'quantity must be a whole number, 1 or more')
  :Else
      r←(productId: i⊃Catalogue.id ⋄ quantity: n)
      r.totalPence←n×i⊃Catalogue.unitPricePence
  :EndIf
∇
```

Stark doesn't check the contract for you: the handler validates `quantity` and picks the status. The server is still running, so redefining `GetPrice` is enough.

## Step 6: Check it against the contract

Try every response in the contract:

=== "Browser"

    - <http://localhost:8080/products/NB/price?quantity=20>
    - <http://localhost:8080/products/PEN/price>
    - <http://localhost:8080/products/NB/price?quantity=0>
    - <http://localhost:8080/products/XYZ/price>

=== "curl"

    On Windows, type `curl.exe` instead of `curl`, and use double quotes instead of single quotes.

    ```bash
    curl 'localhost:8080/products/NB/price?quantity=20'
    curl localhost:8080/products/PEN/price
    curl 'localhost:8080/products/NB/price?quantity=0'
    curl localhost:8080/products/XYZ/price
    ```

```{ .text .no-copy }
{"productId":"NB","quantity":20,"totalPence":8000}         200
{"productId":"PEN","quantity":1,"totalPence":200}          200
{"message":"quantity must be a whole number, 1 or more"}   400
{"message":"No such product"}                              404
```

Each line shows the response body, then its status. A browser shows only the body.

The spec didn't change when the code did: `/openapi.json` is exactly what step 4 published.

## Find out more

Stark's documentation covers each part of this walkthrough in more detail:

- [Getting started](https://dyalog-labs.github.io/stark/getting-started/)
- [The router](https://dyalog-labs.github.io/stark/stark-router/): routes, the request namespace, and debugging
- [OpenAPI](https://dyalog-labs.github.io/stark/openapi/): how the spec is built from your routes
- [Authentication](https://dyalog-labs.github.io/stark/authentication/)
- [Examples](https://dyalog-labs.github.io/stark/examples/)
