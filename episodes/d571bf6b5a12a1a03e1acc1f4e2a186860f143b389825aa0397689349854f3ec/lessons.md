# Lessons: morluto-rea

Lineage `d571bf6b5a12a1a03e1acc1f4e2a186860f143b389825aa0397689349854f3ec` on [https://github.com/morluto/rea](https://github.com/morluto/rea) at `fb28ec1537e2d02aaca96325483c849c5c8ade6f`. 29 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:codex_select_event_ir

- Attempts: 8 (4 accepted, 4 no candidate)
- Best accepted effect: 11.87% (ratio 0.8813), episode `468e974ddf79`
- Accepted effects: 11.87%, 10.81%, 1.65%, 1.15%
- Files changed by accepted candidates: `experiments/analysis-efficiency/codex_context.py`
- Median model spend per attempt: 0.4781 USD (over 8 attempts with a recorded cost)

## Target perf:codex_capture_ir

- Attempts: 12 (8 accepted, 4 no candidate)
- Best accepted effect: 49.46% (ratio 0.5054), episode `123d36434c8b`
- Accepted effects: 49.46%, 6.56%, 4.83%, 4.76%, 3.90%, 1.63%, 1.61%, 1.60%
- Files changed by accepted candidates: `experiments/analysis-efficiency/codex_context.py`
- Median model spend per attempt: 0.3967 USD (over 12 attempts with a recorded cost)

## Target perf:resource_observation_ir

- Attempts: 9 (5 accepted, 4 no candidate)
- Best accepted effect: 14.93% (ratio 0.8507), episode `ff27420a8738`
- Accepted effects: 14.93%, 3.41%, 2.90%, 2.49%, 1.28%
- Files changed by accepted candidates: `experiments/analysis-efficiency/resource_payload.py`
- Median model spend per attempt: 0.4516 USD (over 9 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked in experiments/analysis-efficiency/codex_context.py on selected_event. It called the private helper _select_token_counts once for every token_usage_record and token_count event, so each of those events paid for one Python call frame. I moved that loop inline. The two branches now set up their source mapping and key tuple and share one copy of it. Key order, checks and error messages are unchanged.
> The first attempt did not help. codex_select_event_ir went from 2590269871 to 2799625434, ratio 1.0808, and was rejected. The second reached 0.9914, still short of the required 0.9900. The third measured 2555469088, ratio 0.9866, and was accepted in my sandbox. Outputs were identical every time, and codex_capture_ir and resource_observation_ir stayed within their limits.
> I now think call overhead in this path is real but small, and the shape of the inlined loop decides whether it pays off.
> Next I would wait for the replays, then look at the other per-event branches in selected_event for the same helper-call pattern.
>
> 2026-10-13, episode `f4bdc96de979` (accepted)

> I worked in experiments/analysis-efficiency/codex_context.py, in capture_tool_context, after reading lineage_bench.py to see what codex_capture_ir measures. Each rollout line's outcome depends only on its bytes. That outcome is a parse error reason, or an event type count plus the encoded selected event. So I added a per-source dict keyed by line bytes. A repeated line in the same file reuses its stored outcome. It is not decoded, run through selected_event and re-encoded again.
> My first attempt, 7 lines, did not help. codex_capture_ir went from 2923717277 to 2973886774, ratio 1.0172, and was rejected. The 25 line version was accepted in my sandbox. codex_capture_ir went from 2923717273 to 1464534223, ratio 0.5009. The other two metrics stayed near 1.0003. Outputs were identical.
> I now believe the capture loop does a lot of repeated work on duplicate lines, and caching by raw bytes removes most of it.
> Next I will wait for the replay verdict on the queued candidate. Then I will check whether the cache's memory cost matters on large files.
>
> 2026-10-13, episode `123d36434c8b` (accepted)

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

## Licence

Code excerpts from https://github.com/morluto/rea at fb28ec1537e2d02aaca96325483c849c5c8ade6f, copyright its authors; the licence was not determined (see the repository).
