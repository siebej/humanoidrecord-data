# Data schema and evidence rules

One JSON file per robot in `data/robots/<slug>.json`. `<slug>` must match the
`slug` field exactly (kebab-case, e.g. `unitree-g1`).

## Top-level fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `slug` | string | yes | matches filename without `.json` |
| `name` | string | yes | model name, e.g. "G1" |
| `manufacturer` | object | yes | `{ name, country, url }` |
| `category` | enum | yes | one of `full-size`, `compact`, `research`, `industrial` |
| `announced` | object | yes | `{ date: YYYY-MM-DD, source }` |
| `specs` | object | yes | see Spec fields below |
| `price_history` | array | yes (may be empty) | see Price entries below |
| `delivery` | object | yes | `{ status, timeline: [] }` |
| `notes` | array | no | free-text notes, e.g. flagging contradicting sources |
| `updated` | string | yes | `YYYY-MM-DD`, last edit date of the file |

## Spec fields

`specs` holds a fixed set of keys, each optional individually but if present
must follow this shape:

```json
{ "value": <number|string>, "status": "claimed|demonstrated|shipped", "source": { ... } }
```

Recognized keys: `height_cm`, `weight_kg`, `dof`, `battery_runtime_min`,
`walking_speed_kmh`, `payload_kg`, `hands`. Add more only if a robot has a
genuinely distinct, sourced attribute — do not pad with unsourced fields.

## Source object

Every value — spec, price, or timeline event — carries a `source`:

| Field | Required | Notes |
|---|---|---|
| `url` | yes | link to the primary evidence |
| `quote` | yes | literal quote, max 300 characters |
| `accessed` | yes | `YYYY-MM-DD`, date the source was checked |
| `archive` | no | archive.org (or similar) permalink |

## Price entries

`price_history` is an array, oldest first:

```json
{ "date": "YYYY-MM-DD", "amount": <number>, "currency": "USD", "variant": "string", "status": "claimed|demonstrated|shipped", "source": { ... } }
```

## Delivery

`delivery.status` is the robot's current overall standing: `shipped`,
`demonstrated`, `claimed`, or `announced-only`. `delivery.timeline` is an
array of dated events:

```json
{ "date": "YYYY-MM-DD", "event": "announced|first-demo|preorders-open|first-customer-delivery|volume-estimate", "detail": "string", "quantity_min": null, "quantity_max": null, "source": { ... } }
```

Only include an event once it has real evidence. Do not invent a
`first-customer-delivery` event to fill a gap — if it hasn't happened, the
timeline simply stops at the last real event, and `delivery.status` reflects
that (`claimed` or `demonstrated`, not `shipped`).

## The three statuses, precisely

### claimed
The manufacturer's own words: a specs page on the manufacturer's domain, a
press release, a regulatory/investor filing, or an interview with a named
person and a date, published by the manufacturer or a credible outlet
quoting them directly. Blog posts, "reviews", or news articles that merely
repeat a number without citing the manufacturer directly do **not** qualify
as the source for a `claimed` value — trace it back to the manufacturer.

### demonstrated
An uncut video or live demonstration, run by an independent party (press,
customer, researcher), OR a manufacturer-produced video in which the
property in question is visibly performed with a timestamp (e.g. a battery
runtime demonstrated across a visibly continuous, timestamped clip). CGI
renders, heavily cut marketing reels, and "coming soon" trailers do not
qualify.

### shipped
A named customer, with photographic or video evidence of a physical unit in
their possession, plus at least one of: an invoice, a serial number, an
independent unboxing, or a customs/import record. A signed contract,
preorder, deposit, or a "shipping in Q4" statement is not shipped — it is
`claimed` (or `demonstrated` if a working unit was shown).

## Contradicting sources

If two credible sources disagree on a value (different height, different
delivery date, etc.), do not silently pick one. Record both, each with its
own `source` and `accessed` date, and add a short note in `notes` explaining
the discrepancy. The comparison table on the homepage links to whichever
value is most recently sourced, but the robot page must show both.

## Quantities

Delivered/produced quantities are almost never precisely disclosed. Use
`quantity_min` / `quantity_max` as a range, each end backed by its own
source where possible (e.g. "at least 500" from one filing, "fewer than
2,000" inferred from another). If no credible number exists at all, omit
the timeline event entirely rather than guessing.

## Validation

`tools/validate.js` enforces: required fields present, `status` is exactly
one of the three values, every value-bearing object has a non-empty
`source.url`, `source.quote` (≤300 chars) and `source.accessed`
(`YYYY-MM-DD`), and `slug` matches the filename.
