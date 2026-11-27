---
icon: percent
---

# PlaceholderAPI

Install PlaceholderAPI to enable Sertraline's expansion. Sertraline registers the `%sertraline_...%` expansion automatically when PlaceholderAPI is present.

## Syntax and testing

Use `%sertraline_<prefix>:<parameter>%`. For example, test a random integer in game with:

```text
/papi parse me %sertraline_random:40to80%
```

The prefix selects the operation; the text after `:` is its input.

## Supported prefixes

| Prefix | Purpose |
| --- | --- |
| `kether`, `ke` | Run Kether with `player` and Sertraline's default script utilities in the context. When the PAPI player is online, it is also used as the Kether sender. |
| `javascript`, `js` | Run JavaScript with `player` and the default script utilities. Compiled scripts are cached. |
| `jexl`, `je` | Run JEXL with `player` and the default script utilities. Compiled scripts are cached. |
| `graaljs`, `gjs` | Evaluate GraalJS with `player` and the default script utilities. |
| `fluxon`, `fl` | Run Fluxon with `player` and the default script utilities. |
| `config` | Read a raw value from Sertraline's configuration by key, such as `config:debug`. |
| `random`, `rand` | Generate a random number using `starttoend`, for example `40to80` or `1.5to3.5`; an optional `:2` sets decimal places. |
| `data` | Read a player's raw stored value by key. |
| `data1`, `data2` | Read numeric player data rounded to one or two decimal places. |
| `data1a`, `data2a` | Same numeric formatting, but remove a trailing `.0`. |
| `cdleftmil` | Read remaining cooldown time in milliseconds for a key. |
| `cdleftsec`, `cdleftsec1`, `cdleftsec2` | Read remaining cooldown time in seconds, rounded to an integer, one decimal, or two decimals. |
| `cdleftsec1a`, `cdleftsec2a` | One- or two-decimal cooldown values with a trailing `.0` removed. |

Several value-returning prefixes support a fallback after `?:`, for example `%sertraline_data:my_key?:unset%`. Use a fallback with the prefixes that return one value; it is not a general script-language feature.

{% hint style="warning" %}
Script prefixes execute code with access to Sertraline's script utilities and the current player context. Use only scripts you trust, and do not assume a given PlaceholderAPI call runs on the server thread.
{% endhint %}
