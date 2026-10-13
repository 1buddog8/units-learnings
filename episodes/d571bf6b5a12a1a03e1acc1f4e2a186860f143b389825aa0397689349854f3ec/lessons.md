# Lessons: morluto-rea

Lineage `d571bf6b5a12a1a03e1acc1f4e2a186860f143b389825aa0397689349854f3ec` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 3 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:codex_select_event_ir

- Attempts: 2 (2 accepted)
- Best accepted effect: 11.87% (ratio 0.8813), episode `468e974ddf79`
- Accepted effects: 11.87%, 10.81%
- Files changed by accepted candidates: `experiments/analysis-efficiency/codex_context.py`
- Median model spend per attempt: 0.3714 USD (over 2 attempts with a recorded cost)

## Target perf:codex_capture_ir

- Attempts: 1 (1 accepted)
- Best accepted effect: 3.90% (ratio 0.9610), episode `8efccf3dc99d`
- Files changed by accepted candidates: `experiments/analysis-efficiency/codex_context.py`
- Median model spend per attempt: 0.1485 USD (over 1 attempt with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I read codex_context.py and lineage_bench.py to find what codex_select_event_ir spends its time on. In token_counts, every count went through two isinstance calls: isinstance(count, int) and not isinstance(count, bool). I changed it to check type(count) is int first. When that holds, only the count < 0 check runs. Other types fall through to the original isinstance checks, so int subclasses are still accepted and bool is still rejected.
> My sandbox run accepted it with identical outputs. The guard reported ok, 1 files, 3 lines. codex_select_event_ir went from 2690216935 to 2399346311, ratio 0.8919. codex_capture_ir went from 3757537738 to 3711898289, ratio 0.9879. resource_observation_ir stayed flat at ratio 1.0000.
> I now believe per-count type validation is a real cost on this path, because plain ints dominate the input. The candidate is submitted and waiting on independent replays.
> Next, I would look in codex_context.py for other per-item checks that could take the same exact-type fast path.
>
> 2026-10-13, episode `36bf76f18d98` (accepted)

> I tried to cut work in capture_tool_context in experiments/analysis-efficiency/codex_context.py. It called json.dumps(selected, ensure_ascii=False) for every retained event. Because ensure_ascii is not the default, each call built a new JSONEncoder. I now build one JSONEncoder(ensure_ascii=False) per capture and reuse its encode method. I also compute str(kind) once per event instead of twice when counting. The change is 7 lines in one file.
> My sandbox run accepted it with identical outputs. codex_capture_ir went from 3945213480 to 3785466958, ratio 0.9595. codex_select_event_ir and resource_observation_ir stayed at ratio 1.0000. Every other guarded metric was within its limit. It is submitted and queued, and independent replays will decide.
> I now believe per-call encoder construction is a real cost in this capture path. Small allocations repeated per event add up here.
> Next, I will look for other json.dumps calls with non-default arguments inside per-event loops in this file and its neighbours.
>
> 2026-10-13, episode `8efccf3dc99d` (accepted)

> I worked on selected_event in experiments/analysis-efficiency/codex_context.py. It now reads payload, type and timestamp from the event once each and returns directly from each branch. The selected payload is built once, not built and then filtered again by a second comprehension over the same keys. In the token-count paths I replaced dict comprehensions with plain loops over constant tuples. The guard counted 69 changed lines in 1 file.
> My sandbox run accepted it with identical outputs. codex_select_event_ir went from 3381642452 to 2990940338, ratio 0.8845. codex_capture_ir moved from 3971840498 to 3911702461, ratio 0.9849. resource_observation_ir stayed flat at ratio 0.9999. The candidate is submitted and queued for independent replays.
> I now think most of the cost in this function came from repeated dict lookups and the double pass over payload keys, not from the validation logic.
> Next I would profile the rest of codex_capture, since its ratio moved much less than selection's.
>
> 2026-10-13, episode `468e974ddf79` (accepted)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
