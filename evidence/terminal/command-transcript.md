# Reconstructed Command Transcript

**Important:** This documentary transcript is assembled from commands and outputs shared in the conversation. It is not a raw PowerShell history export, and the images beside it are not live screen captures. Where the original command text was not preserved, that is stated rather than guessed.

## 1. Environment setup and dependency recovery

### Setup attempt

```powershell
.\\scripts\\setup.ps1
```

Observed error:

```text
No suitable Python runtime found.
```

Correction recorded in the final setup script:
- Enumerate installed runtimes with py -0p.
- Prefer 64-bit Python 3.12, then 3.11.
- Probe architecture before creating the project virtual environment.
- Verify native imports before downloading model weights.

### CTranslate2 retry

```powershell
python -m pip install --resume-retries 15 --timeout 120 -r requirements.txt
python -m pip check
python -m pytest -q
```

Recorded outputs:
- Dependency installation completed.
- No broken requirements found.
- 28 passed in 2.76s.

The preceding pip failures near 19 MB of the approximately 20.4 MB CTranslate2 wheel are summarized in the issue register; the exact network exception text was not preserved in the visible transcript.

## 2. Faster-Whisper Small — automatic language, VAD off

```powershell
python -m src.evaluate --audio ".\\audio\\test12.m4a" --model small --device cpu --compute-type int8 --max-audio-seconds 900
```

Recorded result:
- Input: 731.743 s.
- Transcription: 145.798 s.
- RTF: 0.19925.
- Segments: 63.
- Detected language: English; probability 0.29498.
- Peak sampled process RSS: 1,064.01 MiB.
- Average sampled process CPU: 358.43%; peak 425%.
- Model commit: 536b0662742c02347bc0e980a01041f333bce120.
- Model cache/download: 30.420 s; load: 10.558 s.
- No human-verified reference; WER/CER not measured.

## 3. Faster-Whisper Small — forced English, VAD off

```powershell
python -m src.evaluate --audio ".\\audio\\test12.m4a" --model small --device cpu --compute-type int8 --language en --max-audio-seconds 900
```

Recorded result:
- Transcription: 138.580 s; RTF 0.18938; 85 segments.
- Peak sampled process RSS: 1,064.84 MiB.
- Average sampled process CPU: 388.81%; peak 425%.
- Language probability 1.0 is not independent detection confidence because English was forced.
- Repeated phrases remained near the transcript end.

## 4. Faster-Whisper Small — forced English, VAD on

```powershell
python -m src.evaluate --audio ".\\audio\\test12.m4a" --model small --device cpu --compute-type int8 --language en --vad --max-audio-seconds 900
```

Recorded result:
- Transcription: 3.779 s.
- Two segments emitted.
- Only about 3.84 s (0.52%) of the full 731.743 s input was retained.
- Reported RTF 0.00516 is not comparable with VAD-off runs because almost all input was discarded.

The later VAD threshold/window diagnostic command text was not preserved in the user-visible transcript. Its measured outputs are reproduced in the VAD exhibit and methodology document.

## 5. Faster-Whisper Medium — automatic language, VAD off

```powershell
python -m src.evaluate --audio ".\\audio\\test12.m4a" --model medium --device cpu --compute-type int8 --max-audio-seconds 900
```

Recorded result:
- Transcription: 302.089 s; RTF 0.41283.
- Segments: 147.
- Detected language: English; probability 0.35788.
- Peak sampled process RSS: 2,644.46 MiB.
- Average sampled process CPU: 385.76%; peak 425%.
- Model commit: 08e178d48790749d25932bbc082711ddcfdfbc4f.
- Model cache/download: 66.484 s; load: 3.518 s.
- Same 731.743 s input, CPU INT8, beam 5, automatic language, VAD off.
- No human-verified reference; WER/CER unavailable.

## 6. Building the sanitized source ZIP

The following commands were pasted into PowerShell to create a temporary staging directory and source ZIP.

```powershell
$Stage = Join-Path $env:TEMP "speech-eval-publication"
$Zip = Join-Path $env:USERPROFILE "Downloads\\speech-eval-source.zip"

Remove-Item $Stage -Recurse -Force -ErrorAction SilentlyContinue

New-Item -ItemType Directory -Path "$Stage\\src" -Force | Out-Null
New-Item -ItemType Directory -Path "$Stage\\scripts" -Force | Out-Null
New-Item -ItemType Directory -Path "$Stage\\tests" -Force | Out-Null

Copy-Item .\\src\\evaluate.py "$Stage\\src\\"
Copy-Item .\\scripts\\setup.ps1 "$Stage\\scripts\\"
Copy-Item .\\scripts\\run-small.ps1 "$Stage\\scripts\\"
Copy-Item .\\tests\\test_evaluate.py "$Stage\\tests\\"

Copy-Item .\\requirements.txt $Stage
Copy-Item .\\CHANGELOG.md $Stage
Copy-Item .\\MODEL_NOTES.md $Stage
Copy-Item .\\TEST_PROTOCOL.md $Stage
Copy-Item .\\VERIFICATION_REPORT.md $Stage
Copy-Item .\\SHA256SUMS.txt $Stage
Copy-Item .\\README.md "$Stage\\PACKAGE-README.md"

Compress-Archive -Path "$Stage\\*" -DestinationPath $Zip -Force
Get-Item $Zip | Select-Object FullName, Length
```

Recorded archive size: **22,401 bytes**.

## 7. Clone, extract and test the source package

The shared commands included:

