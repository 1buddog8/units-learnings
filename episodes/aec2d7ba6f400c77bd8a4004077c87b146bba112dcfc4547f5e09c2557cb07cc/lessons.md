# Lessons: rea

Lineage `aec2d7ba6f400c77bd8a4004077c87b146bba112dcfc4547f5e09c2557cb07cc` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 41 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:report_ir

- Attempts: 16 (9 accepted, 7 no candidate)
- Best accepted effect: 34.92% (ratio 0.6508), episode `d02dad97cc12`
- Accepted effects: 34.92%, 16.74%, 10.08%, 4.12%, 3.50%, 2.45%, 2.32%, 1.99%, 1.24%
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.4441 USD (over 16 attempts with a recorded cost)

## Target perf:hits_ir

- Attempts: 25 (17 accepted, 2 rejected, 6 no candidate)
- Best accepted effect: 20.45% (ratio 0.7955), episode `0542a96fab94`
- Accepted effects: 20.45%, 15.33%, 9.30%, 8.00%, 6.92%, 6.81%, 6.70%, 4.77%, 4.76%, 3.24%, 2.20%, 1.91%, 1.68%, 1.68%, 1.19%, 1.07%, 1.06%
- Why candidates failed: stale_conflict (1), no_improvement (1)
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Files changed by rejected candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.4018 USD (over 25 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked in bridge/native/rea_lldb_tracer.py, in _handle_breakpoint. For every frame, the backtrace loop made six SB calls: GetModule, GetSymbol, GetLoadAddress, GetFileAddress, IsValid and GetName. All six only built the key for the cached _frame_location entry. I added a module-level _FRAME_ADDRESSES dict that maps id(address) to (address, entry). The entry holds the address, so the object stays alive and its id cannot be reused. I also made two small edits near lines 404 and 724.
> In my sandbox run, hits_ir went from 505171827 to 483181969, a ratio of 0.9565. report_ir went from 478402976 to 479553033, a ratio of 1.0024, so it did not improve. Outputs were identical and the guard passed. This is my own measurement, not a verdict.
> I now think per-frame SB calls are a real cost in the hit path. Report work is untouched by this change.
> Next, I should check whether the address objects are stable across hits in real lldb, not only in lineage_fakelldb.py. After that I should look for similar repeated SB calls on the report path.
>
> 2026-10-13, episode `9416a1233644` (accepted)

> I tried a leaner inner loop in _breakpoint_report in bridge/native/rea_lldb_tracer.py. I replaced repeated len() calls and global lookups with a local append and a counter. There were three small edits around lines 907, 924 and 956.
> It did not help. In my sandbox evaluation the outputs were identical and the build passed. report_ir went from 496494755 to 497076292, a ratio of 1.0012. The required ratio was 0.9900, so the evaluation rejected it with no_improvement. hits_ir went from 496134728 to 496044174, a ratio of 0.9998. I did not submit a candidate.
> I now believe the report loop is a negligible share of report_ir. hits_ir and report_ir both sit near 496M instructions. That means a shared fixed cost dominates both, not the code that builds the report.
> Next time I will measure where that shared fixed cost lives before editing anything. I would start with tracer setup and how lineage_bench.py loads the tracer through lineage_fakelldb.py. Micro-tuning _breakpoint_report again is not worth the time.
>
> 2026-10-13, episode `fea32001f136` (no candidate)

> I removed dead code from bridge/native/rea_lldb_tracer.py. The first attempt took out the unused _location_sizes/_LOCATION_SIZES cache, the registers_text=None fallback in _record_json, and _record as well. The second attempt put _record back and kept only the first two removals.
> The first attempt changed outputs on seeded inputs and was rejected. It measured hits_ir 512639624 to 509079270, ratio 0.9931, and report_ir ratio 0.9933. The second attempt gave identical outputs. It measured hits_ir 512639624 to 510756333, ratio 0.9963, and report_ir ratio 0.9944. The bar is 0.9900, so it did not help enough. I submitted nothing.
> _record is not dead. Something reaches it, and removing it changes output. The cache and the fallback really are unused, but they cost almost nothing. Trimming dead code in this file will not move hits_ir to the bar.
> Next time, find out who calls _record before touching it. Then profile the hot per-hit path in the tracer for real work to cut.
>
> 2026-10-13, episode `f781f54fbcec` (no candidate)

> I tried two changes in bridge/native/rea_lldb_tracer.py. In _handle_breakpoint I rebuilt the journal row as one format string, and I counted the separators in the frame loop instead of branching on each frame. I also removed the unused _location_sizes cache, which replaced 15 lines with 2.
> The build passed for base and candidate, and the outputs were identical. hits_ir went from 519046438 to 520052376, a ratio of 1.0019 against the required 0.9900. report_ir went from 477762924 to 476558865, a ratio of 0.9975. The change did not help, so I stopped and did not submit a candidate.
> The per-hit cost in _handle_breakpoint does not come mainly from how the row string is put together. Moving the separator logic into the loop costs about as much as the branch it replaced. Removing the dead cache is safe but does not move the numbers much.
> Next, I will profile which calls inside _handle_breakpoint dominate hits_ir before editing again. I will check the register reads and _frame_location first, and I will measure the cache removal by itself.
>
> 2026-10-13, episode `f4ed70e7d597` (no candidate)

> I removed the unused _location_sizes helper and its _LOCATION_SIZES cache from bridge/native/rea_lldb_tracer.py, lines 113-129, 17 lines replaced by 4. Searches found no callers in the tracer, the tests or the harnesses.
> The perf report_ir evaluation rejected it with no_improvement. report_ir went from 494884742 to 494078951, ratio 0.9984. hits_ir went from 512644868 to 509398568, ratio 0.9937. Both miss the 0.9900 bar. Outputs were identical, both builds were ok, and the guards passed with 1 file and 13 lines. The change did not help enough. I did not submit a candidate.
> Dead code is not where this cost lives. Every other private helper is still used by the tests or the equivalence harness, so there is nothing more to delete. Removing code that never runs cannot move instruction counts much.
> Next I will read _breakpoint_report and the per-hit path in rea_lldb_tracer.py, lines 652-762 and 800-990. I will profile those against lineage_fakelldb.py and look for repeated work per hit.
>
> 2026-10-13, episode `dcbbd960aef8` (no candidate)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
