---
name: record-seeded-fault
description: Reproduce a bikeshare fault and record its test, hook, and repair evidence.
disable-model-invocation: true
---

# Record a seeded fault

Fault to reproduce: $ARGUMENTS

1. Find the matching entry in `docs/SEEDED_FAULTS.md` and identify the exact edit.
2. Use a temporary Git worktree and run `make test` before seeding the fault. If the baseline fails, stop and report the output.
3. Apply the seeded edit and save the diff and failing `make test` output. For a source edit, run the leakage hook with an Edit payload for the changed file and save its stderr and exit code. For the Bash copy fault, use a disposable file and record whether the copy succeeds and the hook fires.
4. Undo the seeded edit or remove the disposable file, then rerun `make test`.
5. Update the matching entry in `docs/SEEDED_FAULTS.md` with the actual output and repair. Label any direct hook invocation and record repair turns if observed.
