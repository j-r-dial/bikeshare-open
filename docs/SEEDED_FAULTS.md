# Exercise 2: Seeded Faults

Model used for agent repairs: bikeshare-coder (qwen3.5:4b via local Ollama, 64k context).

## Fault 1: forbidden column reaches the feature set (`casual`)

**Seeded fault.** Removed the forbidden-column filter from `feature_columns()`
in `src/features.py` (lines 83-85), so `casual` was included in the feature
columns.

**Who made the edit:** the agent (Claude Code), so the PostToolUse hook fired.

**Red test output (`make test`):**

```
FAILED tests/test_leakage.py::test_no_forbidden_features
E   AssertionError: forbidden columns in feature set: {'casual'}
1 failed, 14 passed in 4.90s
make: *** [test] Error 1
```

**Hook stderr (PostToolUse:Edit):**

```
PostToolUse:Edit hook reported a problem
Leakage tests failed after editing src/features.py

AssertionError: forbidden columns in feature set: {'casual'}

FAILED tests/test_leakage.py::test_no_forbidden_features
1 failed, 7 passed in 1.33s
```

**Agent repair:** 1 repair prompt/turn. Restored the filter:

```python
return [c for c in numeric if c not in FORBIDDEN]
```

All 8 tests in `test_leakage.py` passed afterward.

## Fault 2: `.shift(1)` deleted from `cnt_roll_7`

**Seeded fault.** In `src/features.py`:

```diff
-    out["cnt_roll_7"] = out[TARGET].shift(1).rolling(LAG_WARMUP).mean()
+    out["cnt_roll_7"] = out[TARGET].rolling(LAG_WARMUP).mean()
```

Without the shift, the rolling mean includes the same day's count.

**Who made the edit:** me, by hand with sed. The hook only fires on the
agent's edits, so I ran `.claude/hooks/check_leakage.sh` directly with a
PostToolUse-style JSON payload for `src/features.py`.

**Red test output (`make test`):**

```
FAILED tests/test_leakage.py::test_rolling_mean_excludes_today
E   assert np.float64(4389.0) == 4414.857142857143 ± 0.00441486
E     Obtained: 4389.0
E     Expected: 4414.857142857143 ± 0.00441486
1 failed, 14 passed in 3.20s
make: *** [test] Error 1
```

**Hook stderr (script run directly, exit code 2):**

```
Leakage tests failed after editing /Users/juliadial/Downloads/bikeshare-open/src/features.py:
    def test_rolling_mean_excludes_today(feats):
        """cnt_roll_7 must be the mean of the 7 calendar days *before* today."""
        ...
>           assert feats.set_index("dteday").cnt_roll_7[date] == pytest.approx(expected)
E           assert np.float64(4389.0) == 4414.857142857143 ± 0.00441486
E             Obtained: 4389.0
E             Expected: 4414.857142857143 ± 0.00441486

tests/test_leakage.py:57: AssertionError
FAILED tests/test_leakage.py::test_rolling_mean_excludes_today - assert np.fl...
1 failed, 7 passed in 0.86s
exit code: 2
```

**Agent repair:** agent did not finish repair, time out error. restored line with 'git checkout --src/features.py
