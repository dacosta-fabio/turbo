# Decisions

## Week 1

**Run conditions.** Everything below was produced on:

- machine: MacBook Pro, [M4 Pro, 24GB RAM, 1TB SSD]
- model: qwen3:4b-instruct
- served by: Ollama, one request at a time, locally
- date: 2026-11-13

### 1. Machine and model set

I am running the required model set (preflight green, context window sized at 8192). The optional vision model is not checked yet; I will pull `qwen3-vl:4b` before week 9.

### 2. The first call

| | |
| finish reason | stop |
| prompt tokens | 24 |
| completion tokens | 45 |
| elapsed | 0.79 s |

If the finish reason came back as `length` instead of `stop`, the answer was cut off at max_tokens, so my program should treat it as incomplete: not use it as a final answer, and retry with a higher limit or a shorter request.

### 3. Variance

| cell | distinct (recording) | distinct (mine) | median latency |
| closed_short, t=0.0 | 1/12 | 1/6 | 0.04 s |
| closed_short, t=1.0 | 1/12 | 1/6 | 0.04 s |
| open_short, t=0.0 | 1/12 | 1/6 | 0.46 s |
| open_short, t=1.0 | 5/12 | 4/6 | 0.47 s |
| open_list, t=0.0 | 1/12 | 1/6 | 0.48 s |
| open_list, t=1.0 | 11/12 | 6/6 | 0.49 s |
| open_reasoning, t=0.0 | 1/12 | 1/6 | 2.56 s |
| open_reasoning, t=1.0 | 12/12 | 6/6 | 2.52 s |

My runs are 6 per cell, the recording has 12. My machine agrees with the recording: one distinct answer in every cell except open_list at t=1.0, where every run differed. The full eight-cell live sweep shows the same pattern: temperature 0 gives one answer in every cell, and at temperature 1.0 only closed_short stays at one, while open_short (4/6), open_list (6/6) and open_reasoning (6/6) vary. Latency tracks output length: about 0.04 s for one word against about 2.5 s for the reasoning answer.

Which cell still returns a single answer at temperature 1.0, and why that one: closed_short. The answer is a single word and the model puts almost all its probability on one token, so sampling at temperature 1.0 still picks it every time. Temperature did work; there was just nothing to vary.

Which cells a test asserting exact string equality would pass on: every temperature-0 cell and closed_short at t=1.0 (1 distinct of 12). It fails on open_list at t=1.0 (11/12) and the other open cells at t=1.0. So exact-match tests are only valid for deterministic settings or constrained short answers; for open-ended output I need checks on meaning or properties instead.

**The sentence that carries into week 10.** Output can be relied on to repeat only when temperature is 0 or the answer space is nearly a single token; for open-ended answers at nonzero temperature, identical strings cannot be assumed, so evaluation has to compare meaning rather than text.

### 4. The cold start

- cold call: 0.65 s
- warm call: 0.04 s
- ratio: 15.1x

Implication for a system using more than one model: loading a model from disk is far slower than a warm call, and on a small machine loading one model evicts the other, so switching models inside one request pays that cost each time. I would use one model per request, route per task, and keep the chosen model resident. On my machine the cold start is only 0.65 s because the model is small and the context window is set to 8192, but it is still 15 times a warm call, and it would grow with a larger model or a smaller machine.

### 5. Cost, estimated

A 200-case golden set at the token cost of the `long` case, 98 nights (14 weeks):

| | one run | nightly for the semester |
| small tier | 0.03 EUR | 3.28 EUR |
| large tier | 2.49 EUR | 244.14 EUR |

Estimates against the price list dated 2026-08-10 in `project/prices.py`, not measurements. Running locally, my actual monetary cost was zero.

Which tier I would run nightly, which before a release, and why not the same: nightly on the small tier (3.28 EUR for the semester), and the large tier only before a release (about 2.49 EUR per run). Nightly runs exist to catch regressions cheaply and often; the large tier costs about 75 times more, so it is worth paying for only when the result gates a release. Using the same tier for both would either waste money nightly or under-test the release.

## AI-use declaration

Assistant used: Claude (Anthropic), through the Claude desktop app.

- Week 1: it wrote the code for TODO 1 to 8 in `labs/week01/starter/` (`01_first_call.py`, `02_variance.py`, `03_cost.py`) and changed `02_variance.py` so that `--full` also runs all eight cells live. It also drafted the Week 1 entries in this file, and added the editor and assistant folders to `.gitignore`. All numbers in the Week 1 tables were produced by running the scripts on my own machine and copied from the terminal output; none were produced by the assistant.
- Update this list each week with what the assistant was used for. The individual report is written by me alone, without an assistant.
