# Daybook

A brain-app for [BrainFoundry](https://brainfoundry.ai) brains.

Created: 2026-05-20

Twice a day, your brain asks you something it actually knows to ask. You
answer. The arc stays linked.

- **Morning:** your brain reads recurring themes in your corpus (and last
  night's reflection, if any) and proposes one short, concrete question to
  set today's intention.
- **Evening:** your brain quotes your morning intention back to you and asks
  what actually happened.

Both entries are appended to the episodic layer. The evening entry carries a
`paired_id` to its morning pair, so the daily arc stays linked for later
retrieval.

## Install

In your brain: **Settings → Apps**, paste this repository's URL, **Preview**,
**Install**. The app adds a *Daybook* tab.

## How it works

The app carries no intelligence of its own. It borrows the host brain's
reasoner and memory through the brain-app `postMessage` bridge:

- `llm.complete` — the brain generates the prompt over its own corpus (RAG),
  using the operator's selected model. The system prompt instructs the brain
  to prefer one short, concrete question that references a recurring theme
  or yesterday's evening entry, and to never fabricate quoted text.
- `memory.write` — saving the entry appends one episodic record with
  `metadata.kind = "daybook-morning"` or `"daybook-evening"`.

The app picks its mode from the wall clock and what is already saved for
today in episodic. No mode switcher in the UI — one mode at a time:

- **Morning mode** (default before the evening hour) — brain proposes the
  morning prompt; textarea + Save. Default evening hour is 18:00 local.
- **Evening mode** (default at or after the evening hour) — brain quotes
  the morning intention and asks what happened; textarea + Save.
- **Already-written mode** — if today's slot is filled, the saved entry
  renders inline (read-only). No second write to the same slot in v0.

A small header line always shows today's date and a one-line peek at
yesterday's evening entry, so the operator sees "yesterday I wrote: …"
without leaving the app.

## Permissions

| Permission | Why |
|---|---|
| `llm.invoke` | generate the day's prompt over the `semantic` + `episodic` layers |
| `memory.write` | append the morning/evening entry to the `episodic` layer |

See `brain-app.yaml` for the full manifest and declared layers.

## Pairing model

- Morning save → `metadata.kind = "daybook-morning"`. The returned entry id
  is cached in `localStorage` keyed by today's date.
- Evening save → `metadata.kind = "daybook-evening"`,
  `metadata.paired_id = <morning entry id>` when the cached id is available.
  When the evening visit happens on a different browser/device than the
  morning, the cached id is absent and `paired_id` is omitted; the pair is
  still derivable by date.

The v1 cleanup is `memory.read` with a date filter — once the bridge ships
it, mode detection and pair lookup move off the `llm.complete` round-trip.

## Standalone

Opened as a file with no host brain, the app shows a clear "no brain
connected — open this app inside a brain" message. Daybook is meaningless
without a brain to ask, so it does not invent local prompts.

## Build

None. One self-contained `index.html` — no dependencies, no CDN, no build
step. A sovereignty claim shouldn't ship third-party script.

## License

AGPL-3.0 — see [`LICENSE`](./LICENSE). Brain-apps must be license-compatible
with the brain they install into.
