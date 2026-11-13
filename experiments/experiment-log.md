# Advanced Local ASR Benchmark — Experiment Log

Chronological lab notebook for the Advanced Local ASR Benchmark project. Entries record configuration, results, errors, warnings, interpretation, and follow-up work.

## Lab rules

- Keep original test audio unchanged. Do not denoise, normalize, trim, or otherwise modify a source recording unless a separate experiment explicitly requires a transformed copy.
- Never commit private recordings, their transcripts, temporary audio excerpts, raw model outputs, local absolute paths, model-weight caches, virtual environments, or machine-specific files.
- Record model repository and resolved revision where available.
- Distinguish measured metrics from subjective listening impressions.
- Do not report WER/CER unless a human-verified reference transcript was supplied.
- Do not compare speed figures when runs did not process equivalent amounts of audio.
- Append new entries; do not silently rewrite earlier observations.

## Environment and setup — 2026-11-13

### Verified environment

| Item | Observed value |
|---|---|
| Operating system | Windows 11 |
| Shell | Windows PowerShell 5.1 |
| Python | 3.12.10, 64-bit |
| Project package version | 1.2.0 |
| Faster-Whisper | 1.2.1 |
| CTranslate2 | 4.8.5 |
| PyAV | 17.1.0 |
| Hugging Face Hub | 0.36.0 |
| RapidFuzz | 3.14.6 |
| psutil | 7.2.2 |
| pytest | 8.4.2 |
| Reported system RAM | approximately 23.96 GiB |

The evaluation runs in a project-local Python virtual environment. The tested inference configuration in this session used CPU INT8. GPU/CUDA inference was not enabled by this package version.

### Setup errors and resolutions

1. **Python runtime discovery failure.** The original setup script attempted to probe a Python runtime that was not installed, and PowerShell surfaced the error: “No suitable Python runtime found”. The setup script was replaced with a Python-discovery version that selected the installed Python 3.12 64-bit runtime.
2. **Setup script copy path failure.** The first Copy-Item attempt failed because the replacement script was not yet present at the expected Downloads path. The file was subsequently downloaded and copied into scripts/setup.ps1.
3. **Interrupted CTranslate2 wheel download.** pip repeatedly lost its connection while downloading the 20.4 MB CTranslate2 wheel, reaching approximately 19.0 MB before failing after six attempts. Retrying with --resume-retries 15 and --timeout 120 completed the installation.
4. **Non-fatal Windows/Hugging Face warnings.** Hugging Face reported that hf_xet was not installed and used regular HTTP downloads instead. It also warned that Windows symlinks were unavailable; caching continued in its less space-efficient fallback mode.
5. **PowerShell command-entry mistakes.** A Python command and a PowerShell listing command were once pasted together, causing a Select-Object property error. A project directory path was also briefly entered as if it were a command. Both were corrected by running the intended commands separately; neither changed project files.

### Environment verification

- python -m pip check: **No broken requirements found.**
- python -m pytest -q: **28 passed in 2.76 seconds.**
- Core imports (faster_whisper, ctranslate2, av, onnxruntime, rapidfuzz, psutil): successful.

These checks validate package compatibility, imports, and the included automated tests. They do not, by themselves, validate transcription accuracy.

---

## Test recording — 2026-11-13

A single approximately 12-minute, 11.74-second M4A recording was used for the current comparison. The original recording was kept unchanged.

The user listened to two 30-second excerpts taken from the recording and reported:
- There are periods of silence.
- The speaker's voice is very faint.
- The voice is distorted.
- There is substantial background noise.

The user considers the first Faster-Whisper Small transcript subjectively very good relative to other transcription tools they have tried. This is a listening impression, not a WER/CER measurement. No human-verified reference transcript has yet been supplied, so formal accuracy metrics are unavailable.

## Experiment 1 — Faster-Whisper Small, automatic language, VAD off

