INPUT FORMAT:
Card pool lines use:

`{count} {Card Name} {setCode}-{cardNumber}#{fields}`

Parse rule:

* `{count}` is the owned quantity for that exact print.
* `{Card Name}` is the printed/display name used for deck export.
* `{setCode}-{cardNumber}` is the exact print identifier.
* `{canonicalName}` is the card name used for legality and same-name copy limits.
* `{fields}` contains compact card data used for deckbuilding decisions.
* Card legality and the 4-copy limit are based on `{canonicalName}`.
* Exact ownership availability is based on `{setCode}-{cardNumber}`.
* Final deck export must use `{Card Name}` and `{setCode}-{cardNumber}`.

FIELD FORMAT:
Fields appear after the first `|`.

Top-level field syntax:

* Top-level fields are separated by `|`.
* Key/value pairs use `=`.
* Lists use `[item,item,item]`.
* Objects use `{key=value,key=value}`.
* A field value may be a raw string, list, or object.
* `\` escapes reserved characters when needed.
* Reserved characters are: `\ | = , [ ] { }`

FIELD KEYS:
Use these keys to understand card function:

* `s` = subtypes or classification.

  * For Pokémon, this may contain Basic, Stage 1, Stage 2, ex, MEGA, Tera, etc.
  * For Trainers, this may contain Item, Supporter, Stadium, Pokémon Tool, etc.
  * For Energy, this may contain Basic or Special.
  * If `s` is a single value, treat it as one subtype.
  * If `s` is a list, treat every listed value as a subtype.

* `h` = HP.

  * Numeric HP value.
  * Mainly relevant for Pokémon and Fossil-like cards that can be played as Pokémon.

* `t` = type.

  * For Pokémon, this is the Pokémon type or types.
  * For nested objects, `t` may also mean the object type, such as Ability.
  * Interpret by context.

* `ef` = evolves from.

  * The lower-stage card this card evolves from.
  * Use this to build evolution lines and check whether the needed Basic or Stage 1 exists.

* `et` = evolves to.

  * The higher-stage card or cards this card can evolve into.
  * Use this to identify possible evolution paths.
  * This is useful but not required for legality if `ef` already defines the line.

* `r` = rules text.

  * Used for Trainer effects, Energy effects, Pokémon rule boxes, restrictions, Fossil rules, Tool rules, Tera Bench protection, copy limits, and other special card text.
  * Any deckbuilding restriction in `r` must be obeyed.

* `ab` = ability or abilities.

  * May be a single object or a list of objects.
  * Use abilities to identify setup engines, draw engines, acceleration, protection, switching, disruption, recovery, and passive effects.

* `at` = attack or attacks.

  * May be a single object or a list of objects.
  * Use attacks to identify starters, bridge attackers, main attackers, backup attackers, Energy costs, damage output, discard costs, status effects, and tempo.

COMMON NESTED KEYS:
Nested objects inside `ab` and `at` may use:

* `n` = name.

  * Name of the attack, Ability, or nested effect.

* `x` = text.

  * Effect text of the attack, Ability, rule, or nested effect.
  * This is often the most important field for judging card function.

* `c` = cost.

  * Usually attack cost.
  * Treat listed Energy symbols as the cost required to use the attack.
  * `Colorless` can be paid by any Energy.
  * `Free` means no Energy cost.

* `d` = damage.

  * Printed attack damage.
  * May include symbols such as `+`, `-`, or `×`.
  * Use the attack text `x` to understand conditional damage.

* `t` = nested type.

  * Usually identifies the nested object type, such as Ability.
  * Do not confuse this with top-level Pokémon type.

* `v` = value.

  * Generic nested value field.
  * Use only when relevant and interpretable from context.

FIELD INTERPRETATION RULES:

* Use actual card text from `r`, `ab`, and `at` over generic assumptions.
* If a field is missing, do not invent it.
* If a card has no `ab`, assume it has no listed Ability.
* If a card has no `at`, assume it has no listed attack.
* If a value is malformed or incomplete, use only what is clear and judge conservatively.
* If `at` or `ab` is a list, evaluate every listed attack or Ability.
* If `at` or `ab` is a single object, treat it as one attack or Ability.
* Use `ef` and `et` to identify evolution support, but do not include unsupported evolution pieces unless the deck can actually use them.
* Use `r` to detect ACE SPEC, Special Energy restrictions, Tool behavior, Fossil behavior, Tera protection, “can’t attack” clauses, “can’t use more than 1” clauses, and other deckbuilding constraints.

FINAL DECK FORMAT:
Every final decklist line must use:

`{count} {Card Name} {setCode}-{cardNumber}`

Basic Energy must also be written as normal decklist lines:

`{count} Basic {EnergyType} Energy {energyCode}`

Basic Energy codes:
Grass=sve-1
Fire=sve-2
Water=sve-3
Lightning=sve-4
Psychic=sve-5
Fighting=sve-6
Darkness=sve-7
Metal=sve-8

Do not output JSON card objects.
Do not output comments inside the decklist block.
Do not output alternate decklist formats.
Do not include example cards in the instructions.

API SIMULATION / ANALYSIS:
After building an exact 60-card candidate deck, make a request to:

`POST https://hailxd.vercel.app/api/ptcg-player/llm-analysis`

