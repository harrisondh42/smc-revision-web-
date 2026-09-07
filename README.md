# SMC Trainer

A single-file web app running the 11-week SMC → BMO1 training plan for Harrison
and Sejun: 68 dated sessions, 17 techniques on spaced review, a paper-level
problem bank, and a peer-marking workflow for handwritten proofs.

Runs 3 September – 18 November 2026. Two users, no accounts, no backend.

## Run it

Open `index.html` in a browser. That's the whole thing — no build step, no
dependencies, no server. It also deploys to GitHub Pages as-is.

Storage is the one exception: shared progress needs the `db` runtime
capability, which is granted when the page is published as a claude.ai
Artifact. Opened as a local file it runs on an in-memory store and says so on
the Today screen rather than pretending to save.

## Source of truth

Built from three documents supplied on 7 September 2026:

| Document | Role |
|---|---|
| `smc-bmo1-plan.md` (v2) | The plan — sessions, techniques, marking protocol |
| `resources.md` | The verified resource stack, R01–R31 |
| `claude_design_smc.md` | The visual and interaction brief |

**The Claude Design canvas was not reachable.** The task named
`SMC Trainer.dc.html` in a `claude.ai/design` project, plus its `_ds` bundle,
`image-slot.js` and `support.js`. None of it could be read from the build
environment: the design MCP requires `/design-login`, which needs an
interactive terminal; the project URL returns 403 unauthenticated; and nothing
was seeded onto disk. The app is therefore built from the brief's written
specification, which fixes every colour, type size, permitted spacing value and
corner radius numerically. Every token lives in `:root` at the top of
`index.html`, so applying refinements made in the canvas is a matter of
changing variables, not rewriting components.

## Design fidelity

Implemented as specified: the seven-colour palette with nothing outside it
(audited — no green, orange, yellow or red is painted anywhere), weight 350
throughout with 400 only under 13px, the halved in-app spacing scale, the
per-element radius scale, the 54px display size for countdowns and the runner
timer, the floating pill nav, the six-state problem machine encoded as fill
weight with blue reserved for "outside help was used", and fixed-position H/S
initials never distinguished by colour.

Two deliberate departures from the brief, both flagged rather than silent:

1. **`window.storage` does not exist.** Brief §6 specifies
   `window.storage` with `shared: true`. There is no such API. The real
   primitive for shared, server-side, hierarchically-keyed state is the `db`
   capability (`await claude.use("db")`), which is what the app uses — three
   documents, `state/harrison`, `state/sejun` and `state/shared`, with
   debounced batched writes and live cross-viewer sync. The brief's
   no-`localStorage` rule is honoured: neither `localStorage` nor
   `sessionStorage` appears anywhere.

2. **Problems and Library carry no filled button.** Brief §1 asks for exactly
   one filled primary action per screen. Both are pure reference screens with
   no single primary action, and inventing one to satisfy the count would be
   worse design. Calendar got a real one — *Jump to today*, which an 11-week
   agenda genuinely needs. Login has none by the brief's own §3.1, which
   specifies both student buttons as hairline pills.

## Judgement calls in the data

- **T16 / T17 mapping is inferred.** The plan names techniques in the week
  table but never binds them to IDs. Cross-referencing the "Techniques
  introduced" and "Under review" columns pins T01–T15 unambiguously. Week 10
  introduces three named techniques (AM–GM, Cauchy–Schwarz, Vieta) with two IDs
  left, so AM–GM and Cauchy–Schwarz are grouped as T16 (they share session
  S060) and Vieta is T17 (S062). Change it in one line if that's wrong.

- **No problem is tagged by topic.** Plan §11 declines to tag BMO1 problems
  because its author had not read them — "assigning topics I have not verified
  would be fabrication". That constraint is kept: every paper reads *untagged*,
  and the Problems screen says why and points at R02 for the tagging job.
  Per-question state *is* tracked, because BMO1's six-problem structure is
  documented and the plan's own sessions reference single questions.

- **Review dates are the plan's, not invented.** Where the plan schedules only
  one review cycle for a technique, one is shown. T02, T16 and T17 have one.

- **Block minutes are reproduced as written**, including where a session's
  blocks don't sum to its stated duration. The runner notes the mismatch rather
  than silently correcting the plan.

## Open questions

- **Assumption A1 is unresolved.** The plan records Mon/Tue/Wed "6:30–10:00"
  with no am/pm and flags it as ambiguous; §14 says to resolve it before
  building. It sets time-of-day on 30 of 68 sessions, so **no session carries a
  start time** — guessing would put half the schedule in the wrong half of the
  day. Resolve it and times can be added in one pass.

- **Week 5's review column has nowhere to land.** The plan's week table lists
  T03, T04 and T07 as "under review" in week 5, but week 5 is the SMC taper and
  none of its sessions contain a review block. Surfaced in the technique
  tracker rather than smoothed over.

## Motion and the first-run guide

