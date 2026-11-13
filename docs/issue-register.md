# Complete Issue Register — Advanced Local ASR Benchmark

This register consolidates the problems, warnings, failed attempts, corrections and limitations identifiable from the shared PowerShell transcripts, model-run records and GitHub publication work on 13 November 2026.

**Evidence policy:** An issue is called an error only where the chat or experiment log records an error. Non-fatal warnings, research limitations and process corrections are separated. Terminal exhibits in ../evidence/terminal are reconstructions from those records, not live captures.

## A. Python and dependency setup

| ID | Problem | Evidence / impact | Resolution / final status |
|---|---|---|---|
| SETUP-01 | Python runtime discovery failed | Initial setup surfaced `No suitable Python runtime found` because it attempted to probe a runtime that was not installed. | Replaced discovery with `py -0p` enumeration, selected installed 64-bit Python 3.12, and verified architecture. Resolved. |
| SETUP-02 | Replacement setup script was not at expected copy path | An initial Copy-Item attempt failed because the replacement script had not yet been placed at the expected Downloads path. | The replacement script was subsequently downloaded/copied into `scripts/setup.ps1`. Resolved. |
| SETUP-03 | CTranslate2 wheel transfer repeatedly interrupted | pip repeatedly disconnected near 19 MB of an approximately 20.4 MB wheel; six attempts failed. | Retried with `python -m pip install --resume-retries 15 --timeout 120 -r requirements.txt`. Installation completed. Resolved. |
| SETUP-04 | `hf_xet` was not installed | Hugging Face warned that Xet storage support was unavailable. | Regular HTTP download fallback worked. Non-fatal warning, not a failed model download. |
| SETUP-05 | Windows symlink support was unavailable | Hugging Face warned that cache symlinks could not be created. | Cache operated using a less space-efficient fallback. Non-fatal warning. |
| SETUP-06 | PowerShell command-entry mistakes | A Python command and PowerShell listing command were pasted together once, producing a Select-Object property error; a directory path was also briefly entered as though it were a command. | Commands were rerun separately. No project data change was recorded. Resolved. |
| SETUP-07 | First-run audio-duration guard | Package defaults to a 120-second maximum. The 12-minute test required an explicit override. | User used `--max-audio-seconds 900` for the 731.74-second benchmark. Safety guard, not a defect. |

## B. Model and audio experiments

| ID | Problem / observation | Evidence / impact | Interpretation / status |
|---|---|---|---|
| MODEL-01 | Parakeet memory-allocation failure | An earlier Parakeet attempt reached a CPU buffer allocation request of 22,837,892,192 bytes (about 22.8 GB). | Failed on the tested system. Resource limitation observed in that configuration, not proof every Parakeet model fails. |
| MODEL-02 | Earlier Parakeet workflow was not fully reproducible | A separate GGUF workflow was reported to work with chunking constraints, but the exact successful artifact, revision, conversion and runtime were not preserved. | Historical exploratory observation; not part of the current runnable evaluator. |
| MODEL-03 | Forced-English Small run retained repeated phrases | Forced-English transcript contained repeated phrases near the end; forcing English did not visibly resolve the behavior. | Qualitative observation only; no verified reference transcript exists. |
| MODEL-04 | Whole-file VAD retained almost none of the recording | Default VAD-on run emitted 2 segments and retained about 3.84 s (0.52%) of 731.743 s input. | Not a valid full-recording benchmark. The apparent 3.779 s runtime / RTF 0.00516 is misleading against VAD-off full-input runs. Default VAD was rejected for this recording. |
| MODEL-05 | VAD retention changed with threshold and window size | Whole-file, 30 s, 60 s, 120 s and 300 s diagnostics retained markedly different durations. | Windowing sensitivity observed; root cause not established. Retained duration is not proof of speech. |
| MODEL-06 | No verified reference transcript | No human-checked verbatim reference was supplied for the 12-minute recording. | WER/CER unavailable; no formal accuracy winner can be declared. Listening impressions remain qualitative. |
| MODEL-07 | Medium transcript contained fragmented material | Inspected output included recognizable lecture terms but also isolated dots and fragmented opening text. | Qualitative note only; not a formal accuracy score. |
| MODEL-08 | Resource measurements are sampled | Process RSS/CPU sampled every 250 ms. | Brief peaks may be missed. Process RSS is not total system RAM attributable only to the model. |
| MODEL-09 | Only one matched run per model | One Small and one Medium automatic-language/VAD-off run recorded. | 2.07× time and 2.49× peak-RSS differences are single-run observations, not medians or statistical claims. |
| MODEL-10 | Old historical Faster-Whisper measurement is not matched | A separate earlier record says 3,466.26 s of audio processed in 26.64 s. | Different experiment; must not be mixed with the November 12-minute comparison. |

