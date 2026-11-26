# Design Notes

Reference material gathered while scoping the engine. Semantics only;
implementation is ours. Sources listed at the bottom.

## Time model

- Pattern time is measured in **cycles**, not seconds. One cycle is one
  full iteration of a pattern. Tempo maps cycles to wall clock.
- Pattern is a function of time: `query(begin, end) -> events`. To render
  a timeslice, call the pattern with the slice bounds and collect events.
- Rational numbers for all pattern-internal time. Keeps tuplets and deep
  nesting exact. Float only at the final conversion to seconds.
- Cycle length never changes. Nesting and grouping divide the cycle,
  adding events never stretches it. This is the core rule of the whole
  notation.

## Mini-notation operators

| op | meaning |
|----|---------|
| space | sequence children, equal division of parent span |
| `[ ]` | group; children share the span one outer element would take |
| `< >` | alternation: pick one child per cycle, cycle through in order |
| `{ }` | polymeter stack: children play simultaneously, each at its own step rate; combined with `%n` scales to n steps total |
| `,` | stack: children overlap in parallel, same span |
| `~` or `-` | rest (silence, still occupies time) |
| `/n` | slow the group by n (takes n cycles to complete) |
| `*n` or `n` suffix | speed up by n |
| `@n` | elongate child to n times its normal span (non-integer ok; default 1) |
| `!n` | replicate child n times, no speed change |
| `?` | 50% chance the event is skipped each query |
| `\|` | choose one of the alternatives at random |
| `(a,b,c)` | euclid: a pulses spread over b segments, rotation c (optional, default 0) |
| `n` bare | with `<>`: n events per cycle |
| `'` ` ` | quoted pattern strings; backticks allow line breaks |

Notes:

- `<>` picks per cycle; `[]` divides within the cycle. `[a b]/2` = the pair
  spread over two cycles.
- Weighted spans: `[a@2 b c]` gives a half the span, b and c a quarter each.
- Euclid negative pulses = inverted (hits become gaps).
- Rest and silence are distinct from "no events"; rests carry position info.

## Control parameters

Control params (`note`, `s`ound, `cutoff`, `gain`, `pan`, etc.) are
patterns too. Each takes a value, list, or pattern and merges into the
event's value dict. Applied in chain order, e.g.
`note("c a f e").s("piano").gain(".5 1")`.

## Higher-order pattern ops (post v1)

`rev`, `palindrome`, `every`, `whenmod`, `iter`, `ply`, `segment`,
`early/late` (nudge), `clip/legato` (duration multiplier), `off`, `stut`
(delay feedback). Table of equivalents:

| function | mini |
|----------|------|
| slow(2) | "/2" |
| fast(2) | "*2" |
| euclid(3,8) | "(3,8)" |
| euclidRot(3,8,1) | "(3,8,1)" |

## Scheduler reference numbers

From Tidal's Tempo module (see sources):

- Process chunks of ~50 ms, scheduled ~333 ms ahead of playback.
- Timeslice polling at ~20 Hz.
- Tempo bridging (Ableton Link in their case) maps cycles <-> wall clock.
- Our equivalent: lookahead thread computes events for the next chunk,
  audio callback consumes from the queue. Queue depth >= lookahead.

## Live highlighting

Strudel wires source positions from parsed mini-notation into the AST,
then highlights the active slice per query window. Cheap to do at
render time: track which text span the currently sounding event came
from. Store (begin, end) byte offsets on each event during parse.

## v1 scope

Adopt: spaces, `[ ]`, `< >`, `,`, `~`/`-`, `/n`, `*n`, `@n`, `!n`, `?`,
`(a,b,c)` euclid, bare numeric rate, control params as patterns.

Defer: `{ }` polymeter, `|` choice, higher-order ops, `%n` scaling,
negative-pulse euclid inversions.

## Sources

- strudel.cc/learn/mini-notation (operator semantics)
- strudel.cc/learn/time-modifiers
- strudel.cc/learn/factories
- uzu.lurk.org forum thread "How does Tidal scheduler work?" (Tempo module internals)
- tidalcycles/strudel wiki, Technical Manual (parser/AST/highlighting)
