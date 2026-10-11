# Setup

You need **Dyalog APL 20.0 or later**:

```apl
]version
```

The first line of the output should show 20.0 or later:

```{ .text .no-copy }
 Dyalog  20.0.…
```

## Get the workshop folder

[Clone or download dyalog-training/2026-MA1](https://github.com/dyalog-training/2026-MA1) (**Code**, then **Download ZIP**, then unpack it). It contains:

| Folder | Contents |
|:-------|:---------|
| `OpenMeteo/` | The client for [Exercise 1](exercise-1.md) |
| `Quotations/` | The client for [Exercise 2](exercise-2.md) |
| `Stark/` | Stark and Jarvis, for [Exercise 3](exercise-3.md) |
| `Quotes/` | The Exercise 2 service, for reference |

## Load both clients

Do this before Session 2. Replace `/path/to` with the folder you put the workshop folder in (forward slashes work on every platform):

```apl
]LINK.Import OpenMeteo /path/to/OpenMeteo/APLSource
]LINK.Import Quotations /path/to/Quotations/APLSource
OpenMeteo.Version
Quotations.Version
```

```{ .text .no-copy }
 Open-Meteo Weather Forecast API  1.0  2026-09-29
 Quotations  1.0.0  2026-09-29
```

`OpenMeteo` and `Quotations` must end up in separate namespaces. Problems? Tell an assistant during the break.

## Load Stark

Do this before Exercise 3. Stark looks for Jarvis beside itself, so import the folder into `#`:

```apl
]LINK.Import # /path/to/Stark
```
