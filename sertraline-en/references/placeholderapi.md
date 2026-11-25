---
icon: percent
---

# PlaceholderAPI

### Usage

{% hint style="success" %}
Sertraline scripts used through PlaceholderAPI can access the script utilities available in their context.
{% endhint %}

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
You can use `/papi parse me %sertraline_PREFIX:PARAM%` to test Placeholders.
{% endhint %}

### PlaceHolders

| PAPI Parameter Prefix      | Description                                       | Notes                                                                                      |
| -------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| kether: / ke:              | Execute Kether script                             | Root variables include player data, Player passed as CommandSender, executes kether script |
| javascript: / js:          | Execute JavaScript script                         | Root variables include player data, Includes script compilation and caching                |
| jexl: / je:                | Execute JEXL script                               | Root variables include player data, Compiled and cached via prodJexlCompiler               |
| fluxon: / fl:              | Execute Fluxon script                             | Root variables include player data                                                         |
| graaljs: / gjs: (`3.6.3+`) | Execute GraalJS script                            | Runs the script. Use a placeholder that returns a value when you need display text                                        |
| config:                    | Read value from configuration by key              | Directly returns config entry                                                              |
| random: / rand:            | Generate random number (supports ints and floats) | Parameter format example: `40to80`                                                     |
| data: (`3.6.3+`)           | Get raw player data                               | Uses DataUtil to get raw data                                                              |
| data1: / data2: | Read numeric player data | Rounds to one or two decimal places |
| data1a: / data2a: | Read numeric player data | Also trims the trailing ".0" |
| cdleftmil: (`3.6.3+`)      | Get cooldown left in milliseconds                 | Uses DataUtil to get cooldown time in milliseconds                                         |
| cdleftsec: (`3.6.3+`)      | Get cooldown left in seconds (rounded integer)    | Converts ms to seconds, rounded                                                            |
| cdleftsec1: (`3.6.3+`)     | Get cooldown left in seconds (1 decimal place)    | Converts ms to seconds, rounded to 1 decimal place                                         |
| cdleftsec2: (`3.6.3+`)     | Get cooldown left in seconds (2 decimal places)   | Converts ms to seconds, rounded to 2 decimal places                                        |
| cdleftsec1a: (`3.6.3+`)    | Variant of cdleftsec1, trims trailing ".0"        | One decimal place with trailing ".0" removed                                               |
| cdleftsec2a: (`3.6.3+`)    | Variant of cdleftsec2, trims trailing ".0"        | Two decimal places with trailing ".0" removed                                              |

In game, run `/papi parse me %sertraline_random:40to80%` to verify the expansion. `random:1.5to3.5:2` rounds to at most two decimal places and removes trailing zeros. Some placeholders accept a fallback, such as `%sertraline_data:my_key?:unset%`.
