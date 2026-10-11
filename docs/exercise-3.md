# Exercise 3: Your own API

You need Stark loaded (see [Setup](setup.md#load-stark)). The [Stark walkthrough](stark.md) shows every step on a worked example.

!!! example "Your turn · 25 min"
    Design a small API of your own, contract first, and serve it with Stark.

    1. Write the contract: path, parameters, responses.
    2. Register it with a stub handler. Read `/openapi.json`.
    3. Write the handler. Bad input gets a 4xx status.
    4. Try every response in the browser.

!!! tip "Ideas"
    - `GET /hello/{name}`
    - `GET /primes?limit=30`
    - `POST /stats` with a list of numbers
    - A function you already have

!!! info "Handy to know"
    - A handler gets the request as its argument: `req.PathParams` and `req.QueryParams` hold the parameters, and `req.Fail 400` sets an error status.
    - For a `POST`, register the route with `router.Post`. The JSON body arrives in the handler as `req.Payload`, already converted to APL.
    - A browser can only send `GET` requests. To send a `POST`, use the HttpCommand that came with either client:

        ```apl
        r←Quotations.HttpCommand.GetJSON 'POST' 'http://localhost:8080/stats' (3 1 4 1 5)
        r.(HttpStatus Data)
        ```

!!! tip "Debugging a handler"
    By default, an error in a handler becomes a 500 response. To stop in the tracer instead, restart the server with `Debug` set:

    ```apl
    router.Stop
    router.Debug←1
    router.Start 8080
    ```

    Then send the request **from your browser**: a call from the same APL session would be stuck waiting for the reply. Set `router.Debug←0` and restart to go back to normal.

??? success "One answer"

    This adds `GET /primes` to the router from the walkthrough, and reuses its `Error` schema. Start with the contract:

    ```apl
    limit←(name: 'limit' ⋄ in: 'query' ⋄ required: ⊂'true'
        schema: (type: 'integer' ⋄ minimum: 2))
    Primes←(type: 'object' ⋄ properties: (
        primes: (type: 'array' ⋄ items: (type: 'integer'))))
    primes←(operationId: 'GetPrimes' ⋄ parameters: (limit ⋄)
        responses: (200 Primes)(400 Error))
    '/primes' router.Get ('GetPrimes' primes)
    ```

    Then write the handler:

    ```apl
    ∇ r←GetPrimes req;n
      n←⊃⊃⌽⎕VFI req.QueryParams ⎕VGET ⊂'limit' ''
      :If (n<2)∨n≠⌊n
          req.Fail 400 ⋄ r←(message: 'limit must be a whole number, 2 or more')
      :Else
          r←(primes: (2=+⌿0=∘.|⍨⍳n)/⍳n)
      :EndIf
    ∇
    ```

    Try <http://localhost:8080/primes?limit=30>:

    ```{ .text .no-copy }
    {"primes":[2,3,5,7,11,13,17,19,23,29]}
    ```

## Stretch

Each of these starts with a change to the contract.

**Reuse a schema.** Move `Price` and `Error` into `router.Spec.components`, and point to them with `$ref`. The Quotes service shows how: see `Quotes/SPEC`.

**Require a key.** `router.AddSecurity` is the server side of Exercise 2. The Quotes service has one: see `Quotes/AUTH`.

**Read your spec as docs.** Paste the contents of `/openapi.json` into [editor.swagger.io](https://editor.swagger.io).
