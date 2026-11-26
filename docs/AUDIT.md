# Exercise 3: Teammate Audit

**Repository audited:** https://github.com/xibahbah/bikeshare-open

## How I ran the audit

From the root of my own `bikeshare-open` repository, I ran:

```bash
git clone https://github.com/xibahbah/bikeshare-open.git ../teammate-bikeshare-open
claude --add-dir ../teammate-bikeshare-open
```

In Claude Code, I entered:

```text
Use the leakage-auditor subagent to audit ../teammate-bikeshare-open. Report only findings that change a reported number or invalidate a conclusion. If the audit completes and finds no issues, reply exactly: No findings to report. Do not modify the repository.
```

## `/leakage-auditor` output

```text
No findings to report.
```

## My verdict

I agree with the auditor’s conclusion. It found no issues that would change the reported model scores or invalidate the comparison with the baselines. No findings to report.

## Part B: Self-audit with `agent/audit.py`

**Command:** `AUDIT_MODEL=bikeshare-coder python agent/audit.py . --out reports/audit.jsonl`

**Setup:** I ran `bikeshare-coder` (`qwen3.5:4b`, 64k context) through local Ollama on an 8 GB MacBook Air.

**Result:** The run did not complete. My Mac became unresponsive, so I stopped the job with `kill %1` at 9:56:40 p.m. EST on November 25, 2026. The script writes its results at the end of the run, so this attempt did not produce `reports/audit.jsonl`.

**What I observed:**

- One `POST /v1/messages` request returned HTTP 500 after 5 minutes and 50 seconds.
- The Ollama log showed cache misses, including `total=15354 matched=0`. One request took roughly six minutes to process about 11,000 tokens.
- `make check-model` confirmed the 65,536-token context and warned that the model answered without calling the test tool.

**Likely cause:** The 64k context and repeated processing made the audit too resource-intensive for my Mac to finish reliably. I also tried Ollama Cloud, but those attempts did not complete successfully.

**Evidence saved locally:** `audit_log.txt`, `ollama_server_tail.txt`, `ollama_errors.txt`, and `audit_stopped_at.txt`.