**Run timestamp:** 2026-11-13 15:22:18 UTC  
**Model:** Systran/faster-whisper-small  
**Resolved model commit:** 536b0662742c02347bc0e980a01041f333bce120  
**Configuration:** CPU, requested INT8, beam size 5, automatic language detection, VAD off.

| Metric | Result |
|---|---:|
| Audio duration | 731.743 s |
| Model download/cache time | 30.420 s |
| Model load time | 10.558 s |
| Transcription time | 145.798 s |
| Download + load + transcription | 186.776 s |
| Real-time factor (transcription only) | 0.19925 |
| Segments emitted | 63 |
| Detected language | English |
| Language probability | 0.29498 |
| Peak sampled process RSS | 1,064.01 MiB |
| Average sampled process CPU | 358.43% |
| Peak sampled process CPU | 425% |

The runtime reported compute type int8_float32. CPU percentages can exceed 100% because process CPU usage is measured across logical CPU cores.

**Observation:** The model completed a full-file transcription in approximately 2 minutes 26 seconds, around 5.02 times faster than the recording duration, excluding model download and loading.

**Accuracy status:** No reference transcript supplied; WER/CER not measured.

## Experiment 2 — Faster-Whisper Small, forced English, VAD off

**Run timestamp:** 2026-11-13 15:43:40 UTC  
**Model and revision:** Same Small model and resolved commit as Experiment 1.  
**Configuration:** CPU INT8, beam size 5, language explicitly set to en, VAD off.

| Metric | Result |
|---|---:|
| Audio duration | 731.743 s |
| Model cache time | 0.728 s |
| Model load time | 1.059 s |
| Transcription time | 138.580 s |
| Download/cache + load + transcription | 140.368 s |
| Real-time factor (transcription only) | 0.18938 |
| Segments emitted | 85 |
| Language requested/detected | English |
| Language probability reported | 1.0 |
| Peak sampled process RSS | 1,064.84 MiB |
| Average sampled process CPU | 388.81% |
| Peak sampled process CPU | 425% |

The reported language probability of 1.0 is expected when English is explicitly supplied; it is not an independent language-detection result.

**Observation:** The transcript remained broadly similar to Experiment 1. Repeated phrases appeared near the end of the output. Explicitly forcing English did not visibly resolve the recognition issues. Runtime was approximately seven seconds faster, but this alone does not establish an accuracy improvement.

**Accuracy status:** No reference transcript supplied; WER/CER not measured.

## Experiment 3 — Faster-Whisper Small, forced English, VAD on

**Run timestamp:** 2026-11-13 15:51:52 UTC  
**Model and revision:** Same Small model and resolved commit.  
**Configuration:** CPU INT8, language en, VAD enabled using Faster-Whisper defaults.

The installed package reported these default VAD options:

- threshold: 0.5
- neg_threshold: None
- min_speech_duration_ms: 0
- max_speech_duration_s: infinity
- min_silence_duration_ms: 2000
- speech_pad_ms: 400

| Metric | Result |
|---|---:|
| Full input duration | 731.743 s |
| Transcription time | 3.779 s |
| Reported real-time factor | 0.00516 |
| Segments emitted | 2 |
| Peak sampled process RSS | 651.22 MiB |

The two output segments covered approximately 3.09–7.09 seconds and contained only two short phrases. An independent VAD timestamp inspection at threshold 0.5 detected one region from 3.09 to 6.93 seconds (3.84 seconds retained, approximately 0.52% of the recording).

**Interpretation:** This run is not a usable full-recording transcription benchmark. The reported real-time factor divides processing time by the full input duration even though VAD retained only a tiny fraction of the input. It must not be compared to the VAD-off real-time factors as if both runs transcribed the same amount of audio.

**Evaluator issue identified:** The current evaluator passes vad_filter=args.vad but does not expose custom VAD options or record the amount of audio retained by VAD. It therefore lacks safeguards against extreme input rejection and misleading speed reporting.

## VAD diagnostic — threshold sweep on the entire recording

The audio was decoded to 16 kHz and Silero VAD timestamps were inspected directly. No Whisper transcription was run for this diagnostic.

