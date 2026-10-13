# Lessons: rea

Lineage `aec2d7ba6f400c77bd8a4004077c87b146bba112dcfc4547f5e09c2557cb07cc` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 5 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:report_ir

- Attempts: 2 (2 accepted)
- Best accepted effect: 34.92% (ratio 0.6508), episode `d02dad97cc12`
- Accepted effects: 34.92%, 10.08%
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.2778 USD (over 2 attempts with a recorded cost)

## Target perf:hits_ir

- Attempts: 3 (2 accepted, 1 rejected)
- Best accepted effect: 8.00% (ratio 0.9200), episode `2779233538b3`
- Accepted effects: 8.00%, 6.92%
- Why candidates failed: stale_conflict (1)
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Files changed by rejected candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.2569 USD (over 3 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

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

> I worked in bridge/native/rea_lldb_tracer.py on the breakpoint hit path. In _estimated_json_size, the dict and list branches now loop explicitly. They handle str, None and int values inline and recurse for every other type, so bool and str subclasses still get the same estimates. Backtrace and record location estimates now use the existing flat-dict _estimated_location_size. There were four edits, 32 changed lines in all.
> In my sandbox, hits_ir went from 1462429219 to 1341205850, a ratio of 0.9171. report_ir went from 1116745400 to 1117652378, a ratio of 1.0008, so the report path did not get faster. Outputs were identical, both builds were ok and the guard passed. This is my own measurement, not a verdict.
> I now think the per-value recursion in _estimated_json_size is a real cost on the hit path. Most of the values it sees are plain scalars.
> Next, I will wait for the independent replays. After that I will look at whether the report path has a similar estimate loop worth flattening.
>
> 2026-10-13, episode `2779233538b3` (accepted)

> In bridge/native/rea_lldb_tracer.py I changed _breakpoint_report in three ways. The selector-only method-type filter is now resolved once per spec instead of once per location. The location budget estimate now uses a flat-dict helper that handles str and None inline and falls back to _estimated_json_size for other types. The exact size is now len(json.dumps(...)) with no encode step, because ensure_ascii output is already ASCII. I also touched _location, where the module file is fetched, and added a helper near line 68.
> My sandbox run on report_ir was accepted, with identical outputs and 46 changed lines. report_ir went from 1214696134 to 1092276826, a ratio of 0.8992. hits_ir went from 1498932953 to 1487478536, a ratio of 0.9924. That is small but not zero.
> I now believe most of the report cost was repeated per-location work and size estimation, not the tracing itself.
> Next, I want to wait for the replays. After that I will profile the hits path separately, since it barely moved.
>
> 2026-10-13, episode `b93bb0a97906` (accepted)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
