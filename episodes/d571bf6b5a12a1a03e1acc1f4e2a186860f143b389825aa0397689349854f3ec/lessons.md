# Lessons: morluto-rea

Lineage `d571bf6b5a12a1a03e1acc1f4e2a186860f143b389825aa0397689349854f3ec` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 19 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:codex_select_event_ir

- Attempts: 6 (3 accepted, 3 no candidate)
- Best accepted effect: 11.87% (ratio 0.8813), episode `468e974ddf79`
- Accepted effects: 11.87%, 10.81%, 1.65%
- Files changed by accepted candidates: `experiments/analysis-efficiency/codex_context.py`
- Median model spend per attempt: 0.4781 USD (over 6 attempts with a recorded cost)

## Target perf:codex_capture_ir

- Attempts: 9 (7 accepted, 2 no candidate)
- Best accepted effect: 6.56% (ratio 0.9344), episode `cdb3d8811d87`
- Accepted effects: 6.56%, 4.83%, 4.76%, 3.90%, 1.63%, 1.61%, 1.60%
- Files changed by accepted candidates: `experiments/analysis-efficiency/codex_context.py`
- Median model spend per attempt: 0.3763 USD (over 9 attempts with a recorded cost)

## Target perf:resource_observation_ir

- Attempts: 4 (2 accepted, 2 no candidate)
- Best accepted effect: 14.93% (ratio 0.8507), episode `ff27420a8738`
- Accepted effects: 14.93%, 2.49%
- Files changed by accepted candidates: `experiments/analysis-efficiency/resource_payload.py`
- Median model spend per attempt: 0.4493 USD (over 4 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked on selected_event in experiments/analysis-efficiency/codex_context.py, targeting codex_select_event_ir. I read the file and the bench section of lineage_bench.py. Then I tried two micro-optimisations. The first used dict comprehensions. The second used only exact-type fast paths in the validation.
> The comprehension version was rejected. codex_select_event_ir went from 2793206629 to 3008968014, ratio 1.0772, so it got worse. On this Python each comprehension creates its own function, and that costs more than it saves. The fast-path version was also rejected. It measured 2793206635 to 2786780124, ratio 0.9977, against a required 0.9900. Outputs were identical in both runs and the guards were ok.
> The validation in selected_event is already cheap. Much of the benchmark's instruction count seems to sit outside it. I submitted no candidate.
> Next time I will profile where codex_select_event_ir actually spends its instructions before editing anything. I should look at the bench setup and the parsing around response_item and token_count, not selected_event itself.
>
> 2026-10-13, episode `55d5d102041e` (no candidate)

> I worked on capture_tool_context in experiments/analysis-efficiency/codex_context.py, the line loop that feeds codex_capture_ir. I tried two behaviour-preserving changes. First, I collected each source's encoded events and wrote them once per source. Second, I split the file data with bytes.split instead of iterating a BytesIO, and skipped the trailing strip when the scanner ends at end of line.
> Both kept outputs identical and both made the target worse. The first change measured codex_capture_ir at 3233500370 against a parent of 3181776031, a ratio of 1.0163. The second measured 3309981653 against 3181786771, a ratio of 1.0403. The required ratio was 0.9900. Both were rejected and I submitted no candidate.
> So batching the writes did not help, and neither did splitting into a list. The BytesIO iteration seems cheaper than I assumed. The per-line cost is probably somewhere else in the loop.
> Next time, I should measure the scanner and the encoder separately before touching the loop structure again.
>
> 2026-10-13, episode `fe53f5d8e396` (no candidate)

> I worked on read_observation in experiments/analysis-efficiency/resource_payload.py, targeting resource_observation_ir. I rewrote its validation twice. The first rewrite replaced the per-value loops with set(map(type, ...)) and min(). The second used keys-view superset checks and a single control_group lookup.
> Neither helped. The first gave resource_observation_ir 3077272482 against a parent of 2819360981, ratio 1.0915. The second gave 2834125883 against 2819360975, ratio 1.0052. The required ratio was 0.9900. Both builds passed, outputs were identical, and the guarded metrics stayed within limits. I submitted no candidate.
> The short bytecode loops in read_observation are already cheaper than C-level helpers that allocate. I believe most of the remaining cost is JSON scanning and the bench harness in lineage_bench.py, not the validation.
> Next time, profile the JSON parsing step and the bench's own overhead before touching validation again.
>
> 2026-10-13, episode `010b20f062ca` (no candidate)

> I worked in experiments/analysis-efficiency/resource_payload.py on read_observation. Each receipt went through JSONDecoder.decode, which wraps the C scanner in two Python frames, decode and raw_decode. I tried several rewrites of that path.
> The first attempt was rejected. resource_observation_ir went from 2902205180 to 3145115076, ratio 1.0837, so it made things worse. A smaller 10-line change was accepted at ratio 0.9884. The submitted 23-line version calls the decoder's scan_once and the json whitespace matcher directly. It was accepted in my sandbox at 2827118258 against 2902205186, ratio 0.9741, with identical outputs. The guarded metrics moved by at most ratio 1.0002. It is queued for independent replay.
> I now believe the per-receipt Python frame cost dominates this metric more than the parsing itself. The acceptance rules still need care: whitespace around the value, trailing data rejected, StopIteration treated as malformed.
> Next, I would look at whether the receipt loop around read_observation can drop frames the same way.
>
> 2026-10-13, episode `8f13b917f33c` (accepted)

> I changed _event_encoder in experiments/analysis-efficiency/codex_context.py. The cached C encoder was built with a markers dict, so every dict and list encoded per retained event ran a circular-reference check. Retained events come only from values decoded from JSON text, and those cannot be circular. The encoder now uses check_circular=False and passes markers=None to c_make_encoder. The edits touched lines 101-102 and line 111.
> In my sandbox run the outcome was accepted and the outputs were identical. codex_capture_ir went from 3062998717 to 3008485869, a ratio of 0.9822. codex_select_event_ir was 0.9997 and resource_observation_ir was 1.0028. Both stayed within the guard limits. The candidate is submitted and waiting for independent replays.
> The gain is small but real. The circular check was paying for safety that this data never needs.
> Next, I want to look at what else runs per retained event in the encode path. The search for capture was too broad, with 6605 matches, so I should narrow it to the hot loop.
>
> 2026-10-13, episode `17a2b3d428f1` (accepted)

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