| Threshold | Regions detected | Audio retained | Percentage of recording |
|---:|---:|---:|---:|
| 0.50 | 1 | 3.84 s | 0.52% |
| 0.40 | 3 | 5.74 s | 0.78% |
| 0.30 | 4 | 8.24 s | 1.13% |
| 0.20 | 4 | 9.84 s | 1.34% |

Lowering the threshold increased detected audio only slightly when the entire recording was processed in one call.

## VAD diagnostic — independent 30-second windows

The same recording was analysed in independent 30-second windows. This was a VAD timestamp diagnostic only, not a Whisper transcription benchmark.

| Threshold | Regions across windows | Total retained | Percentage of recording |
|---:|---:|---:|---:|
| 0.50 | 37 | 145.09 s | 19.83% |
| 0.20 | 57 | 319.50 s | 43.66% |

Independent-window processing retained much more audio than whole-file VAD. This is evidence that the way the audio is presented to the VAD implementation materially affects its detections. It does not prove that all retained sections are speech or that all rejected sections are silence.

## VAD diagnostic — four sampled windows

Measured with decoded audio at 16 kHz:

| Window | RMS | Peak | Threshold 0.50 detections | Threshold 0.20 detections |
|---|---:|---:|---|---|
| 0–30 s | 0.035836 | 0.292725 | 3.09–6.93 s | 0.00–0.72, 3.02–7.34, 10.03–11.92, 15.31–18.22 s |
| 180–210 s | 0.046660 | 0.325500 | 180.00–180.59 s | 180.00–181.71 s |
| 350–380 s | 0.048508 | 0.405212 | 352.19–353.15, 360.90–362.27, 368.83–373.57 s | 350.69–377.02 s |
| 690–731 s | 0.046419 | 0.580139 | 690.14–691.04, 700.99–702.11 s | 690.00–695.84, 697.89–703.04, 706.75–707.97, 715.68–716.80 s |

These measurements show that the audio has non-zero signal energy, but RMS and peak measurements alone cannot establish speech presence or intelligibility.

## VAD diagnostic — chunk-size sweep at threshold 0.50

The recording was analysed in independent windows of different lengths. No transcription was run.

| Window size | Regions detected | Audio retained | Percentage of recording |
|---:|---:|---:|---:|
| 30 s | 37 | 145.09 s | 19.83% |
| 60 s | 20 | 92.06 s | 12.58% |
| 120 s | 12 | 65.07 s | 8.89% |
| 300 s | 4 | 14.29 s | 1.95% |

**Observation:** Shorter independent windows produced more retained audio in this test recording. The mechanism behind the whole-file versus windowed discrepancy is not yet established. Recurrent VAD state handling is a hypothesis to investigate, not a proven root cause.

## Current conclusions and open issues

1. Faster-Whisper Small completed the 12-minute recording on CPU INT8 in approximately 2 minutes 26 seconds with VAD off.
2. The user considers the first Small transcript subjectively very good compared with other tools they have tried. Formal accuracy remains unmeasured because there is no verified reference transcript.
3. Forcing English did not materially change the observed transcript behaviour.
4. The default whole-file VAD run retained only about 3.84 seconds of the 731.74-second recording. Do not use VAD for production runs on this test until the discrepancy is understood.
5. The 30-second VAD-window diagnostic retained substantially more audio than larger windows, but retention is not the same as correct speech detection.
6. The recording contains periods of silence, faint/distorted speech, and substantial background noise according to the user's listening assessment. The original audio is to remain unmodified.
7. The evaluator should record VAD-retained duration/percentage and warn or fail when an unusually large proportion of the input is discarded. Report a VAD-aware effective processing denominator or suppress misleading RTF values.
8. Repetition-loop controls (for example, making condition_on_previous_text configurable) should be evaluated separately, with careful comparisons and reference transcripts.
9. A human-verified transcript is needed before reporting WER/CER or making formal accuracy claims.

## Cleanup policy — after research is finished

Keep system-wide tools needed for coursework and other development:
- Git
- Python installations
- VS Code and other general-purpose development tools

