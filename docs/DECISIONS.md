# Decisions

One line per methodological decision: what you chose, why, and what it changed.
Agents append here via the `/new-feature` skill; you append here by hand.

- Exclude `casual`, `registered`: they sum to `cnt` exactly. Leaky model R^2 1.000 -> honest ridge 0.583.
- Exclude `cnt_2d_bfr`: a shipped 2-day lag we cannot test the alignment of. Build our own lags instead.
- Exclude `instant`, `yr`, `days_since_2011`: time indexes let a tree predict a flat constant past 2011 and a linear model extrapolate a trend we have no grounds for.
- Split 2011 / H1 2012 / H2 2012 by date. Test set touched once, after all feature work.
- Report mean and persistence baselines with every score. Persistence (test R^2 0.510, MAE 840) is within 0.07 R^2 of ridge and beats it on MAE; that caveat goes in the report.
- 2026-11-25: Step 2 data check (model: bikeshare-coder, built from qwen3.5:4b via local Ollama, 4.5B parameters, 64k context; gpt-oss:20b-cloud was tried first but returned 402 errors and did no work). Verified agent's summary against pandas. Right: 728 rows, 2011-01-03 to 2012-12-31, casual/registered identified as leakage. Wrong: listed 3 missing dates (only 2011-03-10 is a gap in range); omitted the GOOD/MISTY/BAD weather labels (463/245/20); filed time indexes and cnt_2d_bfr under "leakage" (a bad bet and an unverified lag, not leakage); gave the temporal-split extrapolation problem as the reason against a random split instead of adjacent-day similarity. Kept the temporal split from CLAUDE.md.
