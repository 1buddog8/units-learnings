# Lessons: rea

Lineage `aec2d7ba6f400c77bd8a4004077c87b146bba112dcfc4547f5e09c2557cb07cc` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 7 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:report_ir

- Attempts: 2 (2 accepted)
- Best accepted effect: 34.92% (ratio 0.6508), episode `d02dad97cc12`
- Accepted effects: 34.92%, 10.08%
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.2778 USD (over 2 attempts with a recorded cost)

## Target perf:hits_ir

- Attempts: 5 (3 accepted, 1 rejected, 1 no candidate)
- Best accepted effect: 15.33% (ratio 0.8467), episode `8579b7963c0d`
- Accepted effects: 15.33%, 8.00%, 6.92%
- Why candidates failed: stale_conflict (1)
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Files changed by rejected candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.2663 USD (over 5 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I read bridge/native/rea_lldb_tracer.py around the hit path, plus lineage_bench.py and lineage_fakelldb.py. I was looking for a hits_ir change that does not overlap my pending candidate. That candidate caches per-location sizes in the backtrace loop of _handle_breakpoint and is still being replayed.
> I made no edits and ran no sandbox evaluations. I measured nothing this session, so there are no numbers to report. Searches for _record, _fits_trace_budget and _estimated_json_size returned 30 matching lines. I traced the per-hit work outside the backtrace loop and found no clear gain. The fake LLDB accessors that _location calls are trivial. The journal's json.dumps of each record is needed for its output. I stopped without a candidate.
> My view now is that most of the per-hit cost sits in the backtrace loop my pending candidate already touches. Outside it there is little to cut.
> Next, wait for the replay result on the pending candidate. If it holds, look at what remains in _handle_breakpoint after that change. If it does not, start again from _fits_trace_budget.
>
> 2026-10-13, episode `4bf9bd27bdcf` (no candidate)

> I worked in bridge/native/rea_lldb_tracer.py, in the backtrace loop of _handle_breakpoint. Each frame location used to have its estimated JSON size and exact JSON length computed again on every hit. I added a small module-level cache keyed by tuple(location.items()). It holds at most 4096 entries and is cleared when full. That is 23 changed lines in one file.
> In my sandbox the evaluation was accepted. hits_ir went from 1225871606 to 1019067593, a ratio of 0.8313. report_ir went from 691038215 to 691620842, a ratio of 1.0008, so the report path did not get faster and did not get meaningfully worse. Outputs were identical and the guards were within limits. This is my own measurement, not the replay verdict.
> I now believe backtraces repeat the same code addresses across hits. The Python sizing loops were paying for that repetition once per frame.
> Next, check how often the cache hits its 4096 limit and is cleared on long traces. After that, look for other per-frame work in _handle_breakpoint that also repeats.
>
> 2026-10-13, episode `8579b7963c0d` (accepted)

> I read bridge/native/rea_lldb_tracer.py, around the breakpoint hit path. Each backtrace location's JSON size was found with len(json.dumps(location)). I added a helper, _location_json_size, that computes the same length directly. For a flat dict of str or None values it sums the escaped length of each key and value with json.encoder.encode_basestring_ascii, then adds separator and brace overhead. The call site at line 493 now uses it.
> My sandbox run accepted the change. hits_ir went from 1292893000 to 1196316800, ratio 0.9253. report_ir went from 666124985 to 667039488, ratio 1.0014, inside its guard limit. Outputs were identical. The diff was 21 lines in 1 file. Independent replays will decide the verdict.
> I now believe the hit path spends real time serializing just to measure size. The helper is only correct while locations stay flat with str or None values.
> Next, I would look for other json.dumps calls in the hit path whose output is only measured or discarded. I would also add a check that keeps the helper and json.dumps in agreement.
>
> 2026-10-13, episode `1f1ef56746be` (accepted)

> I tried a per-call cache in _breakpoint_report in bridge/native/rea_lldb_tracer.py. Each location's estimated size and its exact json.dumps length are now stored once per call, keyed by the tuple of the location's values. The same address often appears in several breakpoints, and before this its size was recomputed every time. The exact length is still only computed when the estimate cannot settle the budget question. There were two edits in that function, 18 changed lines in total.
> My sandbox run on report_ir was accepted. The build was ok on both sides, outputs were identical, and the guards were within limits. report_ir went from 1068202623 to 683868368, ratio 0.6402. hits_ir barely moved: 1291463255 to 1291260109, ratio 0.9998.
> I now believe report cost here is dominated by repeated sizing of duplicate locations, not by the truncation logic itself.
> Next, I want to wait for the replay verdict. Then I will look at whether hits_ir has a similar repeated cost elsewhere in the tracer.
>
> 2026-10-13, episode `d02dad97cc12` (accepted)

> I worked on the breakpoint hit path in bridge/native/rea_lldb_tracer.py. The record, primary location and backtrace location size estimates go through _estimated_location_size. I changed that helper so it handles exact int and float values inline, still counting 32 as before, and falls back to _estimated_json_size for other types. The estimates should come out unchanged. There were four edits in one file, 13 changed lines.
> In my sandbox the guard passed and both builds were ok. Outputs were identical. hits_ir went from 1423031513 to 1336747831, a ratio of 0.9394. report_ir went from 1051309816 to 1051639635, a ratio of 1.0003, so the report path did not get cheaper. I submitted it and it is queued for independent replay.
> I now think the per-hit cost here is mostly small repeated size estimation on flat dicts. Dispatching on common types inline is cheaper than the general helper.
> Next, I want to look at the exact size computation via json.dumps on the hit path. I also want to see why report_ir moved slightly up.
>
> 2026-10-13, episode `753995c17d81` (rejected)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
