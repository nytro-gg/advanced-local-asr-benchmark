# Verification Report — Advanced Local ASR Benchmark

## Independent package audit

The packaged source was inspected before publication.

- Python source compilation passed.
- CLI `--help` smoke test passed.
- Automated test suite: **28 passed**.
- No API-key-like strings, passwords, private-key headers, bearer tokens, or absolute Windows user paths were found in the packaged text files.
- PowerShell setup and runner scripts were statically reviewed; Windows setup was subsequently executed by the user.
- A clean GitHub clone was populated from the package and the same 28 tests passed in 0.35 seconds.

## Windows laptop verification

The user's Windows 11 x64 environment used CPython 3.12.10.

- `pip check`: no broken requirements found.
- Core native imports passed.
- Automated suite: 28 passed.
- Faster-Whisper Small and Medium both completed CPU INT8 transcription on the same 731.743-second M4A recording with VAD off and automatic language detection.

| Metric | Small | Medium |
|---|---:|---:|
| Transcription time | 145.798 s | 302.089 s |
| Real-time factor | 0.19925 | 0.41283 |
| Peak sampled process RSS | 1,064.01 MiB | 2,644.46 MiB |
| Segments | 63 | 147 |

These are single-run results. Medium took approximately 2.07 times as long and used approximately 2.49 times the peak sampled process RSS. The user judged the Small transcript subjectively very good compared with other tools previously tried. Some lecture-specific phrases were visible in the Medium transcript, but no formal accuracy winner is claimed.

## VAD finding

A separate default whole-file VAD experiment retained approximately 3.84 seconds (0.52%) of the 731.74-second recording. Its runtime is not a comparable full-recording benchmark because most input audio was excluded.

## Verification boundaries and remaining limitations

- The model-run metrics came from one run per model; repeated runs and median comparisons remain future work.
- No human-verified reference transcript was supplied. WER/CER are unavailable, and subjective impressions must not be presented as formal accuracy scores.
- GPU benchmarking, Large-v3 evaluation, and broader multilingual/noise/diarization testing are outside the verified scope.
- Screenshots in `evidence/` document selected earlier experiments. They are not screenshots of the matched November Small/Medium terminal runs.


## Troubleshooting and terminal evidence

- The consolidated issue register is at [docs/issue-register.md](docs/issue-register.md).
- Fourteen terminal-style exhibits are in [evidence/terminal/](evidence/terminal/README.md).
- A text-form command transcript is in [evidence/terminal/command-transcript.md](evidence/terminal/command-transcript.md).

**Authenticity note:** The terminal-style SVGs are reconstructions from user-shared command/output text and the lab notebook. They are labelled “RECONSTRUCTED · NOT LIVE CAPTURE”; they are not represented as original screenshots. The six PNGs in the top-level evidence directory are historical screenshots from earlier experiments, not captures of the November matched Small/Medium runs.