Added from the motion brief. No decorative motion: every animation either
explains a rule of the plan or confirms a state change the user caused.

**Six kinetic figures**, driven by numerical integration on
`requestAnimationFrame` — not CSS keyframes. A damped spring
(`v += (-k(x-target) - cv)dt`, semi-implicit Euler, `dt` clamped to 32ms)
underlies most; the bouncing figures are ballistic with explicit gravity and
restitution. Constants are the brief's. Each pauses off-screen via
`IntersectionObserver`, and under `prefers-reduced-motion` renders its settled
end state rather than a frozen mid-frame — verified: the branch dot sits at the
end of the BMO1 arm, the ball on Rewrite, the ring full.

The brief is written against a React draft (`useRef` for sim state, `setState`
only to force a repaint). This app is vanilla, so the principle applies more
directly: sim state is a plain object and each frame writes straight to SVG
attributes. Step functions are pure `(state, dt) => void` and live outside the
builders, as specified.

**First-run guide** — six steps, opening automatically on first login and
reopenable from "How this works" beside the week label on Today. Reopening
always starts at step one. Copy is verbatim from the plan and brief. The
`guideSeen` flag is persisted per student through the app's own store, not
`localStorage`; the overlay waits for the store to answer before opening, so a
student who has already been through it never sees a flash of it.

### Departures from the motion brief

- **The 2px footer rule** conflicts with the base visual system, which
  specifies 0.5px hairlines throughout. Implemented at 2px as the motion brief
  states, and marked in the CSS as the one deliberate exception.
- **The countdown chip's scrub spring is not implemented.** The row describes a
  day figure springing "when the clock is scrubbed". There is no clock
  scrubber in the production app, so there is nothing to attach it to.
- **Figure apex heights were raised.** `g=540` and the restitutions are the
  brief's and are unchanged, but the hop apex was not specified and the value
  that matched the draft left the ball crawling along the bottom edge of the
  200x200 box. Raising the apex changes hop duration only, not the physics
  constants.
- **Figures 4 and 6 now start from zero** rather than at their first target.
  Initialising at the target left them motionless for the first 2.6s and 4.8s
  respectively — a teaching diagram that does not move is not teaching.
- **Figure 4 animates fill on all six bars.** The static UI gives
  "unattempted" and "stuck" no fill at all; here every bar animates, because
  the figure's subject is fill weight. The semantics stay in the stroke.

### Session runner

The runner previously rebuilt its markup every second, which would have
destroyed the ring's `transition: 1s` by recreating the element at its
destination value each tick. It is now split: `render()` rebuilds on a block
change, `paint()` updates the timer, ring and hint control in place.

## Catch up

Sessions whose day has passed and which were neither completed nor deliberately
let go. Reached from a card on Today when the student is behind, and in full
from the **Catch up** tab beside Progress.

Ordering comes from the plan's own triage (§13): *cut Sunday first, then
Tuesday, never Thursday*. Recovery runs the other way, so a missed mock ranks
first, then a Thursday paired session, then other paired work, with Tuesday and
Sunday last — and the panel says which band each session is in, so the student
can see what to protect and what to drop.

Three actions per session: run it now (opens the runner), mark it done, or let
it go. Letting go is a first-class decision, not a silent gap: it is recorded,
listed separately, reversible at any time, and still counts against hours
planned. A session completed after its date records `late: true`, so the
calendar shows *caught up* rather than a plain tick and the progress dashboard
separates "done on the day" from "caught up late".

## Bug fixed: progress was not saving

Marking a session done saved once and then silently stopped, and the other
student never saw it. The cause was in the store adoption path:

```js
State.student.harrison = Object.assign(blankStudent(), snapshot.data());
```

`db` freezes delivered snapshots and everything inside them. `Object.assign` is
a shallow copy, so the snapshot's frozen `sessions`, `techniques`, `problems`
and `proofs` objects went straight into application state. Under `"use strict"`
every later mutation threw `TypeError: Cannot assign to read only property` and
was lost. The first mark appeared to work only because the document did not yet
exist, so the fallback object was a fresh unfrozen one; the write that followed
delivered a frozen snapshot and every mark after that failed.

Snapshots are now thawed by `adopt()` before entering state. Two related fixes
came with it: a remote snapshot is no longer applied while a local write is
still queued, which was a window for losing an edit; and the pending-write
marker is cleared once the write goes out, so remote updates resume.

Verified with a stand-in `db` that reproduces the real one's frozen snapshots,
and with two pages against one shared store: a mark by one student now reaches
the other in both directions.

## Layout

Everything is in `index.html`, in labelled sections: design tokens, the data
layer (`SESSIONS`, `TECHNIQUES`, `RESOURCES`, `PROBLEMS`, `WEEKS`, `PHASES`,
`BLACKOUTS`, `EXAMS`), state and persistence, then one function per screen.
The data layer is the part you'll edit; it is plain arrays of plain objects.
