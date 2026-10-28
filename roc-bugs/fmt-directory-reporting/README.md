# `roc fmt --check DIRECTORY` corrupts the failing-file list

Compiler: `nightly-2026-09-04-c125b82`

Run from the repository root:

```console
roc --opt=dev roc-bugs/fmt-directory-reporting/repro.roc
```

The temporary directory contains two valid but unformatted files named
`Alpha.roc` and `Beta.roc`, so the diagnostic should list those two names.
Instead, this nightly reports `Alpha.ro` and `Alpha.roc`; with a larger set of
unformatted files, the repeated names can also contain embedded NUL bytes.
The pinned September 7 compiler reproduces the same output on Apple silicon
macOS. On Ubuntu 24.04, that compiler instead reports `Beta.roc` followed by
a NUL byte, then `Beta.roc` again, omitting `Alpha.roc`. The corrupted file
list therefore differs between the tested environments.

Formatting or checking each file explicitly reports the correct path. This was
found while verifying the generated Redis command modules, so the project uses
an explicit file list for its formatter check as a temporary workaround.

The Roc reproducer uses basic-cli only for temporary-file and child-process
effects. `scripts/check-known-bug.roc` runs it with a hard timeout and requires
exit 1, the formatter diagnostic markers, the reproducer's observed-failure
marker, and one of those two exact corrupted name lists. It rejects a correct
list of `Alpha.roc` and `Beta.roc`, unrelated failures, and unrecognized output.
