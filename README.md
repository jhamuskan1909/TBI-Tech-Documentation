
# TBI Technical Team — Selection Task

## Candidate Details
- Name: Muskan
- Roll No.: 25BCE11431

## Part 1 — Bug Fixes

The original snippet had three bugs:

1. **Multiple intervals on repeated clicks.** `startTimer()` called `setInterval`
   unconditionally, so clicking Start more than once created multiple intervals
   all decrementing the same `timeLeft`, making the countdown speed up with each
   click. Fixed by tracking the interval ID and refusing to start a second one
   while one is already running.

2. **Seconds not zero-padded.** Once `seconds < 10`, the display showed e.g.
   `9:5` instead of `9:05`. Fixed by padding to two digits.

3. **No stop condition at zero.** `timeLeft` decremented past zero forever,
   producing negative minutes/seconds. Fixed by clamping at 0 and clearing the
   interval once it's reached.

See `part1.html` (fixed) vs `original.html` (untouched) for the diff.

## Part 2 — Pause/Resume + Drift-Proofing

**Requirement 1: pause/resume.**
Added a Pause/Resume button. Pausing snapshots the accurate remaining time and
stops the interval; resuming rebuilds the timer state from that snapshot.

**Requirement 2: no drift, even through sleep/backgrounding.**
The root problem with the original design is that it treats `setInterval` as a
reliable metronome — decrementing a counter once per "tick" and trusting that
each tick represents exactly one real second. Browsers throttle or fully
suspend `setInterval` in background tabs, and a sleeping laptop stops firing
it entirely. When it wakes up, only the ticks that actually fired got counted,
so the countdown falls behind real time.

**Fix:** stop counting ticks. Instead, store a target end timestamp
(`endTime = Date.now() + remaining * 1000`) once, and on every tick recompute
`remaining` as `endTime - Date.now()`. This makes the displayed time a function
of real wall-clock time, not of how many interval callbacks happened to run —
so it's immune to throttling, missed ticks, or the tab/laptop being asleep for
an arbitrary stretch. A `visibilitychange` listener also forces an immediate
recalculation the moment the tab regains focus, so the display doesn't wait
for the next scheduled tick to catch up.

## Files
- `original.html` — the buggy snippet, untouched, for side-by-side diffing
- `part1.html` — bug-fixed version (Part 1 only)
- `part2.html` — part1 + pause/resume + drift-proofing (final version)