Remove only this research project's temporary/local assets after the final benchmark and GitHub publication are verified:
- Project virtual environment (.venv/)
- Downloaded speech-model weights and Hugging Face cache (models/hf-cache/)
- Local test recordings, reference transcripts, derived excerpts, and temporary audio
- Local generated benchmark outputs and caches, after a sanitized summary has been committed
- Any project-only packages or tools that were installed outside the virtual environment, but only after checking they are not used by other coursework

Do not delete Git, Python, VS Code, or unrelated development environments. Before removing anything, verify the GitHub commit and make sure no required source code or documentation exists only in the local project folder.

The local project directory itself can be deleted once its public repository contents are verified and the final cleanup checklist has been completed.


## Experiment 4 — preliminary completion note superseded

The initial note that Medium metrics were pending was written immediately after the model run completed. It was superseded after metadata.json and transcript.txt were inspected. The verified measurements and qualitative caveats are recorded in the following Experiment 4 entry and in results/results.md.

## Experiment 4 — Faster-Whisper Medium, automatic language, VAD off

**Run timestamp:** 2026-11-13 16:57:25 UTC  
**Result folder name:** test12_medium_20261113_165725_692132  
**Model:** Systran/faster-whisper-medium  
**Resolved model commit:** 08e178d48790749d25932bbc082711ddcfdfbc4f  
**Configuration:** CPU, requested INT8, beam size 5, automatic language detection, VAD off, 900-second maximum duration.  
**Input:** Same original 731.743-second M4A recording used for the Small experiments; original audio unchanged.

| Metric | Result |
|---|---:|
| Model download/cache time | 66.484 s |
| Model load time | 3.518 s |
| Transcription time | 302.089 s |
| Download + load + transcription | 372.092 s |
| Real-time factor (transcription only) | 0.41283 |
| Segments emitted | 147 |
| Detected language | English |
| Language probability | 0.35788 |
| Requested compute type | INT8 |
| Runtime-reported compute type | int8_float32 |
| Peak sampled process RSS | 2,644.46 MiB |
| Average sampled process CPU | 385.76% |
| Peak sampled process CPU | 425% |
| System RAM total | 23,960.07 MiB |

**Console warnings:** Hugging Face reported that hf_xet was not installed and used regular HTTP download fallback. It also warned that Windows symlinks were unavailable; model caching continued with the less space-efficient fallback. The model download (approximately 1.53 GB) completed, and the evaluation run completed successfully.

**Comparison with Small automatic-language baseline:** Both runs used the same 731.743-second source audio, CPU INT8, beam size 5, automatic language detection, and VAD off.

| Metric | Small | Medium |
|---|---:|---:|
| Transcription time | 145.798 s | 302.089 s |
| Real-time factor | 0.19925 | 0.41283 |
| Peak sampled process RSS | 1,064.01 MiB | 2,644.46 MiB |
| Average sampled process CPU | 358.43% | 385.76% |
| Segment count | 63 | 147 |
| Language probability | 0.29498 | 0.35788 |

Medium took approximately 2.07 times as long and used approximately 2.49 times the peak sampled process RSS. Both remained faster than real time: approximately 2.42x for Medium versus 5.02x for Small. Download and model-load time are reported separately and should not be mixed with the transcription-only RTF.

**Qualitative transcript inspection:** The inspected Medium transcript includes recognizable lecture concepts and phrases such as traffic flow, checkpoints, variables, eigenvalues and eigenvectors. The beginning also contains multiple isolated dots and fragmented phrases. This is a preliminary qualitative observation only; there is no verified reference transcript and no WER/CER score. The user previously judged the Small automatic-language transcript subjectively very good. Do not declare Medium the accuracy winner without a careful aligned listening comparison or human-verified reference.

**Current conclusion:** Small is faster and more memory-efficient. Medium may offer useful differences in wording/detail, but its accuracy advantage is unproven. This is a compute-versus-output comparison, not a formal accuracy ranking.

