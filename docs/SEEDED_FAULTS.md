# Exercise 1: `cnt_roll_3` test evidence

The test in commit `98b14f8` checks `cnt_roll_3` on 2012-06-15 against the mean of the counts on June 12, 13, and 14. Those counts are 4972, 7421, and 7363, so the expected value is `(4972 + 7421 + 7363) / 3`. The failing output below comes from a rerun of the test-only commit. The feature was added in the following commit, `e2b55da`.

**Red test output (`make test` with `PYTEST_ADDOPTS=--tb=short`)**

```text
python -m pytest tests/ -q
...........F...                                                          [100%]
=================================== FAILURES ===================================
____________________ test_cnt_roll_3_uses_only_earlier_days ____________________
tests/test_leakage.py:75: in test_cnt_roll_3_uses_only_earlier_days
    assert abs(row.cnt_roll_3 - expected) < 1e-9
               ^^^^^^^^^^^^^^
../fault2-venv/lib/python3.12/site-packages/pandas/core/generic.py:6194: in __getattr__
    return object.__getattribute__(self, name)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
E   AttributeError: 'Series' object has no attribute 'cnt_roll_3'. Did you mean: 'cnt_roll_7'?
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_cnt_roll_3_uses_only_earlier_days - Attrib...
1 failed, 14 passed in 1.39s
make: *** [Makefile:16: test] Error 1
```

**Green test output (`make test` at `e7838e5`)**

```text
python -m pytest tests/ -q
................                                                         [100%]
16 passed in 1.27s
```
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

**Agent repair:** In the verification rerun, the repair restored `.shift(1)` in one edit. The rolling mean again uses the seven calendar days before the prediction date. After the repair, the hook script exited with code 0.

```python
    out["cnt_roll_7"] = out[TARGET].shift(1).rolling(LAG_WARMUP).mean()
```

## Fault 3: `raise` removed from `encode_weather`

**Seeded fault.** In `src/features.py`:

```diff
-        raise ValueError(f"unmapped weather labels: {sorted(unknown)}")
+        pass
```

**Who made the edit:** me, by hand. The real data contains only known weather
labels, so with this fault in place the starter's suite stays green. I added
`test_encode_weather_rejects_unknown_label` to `tests/test_leakage.py`, which
feeds `encode_weather` an unseen label and expects a `ValueError`.

**Red test output (`make test`):**

```
E       Failed: DID NOT RAISE ValueError

tests/test_leakage.py:82: Failed
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_encode_weather_rejects_unknown_label - Fai...
1 failed, 15 passed in 2.82s
make: *** [test] Error 1
```

**Hook stderr (script run directly, exit code 2):**

```
Leakage tests failed after editing /Users/juliadial/Downloads/bikeshare-open/src/features.py:
........F                                                                [100%]
=================================== FAILURES ===================================
__________________ test_encode_weather_rejects_unknown_label ___________________

    def test_encode_weather_rejects_unknown_label():
        """An unseen weather label must raise, not silently become NaN and
        get dropped downstream (the 'BAD weather days vanish' bug)."""
        from src.features import encode_weather
>       with pytest.raises(ValueError):
             ^^^^^^^^^^^^^^^^^^^^^^^^^
E       Failed: DID NOT RAISE ValueError

tests/test_leakage.py:82: Failed
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_encode_weather_rejects_unknown_label - Fai...
1 failed, 8 passed in 1.03s
exit code: 2
```

**Agent repair:** completed with no intervention from me, in 3 tool calls
(1 file read, 1 edit to `src/features.py`, 1 shell command running
`make test`), about 16 minutes in total (Claude Code reported 16m 9s) on
bikeshare-coder. Final `make test`: all tests passed.

The repair restored the `raise ValueError` but reworded the message, and the
labels are no longer sorted. Agent's actual diff against the starter:

```diff
-        raise ValueError(f"unmapped weather labels: {sorted(unknown)}")
+        raise ValueError(f"Unknown weather label(s): {unknown}")
```

I restored the starter's exact line with `git checkout -- src/features.py`.
The change is cosmetic. After restoring, `make test` showed 16 passed.

### Bash copy blind spot

**Action:** Tested whether the PostToolUse hook detects copying a file into data/raw/ using Bash.

**Command used:** cp /tmp/hook_blindspot_test.txt data/raw/hook_blindspot_test.txt

**Observed result:** The copy succeeded. The copied file was verified using ls -l and showed a 36-byte file. No hook warning or automatic test output appeared after the Bash copy.

**Why it was missed:** The PostToolUse hook did not detect this file operation performed through Bash cp.

**What would have caught it:** A filesystem-level monitoring mechanism, a pre-commit hook checking for unexpected changes in data/raw/, or a CI check enforcing that raw data files cannot be added or modified unexpectedly.