## C. Package audit and source publication

| ID | Problem / observation | Evidence / impact | Resolution / final status |
|---|---|---|---|
| PUB-01 | Initial checksum manifest did not match the packaged archive | Audit found the original SHA256SUMS.txt omitted archive entries and the packaged setup.ps1 hash differed. | Corrected manifest included in the published package. It hashes listed source files, not the ZIP archive. |
| PUB-02 | Git commit author identity was not configured | git commit stopped with Author identity unknown and requested user.name / user.email. | Configured repository-local identity as Nivesh Patro and GitHub noreply address 93911786+nytro-gg@users.noreply.github.com. Commit then succeeded. No global identity was changed. |
| PUB-03 | Browser authentication was required for push | Git printed the browser-authentication prompt. | Authentication completed and push succeeded: 8d7bc6f..44469d3 main -> main. |
| PUB-04 | Windows line-ending warning while staging | Git warned LF would be replaced by CRLF in working-copy files. | Staging and git diff --cached --check succeeded. Non-fatal line-ending warning. |
| PUB-05 | Working tree was one commit ahead before push | git status showed main...origin/main [ahead 1]. | Commit pushed; subsequent status showed main...origin/main with no pending changes. |
| PUB-06 | Source staging required an explicit file list | Clone status included untracked audio/ and models/ directory entries alongside source files. | Explicit git add list staged code, scripts, docs, requirements and README placeholders only; private audio/model weights were not in the staged list. .gitignore excludes local audio, models, caches and generated runs. |
| PUB-07 | Repository purpose initially misunderstood | A portfolio refresh was first applied to the existing repository after the user intended a wholly separate LinkedIn project. | Old repository main tree restored to its pre-November baseline; new public advanced-local-asr-benchmark repository created separately. Original URL and history retained. Corrected. |
| PUB-08 | New local clone was initially addressed at its old Downloads path | git pull failed with No such file or directory after the clone had been moved to Desktop. | Updated paths to Desktop\Local Speech Model\V2; old clone fast-forwarded and new repo cloned successfully. Resolved. |

## D. Windows cleanup

| ID | Problem / observation | Evidence / impact | Resolution / final status |
|---|---|---|---|
| CLEAN-01 | Original project directory was in use | Remove-Item failed with a Windows message that the directory was being used by another process. | Closed/released project context and restarted Windows. |
| CLEAN-02 | Failed deletion was followed by a misleading success message | PowerShell continued after a non-terminating Remove-Item error and printed Research cleanup completed although Test-Path showed the folder still existed. | Message was not treated as proof. Test-Path and Get-ChildItem were used to inspect what remained. |
| CLEAN-03 | Empty nested directory remained locked | Nested folder contained zero items, but Windows still refused to remove the directory because a process held an open handle. | After Windows restart, both directories were deleted using -ErrorAction Stop. Final output: Old project exists: False. Resolved. |
| CLEAN-04 | Separate user-level cache check | Get-ChildItem $env:USERPROFILE\.cache returned no entries at the time of the check. | No entries were listed by that command. This does not prove every possible application cache elsewhere was empty. |

## E. Evidence and reproducibility boundaries

1. The six original screenshots in evidence/ are historical screenshots of earlier Faster-Whisper and Parakeet experiments. They are not terminal screenshots of the matched November Small/Medium runs.
2. The new images in evidence/terminal/ are reconstructed terminal transcript exhibits assembled from commands and outputs visible in the conversation and experiment records. They are labelled as reconstructions, not original screen captures.
3. Where the original ad hoc VAD diagnostic command text was not retained, the exhibit reports measured diagnostic output and explicitly says command text was not preserved.
4. No private audio recordings, raw full transcripts, per-run metadata JSON, segments CSV files, downloaded model weights or virtual environments are included.
5. The project does not claim formal transcription accuracy without a human-verified reference transcript.

## Final outcome

- Package and benchmark documentation are published in nytro-gg/advanced-local-asr-benchmark.
- Matched Small and Medium CPU INT8 runs completed successfully.
- 28 automated tests passed.
- Original research directory and its project-local model cache/audio/results/virtual environment were deleted after a Windows restart released the directory lock.
- Git, Python installations, VS Code and unrelated development tools were retained.
- Original nytro-gg/local-speech-model-evaluation repository was restored to its pre-November file tree; new research project has its own repository.
