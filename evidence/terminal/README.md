# Reconstructed Terminal Evidence

This folder contains **14 terminal-style evidence exhibits** reconstructed from PowerShell commands, outputs and experiment records shared during the project.

## Important authenticity note

These files are **not original screen captures**. Each SVG visibly carries a “RECONSTRUCTED · NOT LIVE CAPTURE” label. They are documentary exhibits so the sequence can be read and cited without pretending a screen capture was taken at the time.

- Where exact command and output were pasted in chat, the exhibit reproduces them.
- Where an issue is documented only in the lab notebook, the exhibit labels it as a summary.
- For VAD threshold/window sweeps, the original ad hoc diagnostic command text was not preserved; only measured diagnostic output is shown.
- No terminal output has been invented to fill a gap.

## Exhibit index

| Exhibit | Subject |
|---|---|
| [01 — Python runtime discovery](01-python-runtime-discovery.svg) | Initial runtime-discovery failure and corrected Python selection |
| [02 — Dependency download](02-dependency-download-and-verification.svg) | CTranslate2 transfer interruption, retry, pip check, tests and non-fatal warnings |
| [03 — Small baseline](03-small-automatic-language-run.svg) | Automatic-language Small CPU INT8 baseline |
| [04 — Forced English and VAD](04-forced-language-and-vad.svg) | Forced-English run and whole-file VAD rejection |
| [05 — VAD diagnostics](05-vad-diagnostic-sweep.svg) | Threshold and window-size retention measurements |
| [06 — Medium benchmark](06-medium-benchmark-run.svg) | Matched Medium CPU INT8 run |
| [07 — Package test and staging](07-tests-packaging-and-staging.svg) | Clean-clone tests, staging and whitespace checks |
| [08 — Git identity and push](08-git-identity-and-publication.svg) | Commit identity error, local config, commit and push |
| [09 — Git sync and path correction](09-github-sync-and-path-correction.svg) | Moved clone path and successful new clone |
| [10 — Cleanup lock](10-cleanup-lock-and-restart.svg) | Windows open-handle deletion failure and restart |
| [11 — Repository separation](11-two-repository-separation.svg) | Independent new repository and original project restoration |
| [12 — Source ZIP build](12-source-package-archive-build.svg) | Sanitized source package staging commands |
| [13 — Clone and staging details](13-clean-clone-test-and-git-staging.svg) | Explicit git add command and diff checks |
| [14 — Final cleanup checks](14-cleanup-safety-checks.svg) | Empty-folder checks and successful post-restart deletion |

## Full text transcript

See [command-transcript.md](command-transcript.md) for the longer text-form record of commands, observed outputs and caveats.

## Related incident register

See [issue-register.md](../../docs/issue-register.md) for the consolidated problem inventory and resolutions.
