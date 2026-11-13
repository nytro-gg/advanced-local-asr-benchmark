# Benchmark Methodology

## Matched Faster-Whisper Small vs Medium comparison

The primary comparison uses the same original M4A input for both models.

### Input and controls

| Item | Setting |
|---|---|
| Recording duration | 731.743229 seconds (12 min 11.74 sec) |
| Input | Same M4A file for both models; original left unchanged |
| Device | CPU |
| Requested compute type | INT8 |
| Runtime-reported compute type | `int8_float32` |
| Language | Automatic detection |
| Beam size | 5 |
| VAD | Off |
| Maximum audio duration | 900 seconds |
| Python | 3.12.10, 64-bit |
| OS | Windows 11 x64 |
| System RAM reported | 23,960.07 MiB (~23.96 GiB) |

### Model revisions and results

| Measurement | Small | Medium |
|---|---:|---:|
| Model repository | `Systran/faster-whisper-small` | `Systran/faster-whisper-medium` |
| Resolved commit | `536b0662742c02347bc0e980a01041f333bce120` | `08e178d48790749d25932bbc082711ddcfdfbc4f` |
| Download/cache time | 30.420 s | 66.484 s |
| Model load time | 10.558 s | 3.518 s |
| Transcription time | 145.798 s | 302.089 s |
| Download + load + transcription | 186.776 s | 372.092 s |
| RTF | 0.19925 | 0.41283 |
| Segments | 63 | 147 |
| Detected language | English | English |
| Language probability | 0.29498 | 0.35788 |
| Peak sampled process RSS | 1,064.01 MiB | 2,644.46 MiB |
| Average sampled process CPU | 358.43% | 385.76% |
| Peak sampled process CPU | 425% | 425% |

## Metric definitions

- **Transcription time** is measured through consumption of the lazy segment generator. Model snapshot resolution/download and model construction are reported separately.
- **RTF** = transcription seconds ÷ audio duration seconds. Lower is faster; below 1 means transcription completed faster than playback time.
- **Peak sampled process RSS** is the largest resident-set-size sample recorded by the process monitor. Sampling interval was 250 ms, so brief peaks may be missed. It is not the same as total system RAM consumed by the model alone.
- **Average process CPU** can exceed 100% because usage is measured across multiple logical processors.
- **Segments** are output segment count, not an accuracy score.
- **Language probability** does not measure transcription correctness.

## Interpretation and limitations

Small completed transcription in 145.798 seconds compared with 302.089 seconds for Medium. Medium took about 2.07× as long. Its peak sampled process RSS was about 2.49× Small's. Both RTF values were below 1.

These are single-run observations. Three or more repeated warm-cache runs and medians would be preferable for a formal performance study. The current project does not present the measurements as repeated-run results.

No human-verified reference transcript was supplied. WER/CER are therefore unavailable. The user's positive listening impression of Small and observations of lecture-specific wording in Medium are qualitative only and do not prove that one model is more accurate.

## VAD investigation

A separate default whole-file VAD run retained two segments totaling approximately 3.84 seconds (0.52%) from the 731.74-second input. Its short runtime and RTF are not comparable with the VAD-off full-input runs.

Separate timestamp-only sweeps—not Whisper transcription benchmarks—showed:

| Windowing | Threshold | Regions | Retained | Retention |
|---|---:|---:|---:|---:|
| Whole file | 0.50 | 1 | 3.84 s | 0.52% |
| Whole file | 0.20 | 4 | 9.84 s | 1.34% |
| 30-second chunks | 0.50 | 37 | 145.09 s | 19.83% |
| 60-second chunks | 0.50 | 20 | 92.06 s | 12.58% |
| 120-second chunks | 0.50 | 12 | 65.07 s | 8.89% |
| 300-second chunks | 0.50 | 4 | 14.29 s | 1.95% |
| 30-second chunks | 0.20 | 57 | 319.50 s | 43.66% |

Retention figures show how much audio VAD timestamps retained; they do not prove that retained regions are speech or that rejected regions are silence. The default VAD behavior is not recommended for this recording.

## Historical experiments kept separate

- An earlier Faster-Whisper run processed 3,466.26 seconds of audio in 26.64 seconds. It is a separate historical run and is not comparable with the November 12-minute benchmark.
- A separate Small run explicitly forced English and reported probability 1.0. Because English was supplied, that is not independent language-detection confidence.
- Earlier Parakeet work included an attempted CPU buffer allocation of 22,837,892,192 bytes (~22.8 GB) and a different GGUF workflow reported as successful with chunking constraints. The exact successful artifact/revision/runtime was not preserved; Parakeet is not part of the reproducible package.

## Reproducibility checklist

1. Use the exact same input file and record its SHA-256.
2. Pin the model repository and 40-character resolved revision.
3. Keep language, beam size, VAD, compute type and audio-duration cap identical.
4. Record download/cache, model load and transcription times separately.
5. Run at least three repeats where practical and report median plus spread.
6. Use a manually verified reference transcript for WER/CER.
7. Keep private or permission-restricted audio and transcripts out of the public repository.
