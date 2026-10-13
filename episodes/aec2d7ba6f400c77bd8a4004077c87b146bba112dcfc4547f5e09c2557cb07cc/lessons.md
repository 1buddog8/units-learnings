# Lessons: rea

Lineage `aec2d7ba6f400c77bd8a4004077c87b146bba112dcfc4547f5e09c2557cb07cc` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 1 published attempt.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:report_ir

- Attempts: 1 (1 accepted)
- Best accepted effect: 10.08% (ratio 0.8992), episode `b93bb0a97906`
- Files changed by accepted candidates: `bridge/native/rea_lldb_tracer.py`
- Median model spend per attempt: 0.2763 USD (over 1 attempt with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> In bridge/native/rea_lldb_tracer.py I changed _breakpoint_report in three ways. The selector-only method-type filter is now resolved once per spec instead of once per location. The location budget estimate now uses a flat-dict helper that handles str and None inline and falls back to _estimated_json_size for other types. The exact size is now len(json.dumps(...)) with no encode step, because ensure_ascii output is already ASCII. I also touched _location, where the module file is fetched, and added a helper near line 68.
> My sandbox run on report_ir was accepted, with identical outputs and 46 changed lines. report_ir went from 1214696134 to 1092276826, a ratio of 0.8992. hits_ir went from 1498932953 to 1487478536, a ratio of 0.9924. That is small but not zero.
> I now believe most of the report cost was repeated per-location work and size estimation, not the tracing itself.
> Next, I want to wait for the replays. After that I will profile the hits path separately, since it barely moved.
>
> 2026-10-13, episode `b93bb0a97906` (accepted)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
