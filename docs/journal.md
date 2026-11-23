# Project Journal

A running log of work on this repo and a tracker of known issues. Add new log
entries at the top. Never delete an issue: when it is fixed, change its status
and note how it was fixed.

---

## Log

### 2026-11-22: README rewrite and code-path audit

- Rewrote `README.md` to the standard layout: overview, how it works,
  structure, setup, usage, data format, constraints, reproduction. Added the
  Ollama backend, interactive CLI, retries and logging, which were all
  undocumented.
- Checked the "How it works" diagram by running `invoke_agent` with a stubbed
  `call_model` (no API calls) across 9 cases. Fixed 3 errors in the diagram
  (see the README history).
- Ran the router and normalizer over the 1,000-question dev set. This turned
  up most of the issues below.
- No code changes yet. All issues are still open.

---

## Known issues

Severity: **High** = likely costs accuracy or a whole run · **Med** = wrong in
some cases · **Low** = robustness or cleanup.
Evidence: **Repro** = reproduced by running code · **Data** = checked against
the dev set · **Read** = from reading the code, not yet run.

| ID | Sev | Area | Issue | Evidence | Status |
|---|---|---|---|---|---|
| 1 | High | `normalize_answer.py:11-14` | Coding answers lose their leading indentation: `"    return x + 1"` becomes `"return x + 1"`. If the grader appends the answer to the starter code, the result likely won't parse. | Repro | Open |
| 2 | High | Answer format | 100 common-sense dev labels are JSON booleans (`true`/`false`), but the agent outputs the strings `"yes"`/`"no"`. Exact match never succeeds. Need to confirm what the real grader expects. | Data | Open |
| 3 | High | `normalize_answer.py` (math) | Only the last number is kept: `\frac{3}{4}` becomes `4`. 41 math dev answers aren't integers (fractions, `\sqrt`, complex numbers, tuples, `x=5`). | Repro + Data | Open |
| 4 | High | Router + MCQ normalizer | The MCQ regex matches labels like "point D." in geometry text: 23 dev questions route to `mcq` (20 common-sense, 3 math), and none sampled has a letter answer. The normalizer then reduces answers to a letter: `"Washington, D.C."` becomes `D`, `"Vitamin B"` becomes `B`. The MCQ prompt also asks for answer text while the normalizer extracts a letter. | Repro + Data | Open |
| 5 | High | `generate_answers.py` `is_done` | `"unknown"` (the value written on errors) counts as done, so resuming a run never retries failed questions. | Read | Open |
| 6 | Med | Router | Dev routing mismatches: 197 common-sense → `general` (single pass, no self-consistency), 22 common-sense → `math`, 14 math → `general`. | Data | Open |
| 7 | Med | `normalize_answer.py` (yes/no) | Substring match: `"I don't know"` becomes `no` (the "no" in "know"). | Repro | Open |
| 8 | Med | `normalize_answer.py` (future) | Nested braces break `\boxed{}` extraction: `\boxed{\text{Yes}}` becomes `['\text{Yes']`. | Repro | Open |
| 9 | Med | `evaluate_answers.py` | Exact match marks correct GSM8K answers wrong, because 100 math labels are full solutions ending in `#### N`. Local accuracy is understated. | Data | Open |
| 10 | Med | `evaluate_answers.py` | Default paths point at `data/{input,output}/eval.json`, which don't exist. Results are written to `data/eval/`, which also doesn't exist, so the write will fail. | Read | Open |
| 11 | Low | `client.py` `call_model` | Returns `None` after all retries fail (the declared type is `str`). The failure is silent and wastes a retry call. Bad JSON or a `KeyError` in the response isn't retried. | Read | Open |
| 12 | Low | `client.py` (Ollama) | The `timeout` argument is ignored; Ollama always uses 180 s. | Read | Open |
| 13 | Low | `agent_logger.py` | `agent.log` is overwritten each run. With 8 threads, entries interleave and have no question ID. Errors are logged under the `OUTPUT/ASSISTANT` label. | Read | Open |
| 14 | Low | `generate_answers.py` | If the existing output file's length doesn't match the input, all progress is silently discarded and overwritten. | Read | Open |
| 15 | Low | `tools.py` | The regex allows `**`, so a model-generated expression like `9**9**9` can hang a worker. | Read | Open |
| 16 | Low | Graph | Calculator results skip the malformed check (by design, but noted). | Repro | Open |
| 17 | Low | Deps | `requirements.txt` pins about 35 packages, but `src/` only imports `requests` and `python-dotenv` (as `pyproject.toml` says). The two files have drifted apart. | Read | Open |
| 18 | Low | Cleanup | `generate_answers.py` docstring still describes a placeholder generator, and `from importlib.resources import path` is unused. | Read | Open |
| 19 | Low | Docs | The README's Mermaid diagram hasn't been checked on GitHub, since no renderer was available locally. | — | Open |