**Next action:** Ask the user to compare specific sections they know from the lecture, or prepare a short manually verified reference segment. Keep VAD off and keep the original recording unchanged for further model comparisons.


---

## GitHub source publication and repository verification — 2026-11-13

### Sanitized source package

A temporary source staging directory and a 22,401-byte ZIP were created. The staged package contained the evaluator, tests, PowerShell setup/runner scripts, pinned requirements, and documentation placeholders. The audio/model directories were not part of the explicit source-file staging list; private audio and downloaded model weights were not published.

The repository clone was tested before commit:
- Clean-clone test suite: **28 passed in 0.35 seconds**.
- Explicit Git staging contained 16 files: 1,200 insertions and 11 deletions.
- `git diff --cached --check` passed.

Git emitted LF-to-CRLF line-ending warnings while staging in Windows. The whitespace check succeeded; these were not commit failures.

### Git identity and first push

The first source commit failed because Git author identity was not configured in the fresh clone. The identity was set **locally to that repository**:
- user.name: Nivesh Patro
- user.email: 93911786+nytro-gg@users.noreply.github.com

The source commit then succeeded as `44469d3`. Git requested browser authentication during push; browser authentication was completed and the push succeeded:
- `8d7bc6f..44469d3  main -> main`
- Final local status after push: `## main...origin/main`

No global Git author identity was changed.

### Package checksum correction

The independent package audit detected that the initial SHA256SUMS manifest did not exactly match the packaged archive: some listed entries were omitted from the archive and the packaged setup script hash differed. A corrected manifest was included with the source publication. The manifest hashes the listed source files and does not claim to hash the ZIP archive.

## Windows cleanup and directory-lock issue — 2026-11-13

The first recursive Remove-Item attempt failed because Windows reported that the original project directory was being used by another process. The PowerShell block nevertheless printed a completion message because the non-terminating deletion error was not checked. A subsequent Test-Path showed the outer project directory still existed.

The nested project directory was then confirmed empty (zero items), but Windows still refused to remove the directory because a handle remained open. The user restarted Windows and reran the guarded deletion script with `-ErrorAction Stop`. The final output was:
- `Old project exists: False`

The separate GitHub clone was checked independently and was not part of the deletion target. A user-level `.cache` listing had returned no entries at the time of the earlier check; this does not establish that every application cache elsewhere on the PC was empty.

## Separate repository and Desktop clone — 2026-11-13

The user's LinkedIn already referenced the earlier `local-speech-model-evaluation` repository. The user clarified that the new ASR research should be a wholly separate project.

An interim portfolio refresh had been applied to the old repository during the misunderstanding. Its main tree was subsequently restored to the pre-November baseline tree from commit `3ae319563681cc2a56ad560f02015dc9f4207398`, with a new restoration commit `e5335f5917959098280c587737b719525cd5e3ff`. The original URL and historical commits were retained.

The new public repository `nytro-gg/advanced-local-asr-benchmark` was created separately and populated with the evaluator, setup/runner scripts, tests, benchmark reports, issue documentation, and six historical screenshots. Its first full project commit is `f47f8f9f5c407625d971c97f58c81c39beff486b`.

The local clones were organized under `Desktop\Local Speech Model\V2`. An initial pull attempt still used the old Downloads path and failed with `No such file or directory`. Updating the PowerShell variables to the Desktop location corrected it:
- Original repository clone fast-forwarded to the restored tree.
- New repository clone downloaded 47 Git objects (about 1.74 MiB).
- New clone status: `## main...origin/main`.

## Terminal evidence reconstruction — 2026-11-13

Fourteen SVG terminal-style exhibits were added under `evidence/terminal/`, alongside a text command transcript and a consolidated issue register. These are explicitly labelled **RECONSTRUCTED — NOT LIVE CAPTURE**. Exact user-shared command/output pairs are reproduced where available; notebook-only outcomes are labelled as summaries. The original VAD diagnostic command text was not preserved, so only its recorded outputs are shown. No fake live terminal captures were created.
