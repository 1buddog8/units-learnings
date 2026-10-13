# Lessons: rea

Lineage `aec2d7ba6f400c77bd8a4004077c87b146bba112dcfc4547f5e09c2557cb07cc` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 38 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:report_ir

- Attempts: 15 (9 accepted, 6 no candidate)
- Best accepted effect: 34.92% (ratio 0.6508), episode `d02dad97cc12`
- Accepted effects: 34.92%, 16.74%, 10.08%, 4.12%, 3.50%, 2.45%, 2.32%, 1.99%, 1.24%
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.3951 USD (over 15 attempts with a recorded cost)

## Target perf:hits_ir

- Attempts: 23 (16 accepted, 2 rejected, 5 no candidate)
- Best accepted effect: 20.45% (ratio 0.7955), episode `0542a96fab94`
- Accepted effects: 20.45%, 15.33%, 9.30%, 8.00%, 6.92%, 6.81%, 6.70%, 4.76%, 3.24%, 2.20%, 1.91%, 1.68%, 1.68%, 1.19%, 1.07%, 1.06%
- Why candidates failed: stale_conflict (1), no_improvement (1)
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Files changed by rejected candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.3830 USD (over 23 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

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

> I tried to cut hits_ir in bridge/native/rea_lldb_tracer.py by memoising the per-target values that _dynamic_class reads on every hit: byte order, address size and the ObjC id type. I also reused the register-read SBError for the selector read instead of making a new one.
> The sandbox measured hits_ir going from 512783272 to 512616935, a ratio of 0.9997 against the required 0.9900. Outputs were identical and the guards were fine. report_ir moved from 503731284 to 504943416, ratio 1.0024. The change did not help, so I reverted it and submitted nothing.
> What I believe now: the per-hit lookups in _dynamic_class are not where the cost is. report_ir is about the same size as hits_ir, 503M against 512M, even though the two do different work. That suggests a large fixed cost shared by both runs.
> Next time I should measure what lineage_bench.py spends before any hit happens, such as setup and import in the tracer and the fake LLDB, and only then look at per-hit code.
>
> 2026-10-13, episode `d2084b692f3f` (no candidate)

> I worked on per-hit cost in bridge/native/rea_lldb_tracer.py. The journal row is now built as one list and joined once. It goes through a new _ObservationJournal.append_line, which takes text already ending in its newline, and append_json delegates to it. Before, prefix, head, backtrace, tail and newline were concatenated, copying the growing row several times. I also reworked backtrace frame memoisation around _new_frame_location and _frame_module_names.
> The first sandbox run gave hits_ir 516773816 to 515352235, ratio 0.9972. That was rejected against the required 0.9900. The second gave 0.9916, also rejected. The third gave 508395005, ratio 0.9838, and was accepted. report_ir stayed near flat at 0.9992, and outputs were identical each time. The verdict waits on replays.
> I now think the per-hit string building and frame lookups are where hits_ir goes. Each piece alone moved it very little.
> Next I want to profile the remaining per-hit path past the journal write and see what dominates now.
>
> 2026-10-13, episode `882f8f7415df` (accepted)

> I worked on the per-hit path in _handle_breakpoint in bridge/native/rea_lldb_tracer.py. _record_json now returns the encoded record as a (head, tail) pair, split at the empty backtrace. That way the journal row no longer partitions the record text to find that point. Unexpected value types still go through _encode_json and are split the same way. I also moved the cache-hit path of _frame_location into the backtrace loop.
> The first evaluation, with 130 changed lines, was rejected. hits_ir went from 531651582 to 527402817, ratio 0.9920, and the required ratio was 0.9900. The third evaluation, with 155 lines, was accepted at 523498695, ratio 0.9847. report_ir rose to 478131099, ratio 1.0076, and that stayed within its guard. Outputs were identical in both runs.
> My reading is that most of the per-hit cost is in record encoding and frame lookup. Avoiding the partition alone was not enough.
> Next, I would look at why report_ir rose about as much as hits_ir fell. The candidate is queued for replay.
>
> 2026-10-13, episode `27cf399fabe3` (accepted)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
