# Web Services in APL

**An Introduction to REST and OpenAPI**: Dyalog '26 workshop MA1.

## Before you start

You need **Dyalog APL 20.0 or later** and the workshop materials, [cloned or downloaded](https://github.com/dyalog-training/2026-MA1). Then follow [Setup](setup.md) to load the clients.

## The workshop

| Session | Topic | Exercise |
|:--------|:------|:---------|
| 1. Basics | Web services, REST and HttpCommand | |
| *Break* | | |
| 2. Consuming | OpenAPI and generated clients | [Exercise 1: OpenMeteo](exercise-1.md), [Exercise 2: Quotations](exercise-2.md) |
| *Break* | | |
| 3. Serving | Jarvis and Stark | [Stark walkthrough](stark.md), [Exercise 3: Your own API](exercise-3.md) |

## How the exercises work

The exercises are built in **layers**. Each layer is either code to type in, or a **Your turn** task with a hint on where to look. In each layer:

- Code blocks with a copy button (top right) are for your session.
- Blocks without a copy button show what your session should print. When the output depends on the weather or on what other people have done, yours will be different.
- Each task is followed by its answer, collapsed until you click it. Try the task before you open it.
- **Lost?** tells you which lines to run again if your session has lost track.

Lines that start with `⍝` are comments. You don't need to type them.

## Learning objectives

- Understand how REST uses URLs, HTTP methods and status codes
- Call a web service from APL using a client generated from its OpenAPI specification
- Read a response to tell success from failure
- Expose APL functions as a web service with an OpenAPI specification

## Resources

The [Reference card](reference.md) sums up what the exercises use, on one page.

### The tools

- The client generator: [dyalog-labs.github.io/OpenAPI/](https://dyalog-labs.github.io/OpenAPI/)
- Stark: [dyalog-labs.github.io/Stark](https://dyalog-labs.github.io/stark)
- HttpCommand: [dyalog.github.io/HttpCommand](https://dyalog.github.io/HttpCommand)
- Jarvis: [dyalog.github.io/Jarvis](https://dyalog.github.io/Jarvis)

### Dyalog APL

- [Array notation](https://dyalog.github.io/documentation/20.0/programming-reference-guide/introduction/arrays/array-notation/), for writing namespaces such as `(latitude:'50.76' ⋄ longitude:'0.29')`
- [`⎕JSON`](https://dyalog.github.io/documentation/20.0/language-reference-guide/system-functions/json/), which converts between JSON and APL arrays

### OpenAPI and HTTP

- [Learn OpenAPI](https://learn.openapis.org): an introduction to writing specifications
- [OpenAPI 3.0.3](https://spec.openapis.org/oas/v3.0.3): the version of the specification that Stark generates
- [Swagger Editor](https://editor.swagger.io): paste in a specification to read it as documentation
- [HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status), on MDN

### The services in the exercises

- [Open-Meteo API documentation](https://open-meteo.com/en/docs): every weather variable, with a URL builder
- `OpenMeteo/forecast.yml` and `Quotations/openapi.json` in the workshop folder: the specifications the clients were generated from
- `Quotes/` in the workshop folder: the Exercise 2 service, written with Stark, for reading