```powershell
$Source = (Get-Location).Path
$RepoPath = Join-Path $env:USERPROFILE "Downloads\\local-speech-model-evaluation-github"
$Zip = Join-Path $env:USERPROFILE "Downloads\\speech-eval-source-audited.zip"

if (Test-Path $RepoPath) {
    throw "Destination already exists. Stopping to avoid overwriting anything."
}

git clone https://github.com/nytro-gg/local-speech-model-evaluation.git $RepoPath
if ($LASTEXITCODE -ne 0) {
    throw "Git clone failed."
}

Expand-Archive -LiteralPath $Zip -DestinationPath $RepoPath -Force
$Python = Join-Path $Source ".venv\\Scripts\\python.exe"

Push-Location $RepoPath
try {
    & $Python -m pytest -q
    if ($LASTEXITCODE -ne 0) { throw "Tests failed." }
    git status --short
}
finally {
    Pop-Location
}

git -C $RepoPath status --short
```

Recorded clean-clone test result: **28 passed in 0.35s**.

## 8. Explicit source staging and whitespace review

```powershell
git -C $RepoPath add -- .gitignore README.md CHANGELOG.md MODEL_NOTES.md SHA256SUMS.txt TEST_PROTOCOL.md VERIFICATION_REPORT.md requirements.txt docs/package-quickstart.md audio/README.md models/README.md results/README.md scripts/setup.ps1 scripts/run-small.ps1 src/evaluate.py tests/test_evaluate.py

git -C $RepoPath diff --cached --check
git -C $RepoPath status --short
git -C $RepoPath diff --cached --stat
```

Recorded result: **16 files changed, 1,200 insertions, 11 deletions**. Git emitted LF-to-CRLF working-copy warnings; the staged whitespace check passed.

## 9. Git author configuration, commit and push

The first commit attempt failed:

```powershell
git -C $RepoPath commit -m "feat: publish verified speech evaluation source"
```

Observed:
```text
Author identity unknown.
Please tell me who you are.
fatal: unable to auto-detect email address.
```

Repository-local correction:

```powershell
git -C $RepoPath config user.name "Nivesh Patro"
git -C $RepoPath config user.email "93911786+nytro-gg@users.noreply.github.com"
git -C $RepoPath config user.name
git -C $RepoPath config user.email
```

Then:

```powershell
git -C $RepoPath commit -m "feat: publish verified speech evaluation source"
git -C $RepoPath status --short --branch
git -C $RepoPath push origin main
git -C $RepoPath status --short --branch
```

Recorded:
- Commit: 44469d3.
- Browser authentication prompt appeared and was completed.
- Push succeeded: 8d7bc6f..44469d3 main -> main.
- Final status: ## main...origin/main.

## 10. Cleanup attempt and Windows folder lock

The first removal command:

```powershell
Remove-Item -LiteralPath $OldRoot -Recurse -Force
```

failed with a Windows message that the folder was in use by another process. The PowerShell block nevertheless printed a completion message because the non-terminating error was not checked; subsequent Test-Path proved the folder still existed.

The nested project directory was then confirmed empty:

```powershell
$Remaining = @(Get-ChildItem -LiteralPath $Inner -Force -ErrorAction Stop)
Write-Host "Remaining items: $($Remaining.Count)"
```

Recorded: Remaining items: 0. Deleting the empty directory still failed due to the open handle.

After restarting Windows, the verified final block was run:

```powershell
$OldRoot = "$env:USERPROFILE\\Downloads\\local-speech-model-evaluation-verification-1.2.0"
$Inner = Join-Path $OldRoot "local-speech-model-evaluation-verification-1.2.0"

if (!(Test-Path $OldRoot)) {
    Write-Host "Already deleted."
    return
}

$Remaining = @(Get-ChildItem -LiteralPath $Inner -Force -ErrorAction Stop)

if ($Remaining.Count -ne 0) {
    throw "Folder is not empty. Stopping safely."
}

Remove-Item -LiteralPath $Inner -Force -ErrorAction Stop
Remove-Item -LiteralPath $OldRoot -Force -ErrorAction Stop

Write-Host "Old project exists: $(Test-Path $OldRoot)"
```

Recorded final output: Old project exists: False.

## 11. Separate repository and Desktop clone

The new public repository was created as nytro-gg/advanced-local-asr-benchmark. The original repository was restored to its pre-November file tree; a separate first project commit was made in the new repository.

The two local clones were organized under:

%USERPROFILE%\\Desktop\\Local Speech Model\\V2

Commands:

```powershell
$Base = "$env:USERPROFILE\\Desktop\\Local Speech Model\\V2"
$OldLocal = Join-Path $Base "local-speech-model-evaluation-github"
$NewLocal = Join-Path $Base "advanced-local-asr-benchmark"

git -C $OldLocal pull origin main
git clone https://github.com/nytro-gg/advanced-local-asr-benchmark.git $NewLocal
git -C $NewLocal status --short --branch
```

The first git pull attempt used the old Downloads path and failed with No such file or directory; using the Desktop path corrected it. The old clone fast-forwarded, the new clone downloaded, and its status was ## main...origin/main.

## 12. Final evidence boundaries

- Original model screenshots: historical experiments in evidence/.
- Reconstructed PowerShell exhibits: evidence/terminal/.
- No exact VAD diagnostic command transcript was preserved; only measured diagnostic results are reproduced.
- The matched November Small and Medium metrics are supported by experiment metadata summarized in the repository; they are not represented as live screenshots.
