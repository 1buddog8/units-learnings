# Lessons: rea

Lineage `aec2d7ba6f400c77bd8a4004077c87b146bba112dcfc4547f5e09c2557cb07cc` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 30 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:report_ir

- Attempts: 13 (9 accepted, 4 no candidate)
- Best accepted effect: 34.92% (ratio 0.6508), episode `d02dad97cc12`
- Accepted effects: 34.92%, 16.74%, 10.08%, 4.12%, 3.50%, 2.45%, 2.32%, 1.99%, 1.24%
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.3751 USD (over 13 attempts with a recorded cost)

## Target perf:hits_ir

- Attempts: 17 (13 accepted, 2 rejected, 2 no candidate)
- Best accepted effect: 20.45% (ratio 0.7955), episode `0542a96fab94`
- Accepted effects: 20.45%, 15.33%, 9.30%, 8.00%, 6.92%, 6.81%, 6.70%, 4.76%, 3.24%, 2.20%, 1.91%, 1.19%, 1.06%
- Why candidates failed: stale_conflict (1), no_improvement (1)
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Files changed by rejected candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.3614 USD (over 17 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked on per-hit cost in bridge/native/rea_lldb_tracer.py. In _record_estimate I stopped calling _read_register once per argument register. That helper made a new SBError on every call. The registers are now read inline, in one loop with one shared SBError, and the same loop builds the register dicts and their size estimate. A failed read still raises with the same message before the next register is read. I also made smaller edits around lines 358, 590 and 620.
> Evaluation 1 at 30 lines: hits_ir went from 561919898 to 556259858, ratio 0.9899. report_ir moved to 496812103, ratio 1.0027. Evaluation 3 at 49 lines: hits_ir 552966343, ratio 0.9841, and report_ir 497805367, ratio 1.0047. Both builds were fine, outputs were identical, and guards held. The candidate is submitted and queued for replay.
> Allocation per register read was real cost in the hit path. The gains are small and report_ir drifts up a little.
> Next, look at why report_ir rises and check the backtrace frame path for the same pattern.
>
> 2026-10-13, episode `68b80abc03b4` (accepted)

> I worked in bridge/native/rea_lldb_tracer.py on the per-location loop in _breakpoint_report. I made two edits, at lines 847-892 and then 865-893. The goal was fewer checks and fewer tuple lookups for each location. I read lineage_bench.py and lineage_fakelldb.py first to see what the benchmark exercises.
> The sandbox measured report_ir at 491378643 for the parent and 491621433 for the candidate, a ratio of 1.0005. The required ratio was 0.9900, so it was rejected as no_improvement. hits_ir went from 595593614 to 597558911, ratio 1.0033. Outputs were identical, the build was ok and the guard was ok. The change did not help. I submitted no candidate.
> I now think the tracer's own share of report_ir is small. The fake SB calls in lineage_fakelldb.py are trivial. Address and module resolution is already cached per call. Tidying the loop cannot move a number it barely owns.
> Next time, profile report_ir first to see where the cost actually sits, maybe in lineage_bench.py itself, before editing _breakpoint_report again.
>
> 2026-10-13, episode `7fc013febf9b` (no candidate)

> I worked in bridge/native/rea_lldb_tracer.py on the per-hit register path. _record_estimate called _read_register once for each argument register, and each call built a new lldb.SBError. I moved those reads inline so they share one SBError. A failure still raises before the next read, so the error state and the messages stay the same. I also changed how _record_json encodes register names.
> The first version, 18 lines, took hits_ir from 601127471 to 594553196, a ratio of 0.9891. report_ir went to 1.0019. The second version, 33 lines, took hits_ir from 601127465 to 592481119, a ratio of 0.9856, and report_ir came in at 0.9988. Both runs gave identical outputs and passed the guard. I submitted the 33-line candidate. It is queued for independent replays.
> The cost here is many small per-register calls on every hit. Allocation and repeated encoding add up, but each one is small.
> Next I will profile _handle_breakpoint and the frame location lookups. I want to know whether a cost per hit remains outside register handling.
>
> 2026-10-13, episode `a93f42a33100` (rejected)

> I read _handle_breakpoint in bridge/native/rea_lldb_tracer.py. On every breakpoint hit it called _encode_json on the whole record. That record includes the five location keys, whose encoded text is already cached in the _frame_location entry. I added a helper, _record_json, that builds the same JSON text directly. It writes integers and floats with repr, strings with json's encode_basestring_ascii, and reuses the cached location text. I made small edits at two call sites near lines 614 and 650.
> In the sandbox, hits_ir went from 573088593 to 532721171, a ratio of 0.9296. report_ir went from 484614811 to 486472508, a ratio of 1.0038, so the report path got slightly worse. Outputs were identical, both builds passed, and the guard was ok with 58 lines in 1 file. The candidate is submitted and queued for independent replay.
> I now believe generic JSON encoding is a real cost per hit in this tracer, and cached fragments are worth reusing.
> Next I would look at why report_ir rose and check whether other per-hit encoders can reuse cached text.
>
> 2026-10-13, episode `2f9abce93645` (accepted)

> I worked on the per-hit path in bridge/native/rea_lldb_tracer.py. _handle_breakpoint already filters symbols with _wanted before it calls _record_estimate, and _record_estimate ran the same check again on every hit. I moved that check into _record, the only other caller, so each hit evaluates it once. I also stopped calling the generic _estimated_json_size for sequence, thread_id, index, receiver_class and selector in _record_estimate. Those values are now sized inline. The change touches 24 lines in one file.
> My sandbox run accepted it with identical outputs. hits_ir went from 630025291 to 622789566, a ratio of 0.9885. report_ir went from 479315782 to 481436376, a ratio of 1.0044. Guards were within limits.
> So the per-hit win is small and real in my run. The report path got slightly worse, and I did not find out why. The verdict waits on independent replays.
> Next I would look at why report_ir rose. I would also read the rest of _record_estimate for other generic sizing calls.
>
> 2026-10-13, episode `8fea2998249b` (accepted)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