Headers:

`Content-Type: application/json`

Request body shape:

`{"name":"JY {Deck Name}","deckText":"{exact 60-card decklist joined by newline characters}"}`

Request body rules:

* `{name}` must be the generated deck name.
* `{deckText}` must contain the exact 60-card final decklist.
* Each deckText line must use: `{count} {Card Name} {setCode}-{cardNumber}`
* Basic Energy must be included as normal deckText lines.
* deckText must not contain comments.
* deckText must not contain markdown.
* deckText must not contain JSON card objects.
* deckText must not contain explanatory text.
* deckText must not contain example or placeholder cards.

Successful response shape:

`{"ok":true,"deckId":"{deckId}","imported":{boolean},"opponents":["{opponentId}"],"text":"{analysisText}"}`

Use only `{analysisText}` for compact analysis unless `{deckId}` or opponent IDs are specifically needed.

The response text format contains:

* `deck=`
* `imported=`
* `games_per_opponent=`
* `fmt:`
* repeated opponent summary lines
* repeated compact game log lines

API USE RULE:
Use the API result before finalizing the deck.

Evaluate:

* winrate spread across opponents
* repeated failure patterns in game logs
* bad openers
* slow setup
* Energy misses
* weak prize race
* poor bridge plan
* attacker sustainability
* whether the main plan actually works

If the API result shows the deck is weak or inconsistent, revise the deck and retest if practical.
Do not blindly obey one API result over deckbuilding fundamentals, but use it as required playtest feedback.
If the API request cannot be made, continue with internal reliability testing and state that API testing was unavailable.

PRINT SELECTION:
When choosing prints, output selected prints as:

`{Card Name} {setCode}-{cardNumber}`

Never exceed owned quantity per exact print.
Never exceed 4 copies by canonical card NAME across all prints.

If multiple prints of the same canonical NAME exist:

1. Prefer the newest set based on pool order.
2. Within a set, prefer lower card number first.
3. Never exceed owned print quantity.
4. Stop once the desired count is reached.

FINAL OUTPUT TEMPLATE:

### JY {Deck Name}

```text id="m9x6yf"
{count} {Card Name} {setCode}-{cardNumber}
{count} Basic {EnergyType} Energy {energyCode}
```

### Rating

Difficulty: X/10
Strength: X/10

### Strategy

{2-sentence plan summary, why this build was chosen, game plan, ideal opening, midgame, late game, biggest weakness, upgrade priorities, pool-limited status}

### How to use

{step-by-step coaching: benching, evolving, Energy commitment, draw/search Supporter timing, holding resources, transition from bridge to main attacker, common mistakes + fixes}

### Build checks

* Opening starters:
* Setup outs:
* Pivot outs:
* Earliest realistic pressure:
* Main payoff timing:
* Main attacker:
* Bridge plan:
* Backup plan:
* Biggest consistency risk:
* API test:

  * Deck submitted as:
  * Opponents tested:
  * Winrate trend:
  * Main log findings:
  * Revisions made from API result:
  * Final average-draw reliability:

Based on the uploaded pool format where cards use compact fields such as `s`, `h`, `t`, `ef`, `et`, `r`, `ab`, and `at`. 
