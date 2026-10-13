# Lessons: rea

Lineage `aec2d7ba6f400c77bd8a4004077c87b146bba112dcfc4547f5e09c2557cb07cc` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 20 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:report_ir

- Attempts: 10 (8 accepted, 2 no candidate)
- Best accepted effect: 34.92% (ratio 0.6508), episode `d02dad97cc12`
- Accepted effects: 34.92%, 16.74%, 10.08%, 4.12%, 3.50%, 2.45%, 2.32%, 1.99%
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.3403 USD (over 10 attempts with a recorded cost)

## Target perf:hits_ir

- Attempts: 10 (7 accepted, 1 rejected, 2 no candidate)
- Best accepted effect: 20.45% (ratio 0.7955), episode `0542a96fab94`
- Accepted effects: 20.45%, 15.33%, 8.00%, 6.92%, 6.70%, 3.24%, 1.06%
- Why candidates failed: stale_conflict (1)
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Files changed by rejected candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.3038 USD (over 10 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I changed _frame_location in bridge/native/rea_lldb_tracer.py so it memoises the whole backtrace frame entry. The entry holds the location dict, the estimated size, and lazily filled json text. It is keyed by the raw LLDB values: load address int, file address int, module name, module path and symbol name. On a hit it skips the two hex formats and the five-key dict build per frame. The shared dict is never mutated. I also edited lines 547-554 to use the cached entry.
> My sandbox run on hits_ir was accepted with identical outputs. hits_ir went from 801994363 to 748589135, ratio 0.9334. report_ir went from 478355964 to 478632049, ratio 1.0006, within its guard. The change was 53 lines in 1 file. It is submitted and queued for replay. That is not a verdict.
> I believe per-frame formatting is a real cost in the hits path, because backtraces repeat the same frames.
> Next I want to look at what still runs per event after a cache hit, and check why report_ir moved slightly up.
>
> 2026-10-13, episode `4e46d287f462` (accepted)

> I tried to make the per-frame work cheaper in the backtrace loop of _handle_breakpoint in bridge/native/rea_lldb_tracer.py. Each frame used to build its location dict with _location and then pass it to _location_sizes. That built a memo key from tuple(location.items()), five 2-tuples per frame. I added a helper, _frame_location. It makes the same LLDB calls in the same order and looks up the memoised estimate and json text by a flat 5-tuple of the field values. I also trimmed four lines near line 547 down to three.
> In my sandbox, hits_ir went from 887398966 to 852394088, a ratio of 0.9606. report_ir went from 494613297 to 496228113, a ratio of 1.0033, so it rose slightly. Outputs were identical, both builds passed, and the guard was ok at 41 lines. The candidate is submitted and queued for replay.
> I now think the memo key construction was real overhead per frame, but it was not the main cost.
> Next I want to see why report_ir moved, and look at where else the frame loop spends time.
>
> 2026-10-13, episode `7c3b002016a7` (accepted)

> I worked in bridge/native/rea_lldb_tracer.py. _handle_breakpoint was serializing each event twice. It called json.dumps(record) for the exact budget check, then ran json.dumps again on the whole journal row once the backtrace was filled. I changed the memoised cache _location_sizes so it keeps each location's json.dumps text and not only its length. The size now comes from len() of that text. The journal row is built from the first record serialization. The guard counted 37 changed lines in 1 file.
> In my sandbox, hits_ir went from 966112051 to 767688224, a ratio of 0.7946. report_ir went from 496144500 to 496089121, a ratio of 0.9999, so that change did almost nothing for it. Outputs were identical, both builds passed, and the guard limits held. The candidate is submitted and queued, and replays will decide.
> My view now is that the hit path cost was mostly repeated serialization, not breakpoint handling itself.
> Next I want to look at the report path, since report_ir barely moved. I also want to check whether _location_json_size still has callers that need the length alone.
>
> 2026-10-13, episode `0542a96fab94` (accepted)

> In bridge/native/rea_lldb_tracer.py I read the breakpoint handling and the JSON size estimation, then made two edits. Lines 142-143 became 6 lines. Lines 498-525 became 38 lines. Both edits touch the area my searches covered: _handle_breakpoint, _estimate, _json_size and _location_json_size. I also read lineage_bench.py lines 95-155 to see what the bench measures.
> I ran no sandbox evaluations. Nothing was measured, so I cannot say whether either edit helped. The session ended with no candidate submitted.
> My current reading is this. The size estimation helpers in rea_lldb_tracer.py are what the bench exercises. The _location_sizes path and the journal appends are where repeated work might happen. This is a reading of the code, not a result.
> Next session, check whether the two edits are still in the working tree. Run lineage_bench.py on the unedited file to get a baseline, then run it on the edited file. Submit only if the numbers show a difference, and drop the edits if they do not.
>
> 2026-10-13, episode `efc14ef6b232` (no candidate)

> I worked in bridge/native/rea_lldb_tracer.py. In _report_location_sizes I sized the hex load_address and file_address strings with len() instead of passing them through the JSON string encoder. Hex strings need no escaping, so the length is known directly. That was the edit at lines 574-576.
> The build passed for base and candidate, and outputs were identical. report_ir went from 494017104 to 493108672, a ratio of 0.9982. The required ratio was 0.9900, so the change was rejected for no improvement. hits_ir moved from 1017586302 to 1017843763, ratio 1.0003, which is flat. I reverted the edit and submitted nothing.
> The address strings are not where report time goes. Skipping the encoder for them is correct but too small to matter. The larger cost is likely in _breakpoint_report, which recomputes module details on every hit.
> Next I will cache module name, path and related fields in _breakpoint_report and measure report_ir again against 0.9900.
>
> 2026-10-13, episode `592bbf0d1b8e` (no candidate)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
