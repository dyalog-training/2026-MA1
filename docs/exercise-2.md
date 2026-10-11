# Exercise 2: Quotations

You make a quotation for office supplies:

- prepare it from the product catalogue
- revise it when the requirements change
- withdraw it when it is no longer needed

You need the `Quotations` client loaded (see [Setup](setup.md)).

- **The service** is at `https://d26-ma1.gos.dyalog.com`.
- **The API key** is one key for the whole room, given in the workshop. Wherever you see `'<API key>'` below, put the key in its place.
- **Your reference** is your first name, in lower case. The examples use `'sam'`: use your own name instead.

## Layer 1: Configure and list

```apl
config←(
    baseUrl:'https://d26-ma1.gos.dyalog.com'
    security:(apiKey:'<API key>')
)
client←⎕NEW Quotations.Client config
r←client.products.ListProducts ()
r.HttpStatus
```

```{ .text .no-copy }
200
```

One configuration authenticates every call. The key travels in a header on every request, and you set it only once.

```apl
↑r.Data.(id name unitPricePence)
```

```{ .text .no-copy }
 NB   Notebook  400
 PEN  Pen       200
 FLD  Folder    300
```

A collection is a vector of namespaces. Pick the fields you want and mix them with `↑`. Prices are in pence, and IDs are case-sensitive.

### Predict, then try

!!! example "Your turn · 2 min"
    Make one client with the wrong key, and one with no key at all. Call `ListProducts` with each. Which one reaches the server?

!!! tip "How to look"
    Build each one like `config`, changing only `security`. For no key, leave `security` out.

!!! warning "Lost?"
    Re-run Layer 1 from `config←`.

??? success "The wrong key"

    ```apl
    badcfg←(baseUrl:'https://d26-ma1.gos.dyalog.com')
    badcfg.security←(apiKey:'WRONG_KEY')
    bad←⎕NEW Quotations.Client badcfg
    r←bad.products.ListProducts ()
    r.HttpStatus
    r.Data.message
    ```

    ```{ .text .no-copy }
    401
    A valid X-API-Key header is required.
    ```

    **The wrong key reaches the server, which rejects it.** 401 means "I do not know you". The body says how to fix it.

??? success "No key at all"

    ```apl
    nocfg←(baseUrl:'https://d26-ma1.gos.dyalog.com')
    nokey←⎕NEW Quotations.Client nocfg
    r←nokey.products.ListProducts ()
    ```

    ```{ .text .no-copy }
    DOMAIN ERROR: No valid/supported security scheme available
          r←nokey.products.ListProducts ()
          ∧
    ```

    **No key: the client refuses, and nothing is sent.** The spec says every operation needs a key, so the client checks before sending.

## Layer 2: Create and fetch

!!! example "Your turn · 7 min"
    Get the service to price 20 notebooks and 20 pens, with a reference that is unique to you, such as your name. That's this JSON, in APL:

    ```json
    { "reference": "sam",
      "items": [ { "productId": "NB",  "quantity": 20 },
                 { "productId": "PEN", "quantity": 20 } ] }
    ```

    Here is one item, in APL:

    ```apl
    nb←(productId:'NB' ⋄ quantity:20)
    ```

    Then fetch the quotation back by its id.

!!! tip "How to look"
    `client.quotes.⎕NL ¯3`, then the header of `CreateQuote`. Every operation takes one namespace. The body goes in the name that the header checks for.

!!! warning "Lost?"
    Re-run Layer 1 from `config←`.

??? success "One answer: create"

    ```apl
    nb←(productId:'NB' ⋄ quantity:20)
    pen←(productId:'PEN' ⋄ quantity:20)
    q←(reference:'sam' ⋄ items:nb pen)
    r←client.quotes.CreateQuote (quoteRequest:q)
    r.HttpStatus
    r.Data.totalPence
    id←r.Data.id
    ```

    ```{ .text .no-copy }
    201
    12000
    ```

    The server did the pricing and saved the result. 201 means created. `id` names your quotation from now on.

??? success "One answer: fetch it back"

    ```apl
    r←client.quotes.GetQuote (quoteId:id)
    r.HttpStatus
    ↑r.Data.items.(name quantity linePence)
    ```

    ```{ .text .no-copy }
    200
     Notebook  20 8000
     Pen       20 4000
    ```

    The result of one call is the input to the next. The id goes into the URL: `GET /quotes/{id}`.

## Layer 3: Revise

!!! example "Your turn · 6 min"
    The customer now wants 25 notebooks and no pens. Revise your quotation in place: same id, new total.

!!! tip "How to look"
    `ReplaceQuote` replaces the whole quotation. If it is refused, `r.Data.message` says why.

