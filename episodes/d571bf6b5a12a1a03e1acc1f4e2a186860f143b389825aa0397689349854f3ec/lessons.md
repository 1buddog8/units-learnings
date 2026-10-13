# Lessons: morluto-rea

Lineage `d571bf6b5a12a1a03e1acc1f4e2a186860f143b389825aa0397689349854f3ec` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 27 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:codex_select_event_ir

- Attempts: 7 (3 accepted, 4 no candidate)
- Best accepted effect: 11.87% (ratio 0.8813), episode `468e974ddf79`
- Accepted effects: 11.87%, 10.81%, 1.65%
- Files changed by accepted candidates: `experiments/analysis-efficiency/codex_context.py`
- Median model spend per attempt: 0.4161 USD (over 7 attempts with a recorded cost)

## Target perf:codex_capture_ir

- Attempts: 11 (7 accepted, 4 no candidate)
- Best accepted effect: 6.56% (ratio 0.9344), episode `cdb3d8811d87`
- Accepted effects: 6.56%, 4.83%, 4.76%, 3.90%, 1.63%, 1.61%, 1.60%
- Files changed by accepted candidates: `experiments/analysis-efficiency/codex_context.py`
- Median model spend per attempt: 0.3851 USD (over 11 attempts with a recorded cost)

## Target perf:resource_observation_ir

- Attempts: 9 (5 accepted, 4 no candidate)
- Best accepted effect: 14.93% (ratio 0.8507), episode `ff27420a8738`
- Accepted effects: 14.93%, 3.41%, 2.90%, 2.49%, 1.28%
- Files changed by accepted candidates: `experiments/analysis-efficiency/resource_payload.py`
- Median model spend per attempt: 0.4516 USD (over 9 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked in experiments/analysis-efficiency/resource_payload.py on the resource_observation_ir bench. First I rewrote pairs to use map(str.split). Then I tried something else in read_observation. When the text has no escapes, it now rejects receipts whose quoted ownership token is absent, before any parsing.
> The pairs rewrite measured 2608937145 against a parent of 2601645535. That is a ratio of 1.0028, so it did not help. The pre-parse rejection measured 2595022524 against 2601645529, a ratio of 0.9975. The required ratio is 0.9900, so both were rejected. Outputs were identical in both runs. The guarded metrics stayed within limits, with codex_capture_ir at 1.0001 at worst.
> I now believe the per-call work left in read_observation is small next to the bench total. Trimming inside it will not reach the threshold. I submitted nothing.
> Next time I would read lineage_bench.py further to see where the rest of the instruction count goes. Then I would look for work outside read_observation that the bench repeats.
>
> 2026-10-13, episode `e1a47874e124` (no candidate)

> I worked on selected_event in experiments/analysis-efficiency/codex_context.py, aiming at codex_select_event_ir. First I replaced dict.get with a membership test plus subscript. Then, in the token-count and tool-item key loops, I replaced membership plus subscript with a bound get and a sentinel.
> Neither change helped. Both kept outputs identical. The first moved codex_select_event_ir from 2481534489 to 2566443999, a ratio of 1.0342. The second moved it from 2481534483 to 2677969753, a ratio of 1.0792. The required ratio was 0.9900, so both were rejected. codex_capture_ir and resource_observation_ir stayed within their guard limits both times. I reverted and submitted nothing.
> On this interpreter, the lookups in selected_event already use the cheapest form I could find. Swapping one lookup idiom for another only added cost.
> Next time I would stop tuning the lookups. I would read how selected_event walks its events and look for repeated work or a loop that can exit early.
>
> 2026-10-13, episode `f4758fbc1912` (no candidate)

> I tried a change in experiments/analysis-efficiency/codex_context.py, in the capture loop at lines 161-173. Each rollout file was decoded once and split on '\n', instead of decoding every line on its own. If the whole-file decode hit invalid UTF-8, the code fell back to decoding line by line.
> Outputs were identical, and the build and guard passed. codex_capture_ir went from parent 2967170421 to candidate 2992476137, a ratio of 1.0085 against the required 0.9900, so the change was rejected. codex_select_event_ir moved by a ratio of 1.0007, and resource_observation_ir by 1.0006. The change did not help, and I reverted it to a single line.
> The seeded inputs contain malformed lines. Many files fail the whole-file decode and then get decoded a second time. The per-line decode was never a large cost here.
> Next, profile capture_tool_context before editing to find where the instructions actually go. Look at the JSON parsing and the hashing rather than the decoding.
>
> 2026-10-13, episode `35126dc558b5` (no candidate)

> I changed read_observation in experiments/analysis-efficiency/resource_payload.py. The old code ran per-value loops to check that each value in memory.events, pids.events and cpu_stat was an int and non-negative. I replaced those loops with a set(map(type, ...)) type check plus min(). I also adjusted the imports at the top of the file.
> Outputs stayed identical and both builds passed. resource_observation_ir went from 2611810851 to 2790100985, a ratio of 1.0683. The required ratio was 0.9900, so the change was rejected. codex_capture_ir was at 1.0000 and codex_select_event_ir at 1.0001. The change did not help. I submitted nothing.
> On this interpreter the plain per-value loops are cheaper than building a set and calling min(). These dictionaries are small, so the extra C calls and the temporary set cost more than they save.
> Next time I would look at where read_observation spends its time before touching the checks. Parsing or file reads may matter more than the validation loops.
>
> 2026-10-13, episode `16870cdd1862` (no candidate)

> I worked in experiments/analysis-efficiency/resource_payload.py on read_observation. My first edit made resource_observation_ir worse. Evaluation 1 measured parent 2580432746 against candidate 2759549732, a ratio of 1.0694, and it was rejected against the required 0.9900. That change did not help. I reworked the function. read_observation now reads every required receipt field with a plain subscript instead of dict.get followed by a None check. A missing key raises KeyError. The existing except clause catches it and returns None, as before. ownership_token still uses get.
> Evaluation 3 measured candidate 2548206137 against parent 2580432746, a ratio of 0.9875, and it was accepted. Outputs were identical. codex_capture_ir was 0.9999 and codex_select_event_ir was 1.0000. The diff is 28 lines.
> I believe the cost in this function sits in per-field lookups, and the gain is small.
> Next I will check whether the except path is ever hot. I will also profile the rest of the observe path. The candidate is queued for replay.
>
> 2026-10-13, episode `27d8d758cc4a` (accepted)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
