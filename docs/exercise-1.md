# Exercise 1: OpenMeteo

[Open-Meteo](https://open-meteo.com) is a real, public weather forecast service. In this exercise you use a client generated from its OpenAPI specification to:

- get the forecast for where we are sitting
- turn it into APL data
- find out where a call goes wrong

You need the `OpenMeteo` client loaded (see [Setup](setup.md)) and internet access.

## Layer 1: Configure and call

Make a client, then call its one operation with a namespace of arguments:

```apl
config←(baseUrl:'https://api.open-meteo.com')
meteo←⎕NEW OpenMeteo.Client config
args←(latitude:'50.76' ⋄ longitude:'0.29')
args.current←'temperature_2m'
r←meteo.weatherForecastAPIs.GetV1Forecast args
r
```

```{ .text .no-copy }
[rc: 0 | msg:  | HTTP Status: 200 "OK" | ≢Data: 1 (namespace)]
```

!!! note "Not at the workshop?"
    `50.76` and `0.29` are the venue, rounded. You can use your own location instead. The coordinates are strings, so write a negative one with an ordinary minus, `'-74.01'`. If you write the number `¯74.01`, it is sent with APL's high minus and the server rejects it.

!!! example "Your turn"
    The temperature is somewhere in `r`. Find it.

    `r` is the HttpCommand result from Session 1. `r.Data` is a namespace.

??? success "Found it"

    ```apl
    r.Data.⎕NL ¯9
    r.Data.current.temperature_2m
    r.Data.current_units.temperature_2m
    ```

    ```{ .text .no-copy }
     current  current_units
    16.5
    °C
    ```

## Layer 2: The array

!!! example "Your turn · 6 min"
    Get this week's hourly temperatures, as a day × hour matrix. What is each day's high, and when is the warmest hour?

!!! tip "How to look"
    ```apl
    )ed meteo.weatherForecastAPIs.GetV1Forecast
    ```
    lists the parameters. Temperature is `'temperature_2m'`. Times are GMT.

!!! warning "Lost?"
    Re-run `config←` and `meteo←` from [Layer 1](#layer-1-configure-and-call).

??? success "One answer: the data"

    ```apl
    args←(latitude:'50.76' ⋄ longitude:'0.29')
    args.hourly←'temperature_2m' 'precipitation'
    r←meteo.weatherForecastAPIs.GetV1Forecast args
    h←r.Data.hourly
    h.⎕NL ¯2
    ⍴h.temperature_2m
    3↑h.time
    ```

    ```{ .text .no-copy }
     precipitation  temperature_2m  time
    168
     2026-10-06T00:00  2026-10-06T01:00  2026-10-06T02:00
    ```

    168 hours is 7 days of 24. The columns arrive as APL vectors.

??? success "One answer: now it is APL"

    ```apl
    t←7 24⍴h.temperature_2m
    ⌈/t
    +/h.precipitation
    h.time⊃⍨⊃⍒h.temperature_2m
    ```

    ```{ .text .no-copy }
    18.8 17 15.3 18.7 15.8 15 17.4
    28.2
    2026-10-06T14:00
    ```

    Each day's high, the week's rain in mm, and the warmest hour (GMT). Yours will differ.

    The service sends columns. The rest is everyday APL.

## Layer 3: Predict, then try

!!! example "Your turn · 5 min"
    Each of these fails in a different place. For each one, guess first:

    - **(A)** the client refuses, and nothing is sent
    - **(B)** the server rejects it
    - **(C)** you get 200, but not what you meant

    | | Experiment | Change |
    |:--|:--|:--|
    | 1 | Leave something out | No `longitude` |
    | 2 | Misspell a name | `hourley:` instead of `hourly:` |
    | 3 | Misspell a value | `hourly:'temp'` |

!!! tip "How to look"
    Is there always an `r` to look at?

!!! warning "Lost?"
    Re-run `config←` and `meteo←` from [Layer 1](#layer-1-configure-and-call).

??? success "1: Leave something out"

    ```apl
    args←(latitude:'50.76' ⋄ current:'temperature_2m')
    r←meteo.weatherForecastAPIs.GetV1Forecast args
    ```

    ```{ .text .no-copy }
    DOMAIN ERROR: Missing required parameter: longitude
          r←meteo.weatherForecastAPIs.GetV1Forecast args
          ∧
    ```

    **A: the client refuses. Nothing was sent.** The client read the spec, knew `longitude` is required, and signalled an error. There is no `r`.

??? success "2: Misspell a name"

    ```apl
    args←(latitude:'50.76' ⋄ longitude:'0.29')
    args.hourley←'temperature_2m'
    r←meteo.weatherForecastAPIs.GetV1Forecast args
    r.HttpStatus
    r.Data.⎕NL ¯9
    ```

    ```{ .text .no-copy }
    200

    ```

    **C: it worked, and you got nothing.** The client sends only the names it knows, so `hourley` was dropped.

??? success "3: Misspell a value"

    ```apl
    args←(latitude:'50.76' ⋄ longitude:'0.29')
    args.hourly←'temp'
    r←meteo.weatherForecastAPIs.GetV1Forecast args
    r.HttpStatus
    r.Data.reason
    ```

    ```{ .text .no-copy }
    400
    Invalid value: Cannot initialize SurfacePressureAndHeight… from invalid String value temp
    ```

    **B: the server rejects it.** The reason is long, but it ends with the value you sent.

## What you just did

Here is the call from Layer 2, as the HTTP request it sent:

```{ .text .no-copy }
GET /v1/forecast?hourly=temperature_2m%2Cprecipitation&latitude=50.76&longitude=0.29 HTTP/1.1
```

- The operation became `GET /v1/forecast`.
- Your `args` became the query string.
- `r.HttpStatus` is the status line of the reply.

A call can fail in the client, in the server, or silently. Check the status, then check that the data is there.

## Stretch

1. Each day's high and total rain, in local time, as a table. Search `OpenMeteo/forecast.yml` for `name: daily` and `name: timezone`.
2. Eastbourne and London in one call: `latitude` and `longitude` take comma-separated lists. What is `≢r.Data` now?
3. Layer 2 in Fahrenheit. Which parameter, and which values does it allow?
4. Add `mock:1` to a configuration and repeat a call. It shows the request instead of sending it.

??? success "1: Daily table"

    ```apl
    args←(latitude:'50.76' ⋄ longitude:'0.29')
    args.timezone←'auto'
    args.daily←'temperature_2m_max' 'precipitation_sum'
    r←meteo.weatherForecastAPIs.GetV1Forecast args
    r.Data.timezone
    d←r.Data.daily
    ⍉↑d.(time temperature_2m_max precipitation_sum)
    ```

    ```{ .text .no-copy }
    Europe/London
     2026-10-06  18.8  0
     2026-10-07  17   18.5
     2026-10-08  15.3  0
     2026-10-09  18.7  6.9
     2026-10-10  15.8  0
     2026-10-11  15    0
     2026-10-12  17.2  2.6
    ```

    `timezone:'auto'` gives days in local time. Without it, days run from midnight GMT. `⍉↑` turns three columns into a table.

    The spec is the documentation: every parameter, every value.

??? success "2: Two places"

    ```apl
    args←(latitude:'50.76,51.51' ⋄ longitude:'0.29,-0.13')
    args.current←'temperature_2m'
    r←meteo.weatherForecastAPIs.GetV1Forecast args
    ≢r.Data
    r.Data.current.temperature_2m
    ```

    ```{ .text .no-copy }
    2
    12.2 11.3
    ```

    `r.Data` is now a vector of two namespaces, one for each place. The coordinates are strings, so London's negative longitude is written `-0.13`, not `¯0.13`.

??? success "3: Fahrenheit"

    `forecast.yml` lists `temperature_unit` with the values `celsius` (the default) and `fahrenheit`:

    ```apl
    args←(latitude:'50.76' ⋄ longitude:'0.29')
    args.hourly←'temperature_2m'
    args.temperature_unit←'fahrenheit'
    r←meteo.weatherForecastAPIs.GetV1Forecast args
    r.Data.hourly_units.temperature_2m
    ⌈/7 24⍴r.Data.hourly.temperature_2m
    ```

    ```{ .text .no-copy }
    °F
    58.6 64.3 67.3 64.4 65.5 64.9 58.1
    ```

??? success "4: Mock"

    ```apl
    mockcfg←(baseUrl:'https://api.open-meteo.com' ⋄ mock:1)
    mock←⎕NEW OpenMeteo.Client mockcfg
    args←(latitude:'50.76' ⋄ longitude:'0.29')
    args.hourly←'temperature_2m' 'precipitation'
    mock.weatherForecastAPIs.GetV1Forecast args
    ```

    ```{ .text .no-copy }
    GET /v1/forecast?hourly=temperature_2m%2Cprecipitation&latitude=50.76&longitude=0.29 HTTP/1.1
    User-Agent: Dyalog-HttpCommand/5.11.1 Dyalog OpenAPI Generated Client/1.0
    Host: api.open-meteo.com
    Accept: */*
    Accept-Encoding: gzip, deflate
    ```