!!! note "Done?"
    Check with `GetQuote` that the id is unchanged.

!!! warning "Lost?"
    Re-run Layer 2 from `nb←` to `id←`.

??? failure "If you set the pens to zero"

    ```apl
    nb.quantity←25
    pen.quantity←0
    q←(reference:'sam' ⋄ items:nb pen)
    r←client.quotes.ReplaceQuote (quoteId:id ⋄ quoteRequest:q)
    r.HttpStatus
    r.Data.message
    ```

    ```{ .text .no-copy }
    400
    Quantity must be a positive integer; omit the item to remove it from the quotation.
    ```

    A rejected request changes nothing. The message says how to fix it: leave the pens out.

??? failure "If you sent one item"

    ```apl
    q←(reference:'sam' ⋄ items:nb)
    r←client.quotes.ReplaceQuote (quoteId:id ⋄ quoteRequest:q)
    r.HttpStatus
    r.Data.message
    ```

    ```{ .text .no-copy }
    400
    items must be an array, even for one item; an object was received.
    ```

    One namespace is a scalar. `⎕JSON` turns a scalar namespace into a JSON object, not a list of one.

??? success "One answer: a list of one"

    ```apl
    nb.quantity←25
    q←(reference:'sam' ⋄ items:,⊂nb)
    r←client.quotes.ReplaceQuote (quoteId:id ⋄ quoteRequest:q)
    r.HttpStatus
    r.Data.totalPence
    ```

    ```{ .text .no-copy }
    200
    10000
    ```

    PUT replaces the whole quotation. Array notation also writes a list of one: `items:(nb ⋄)`.

## The whole room

!!! example "Your turn · 2 min"
    Everyone has made a quotation. Show them all, as a table of id, reference and total.

!!! tip "How to look"
    `client.quotes.⎕NL ¯3`. A collection is a vector of namespaces, as in Layer 1.

!!! warning "Lost?"
    Re-run Layer 1 from `config←`.

??? success "One answer"

    ```apl
    r←client.quotes.ListQuotes ()
    ↑r.Data.(id reference totalPence)
    ```

    ```{ .text .no-copy }
    1  kim  10000
    2  lee  10000
    3  sam  10000
    …
    ```

    This is everyone's quotations, right now: one shared service holds every quotation in the room. Find yours by name.

## Layer 4: Withdraw

!!! example "Your turn · 2 min"
    The deal is off. Withdraw your quotation, then prove that the service no longer has it.

!!! tip "How to look"
    `client.quotes.⎕NL ¯3`. Withdrawing takes the same argument as `GetQuote`.

!!! note "Done?"
    List the quotations again.

!!! warning "Lost?"
    `id` is the number in your row of the list. Set it with `id←` and that number.

??? success "One answer"

    ```apl
    r←client.quotes.DeleteQuote (quoteId:id)
    r.HttpStatus
    r←client.quotes.GetQuote (quoteId:id)
    r.HttpStatus
    r.Data.message
    ```

    ```{ .text .no-copy }
    204
    404
    No quotation with id 3.
    ```

    A well-formed request can refer to something that is gone. 204 means done, with nothing to say. 404 means there is nothing at that URL now.

## What you just did

| Request | Status |
|:--------|:-------|
| `GET /products` | 200 |
| `POST /quotes` | 201 |
| `GET /quotes/{id}` | 200 |
| `PUT /quotes/{id}` | 400, then 200 |
| `GET /quotes` | 200 |
| `DELETE /quotes/{id}` | 204 |
| `GET /quotes/{id}` | 404 |
| any, with the wrong key | 401 |

**Authentication without authorisation.** Which calls changed what the server holds? And everyone used one key: the server knew you were allowed in, but not who you were.

## Stretch

1. Make a second quotation, for 5 folders. It is one item: remember Layer 3.
2. Retrieve someone else's quotation by its id. Nothing stopped you. What would the server need to know to prevent it? (Read only, please.)
3. List only your own quotations. `Quotations/openapi.json` documents a parameter of `GET /quotes` for it.

??? success "1: Folders"

    ```apl
    fld←(productId:'FLD' ⋄ quantity:5)
    q←(reference:'sam' ⋄ items:,⊂fld)
    r←client.quotes.CreateQuote (quoteRequest:q)
    r.Data.totalPence
    ```

    ```{ .text .no-copy }
    1500
    ```

    One item, so it needs the enclose. 5 × 300 = 1500.

??? success "3: Only your own"

    The header of `ListQuotes` shows an optional `reference` query parameter:

    ```apl
    r←client.quotes.ListQuotes (reference:'sam')
    ↑r.Data.(id reference totalPence)
    ```
