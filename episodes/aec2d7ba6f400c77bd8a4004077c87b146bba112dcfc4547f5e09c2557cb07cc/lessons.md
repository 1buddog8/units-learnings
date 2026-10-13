# Lessons: rea

Lineage `aec2d7ba6f400c77bd8a4004077c87b146bba112dcfc4547f5e09c2557cb07cc` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 34 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:report_ir

- Attempts: 14 (9 accepted, 5 no candidate)
- Best accepted effect: 34.92% (ratio 0.6508), episode `d02dad97cc12`
- Accepted effects: 34.92%, 16.74%, 10.08%, 4.12%, 3.50%, 2.45%, 2.32%, 1.99%, 1.24%
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.3851 USD (over 14 attempts with a recorded cost)

## Target perf:hits_ir

- Attempts: 20 (15 accepted, 2 rejected, 3 no candidate)
- Best accepted effect: 20.45% (ratio 0.7955), episode `0542a96fab94`
- Accepted effects: 20.45%, 15.33%, 9.30%, 8.00%, 6.92%, 6.81%, 6.70%, 4.76%, 3.24%, 2.20%, 1.91%, 1.68%, 1.19%, 1.07%, 1.06%
- Why candidates failed: stale_conflict (1), no_improvement (1)
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Files changed by rejected candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.3737 USD (over 20 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked on the per-hit path in _handle_breakpoint in bridge/native/rea_lldb_tracer.py. _record_json now returns the encoded record as a (head, tail) pair, split at the empty backtrace. That way the journal row no longer partitions the record text to find that point. Unexpected value types still go through _encode_json and are split the same way. I also moved the cache-hit path of _frame_location into the backtrace loop.
> The first evaluation, with 130 changed lines, was rejected. hits_ir went from 531651582 to 527402817, ratio 0.9920, and the required ratio was 0.9900. The third evaluation, with 155 lines, was accepted at 523498695, ratio 0.9847. report_ir rose to 478131099, ratio 1.0076, and that stayed within its guard. Outputs were identical in both runs.
> My reading is that most of the per-hit cost is in record encoding and frame lookup. Avoiding the partition alone was not enough.
> Next, I would look at why report_ir rose about as much as hits_ir fell. The candidate is queued for replay.
>
> 2026-10-13, episode `27cf399fabe3` (accepted)

> I worked in bridge/native/rea_lldb_tracer.py on the per-hit path. _record_json walked each record's register dicts a second time to encode them. Every register cost dict lookups, type checks, a cache lookup and an encode_basestring_ascii call. _record_estimate already has each register name and hex value in hand while it builds them, so I changed it to also build the encoded register objects. It uses the same cached name prefix. _record_json now reuses that work. The change is 54 lines in one file.
> In my sandbox the candidate was accepted with identical outputs and both builds ok. hits_ir went from 583016391 to 574007150, ratio 0.9845. report_ir went from 489297505 to 490041498, ratio 1.0015. That is a small cost on the report side.
> The hit path still spends real time on JSON assembly per register. Doing the work once at estimate time is the cleaner shape.
> Next, look at why report_ir rose slightly and check whether _frame_location repeats work in the same way. The replay verdict is still pending.
>
> 2026-10-13, episode `ed2003809d62` (accepted)

> I worked on the breakpoint hit path in bridge/native/rea_lldb_tracer.py. In _handle_breakpoint I inlined _wanted, _can_retain_trace and the cache-hit path of _frame_location for backtrace frames. I also removed the unused _location_sizes helper.
> Four sandbox runs, all with identical outputs, all rejected for no improvement against the required 0.9900. Evaluation 1 gave hits_ir 0.9931 with report_ir 1.0009. Evaluation 3 gave 0.9909 with report_ir 1.0025. Evaluation 5 also inlined the top-frame lookup. It did not help: 0.9932, with report_ir 1.0072. Evaluation 7 was the best, hits_ir 0.9907 with report_ir 1.0019. I ran out of evaluations and submitted nothing.
> Call overhead in _handle_breakpoint is real but small. Removing it gets close to the bar and stops just short. Inlining more grows the function and costs report_ir.
> Next, start from the evaluation 7 shape. Look for work that can be skipped per hit, not just calls that can be flattened. Leave the top-frame lookup as a call.
>
> 2026-10-13, episode `ce427637a045` (no candidate)

> I worked on the per-hit backtrace path in bridge/native/rea_lldb_tracer.py. In _handle_breakpoint I tightened the backtrace loop. I dropped a budget check that the estimate check already covers, bound the append methods to locals, and joined the journal row in one step. For the second attempt I added an id(symbol) cache for symbol lookups.
> The first change gave hits_ir 545465381 to 544846214, ratio 0.9989, and report_ir ratio 0.9965. Outputs were identical. It was rejected because 0.9989 did not reach the required 0.9900. The cache did not help. hits_ir went from 545465387 to 552702074, ratio 1.0133, so the lookup costs more than it saves. Outputs were identical. I submitted nothing.
> The loop body in _handle_breakpoint is already lean. Small local-binding gains are under 1% of hits_ir, and caching by id adds cost on this workload.
> Next time, look outside this loop. Read the rest of the per-hit path in lineage_bench.py and see where hits_ir is actually spent before editing.
>
> 2026-10-13, episode `751769ccb269` (no candidate)

> I worked on per-hit cost in bridge/native/rea_lldb_tracer.py. In _record_estimate I stopped calling _read_register once per argument register. That helper made a new SBError on every call. The registers are now read inline, in one loop with one shared SBError, and the same loop builds the register dicts and their size estimate. A failed read still raises with the same message before the next register is read. I also made smaller edits around lines 358, 590 and 620.
> Evaluation 1 at 30 lines: hits_ir went from 561919898 to 556259858, ratio 0.9899. report_ir moved to 496812103, ratio 1.0027. Evaluation 3 at 49 lines: hits_ir 552966343, ratio 0.9841, and report_ir 497805367, ratio 1.0047. Both builds were fine, outputs were identical, and guards held. The candidate is submitted and queued for replay.
> Allocation per register read was real cost in the hit path. The gains are small and report_ir drifts up a little.
> Next, look at why report_ir rises and check the backtrace frame path for the same pattern.
>
> 2026-10-13, episode `68b80abc03b4` (accepted)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
