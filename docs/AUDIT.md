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
No findings to report
```

## My verdict

I agree with the auditor’s conclusion. It found no issues that would change the reported model scores or invalidate the comparison with the baselines. No findings to report.


## Part B: `agent/audit.py` on my own repo

**Command:** `AUDIT_MODEL=bikeshare-coder python agent/audit.py . --out reports/audit.jsonl`

**Model and machine:** bikeshare-coder (qwen3.5:4b, 4.5B parameters, 64k
context) via local Ollama on an 8 GB MacBook Air.

**Result: the run did not complete, so `reports/audit.jsonl` does not exist.**
I stopped it with `kill %1` at 21:56:40 EST on 2026-11-25, after about NN
minutes. The script writes its output file only at the end of the run, so
nothing was written. I did not create the file by hand.

**What the Ollama server log showed:**

- A `POST /v1/messages` request returned HTTP 500 after 5m50s.
- Each turn re-processed the whole conversation from scratch
  (`cache miss ... total=15354 matched=0`), about 11k to 15k tokens per turn.
  Processing about 11k tokens took roughly 6 minutes.
- `make check-model` also warned that the model does not advertise tool
  support and answered without calling its test tool.

**Why I think it stalled:** a 4.5B model with a 64k context on 8 GB of RAM
could not finish a tool-using audit of up to 40 turns in reasonable time.

**Hosted alternative:** the Ollama cloud models I tried returned HTTP 402, so
I could not use one.

**Evidence kept locally (not committed):** `audit_log.txt`,
`ollama_server_tail.txt`, `ollama_errors.txt`, `audit_stopped_at.txt`.
