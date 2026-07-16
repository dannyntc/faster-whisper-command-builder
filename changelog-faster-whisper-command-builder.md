# Changelog

All notable local changes for the Faster-Whisper command builder, its HTML file, and related manuals are recorded here.

## 2026-06-29

### Organized Project Files

- Created a dedicated folder:
  - `C:\Users\YourName\Desktop\faster-whisper-command-builder`
- Moved the related files into that folder:
  - `faster-whisper-command-builder.html`
  - `faster-whisper-plug-and-play-manual CPU.md`
  - `faster-whisper-plug-and-play-manual-GPU.md`
  - `changelog-faster-whisper-command-builder.md`
- Moved the HTML file from:
  - `C:\Users\YourName\Desktop\faster-whisper-command-builder.html`
- To:
  - `C:\Users\YourName\Desktop\faster-whisper-command-builder\faster-whisper-command-builder.html`
- Updated both manuals so their `Matches:` line points to the HTML file in the new folder.

### Command Builder CPU Defaults

- Changed the main runtime profile default from GPU to CPU.
- Changed the Pyannote diarization device default from auto/GPU-preferred behavior to CPU.
- Updated the initial HTML dropdown state so CPU is visibly selected for:
  - Runtime profile
  - Pyannote device
- Updated JavaScript defaults so `Reset defaults` restores:
  - `profile: "cpu"`
  - `pyannoteDevice: "cpu"`

### CPU Manual

- Rewrote `faster-whisper-plug-and-play-manual CPU.md` to match the current command builder defaults.
- Documented the builder's default transcription setup:
  - Device: `cpu`
  - Model: `medium`
  - Compute type: `int8`
  - Output suffix: `.txt`
  - Language: `zh`
  - Initial prompt: `请使用简体中文转写。`
  - VAD/silence filtering enabled
- Replaced older generic model references with the local model folder paths used by the command builder.
- Added output options for `.txt`, `.srt`, and `.vtt`.
- Added notes for translation, language auto-detection, repeated text reduction, and hotwords.

### GPU Manual

- Rewrote `faster-whisper-plug-and-play-manual-GPU.md` so it no longer presents GPU as the default path.
- Clarified that the command builder now defaults to CPU and GPU is an intentional override.
- Kept GPU instructions available for CUDA use with:
  - Device: `cuda`
  - Compute type: `int8`
- Added GTX 1050 VRAM guidance:
  - Try `medium` to match the builder's model default.
  - Fall back to `small` if GPU memory errors occur.
  - Treat `large-v3-turbo`, `large-v3`, and `distill-large-v3.5-eng-only` as models to test carefully.
- Preserved the builder defaults for:
  - Language: `zh`
  - Initial prompt: `请使用简体中文转写。`
  - VAD enabled
  - Pyannote device staying on CPU first

### Verification

- Confirmed both manuals include the updated CPU-first defaults.
- Confirmed no obsolete hardcoded one-line commands remain using:
  - `device='cuda'`
  - `device='cpu'`
  - `WhisperModel('small'...)`

## 2026-06-27

### Same-Folder Output Default

- Updated `faster-whisper-command-builder.html` so generated transcription output is saved beside the input file by default.
- Updated the single-file hint to say the generated command saves output beside the input file.
- Updated the multi-file hint to say each output is saved beside its matching input file.
- Confirmed output-writing logic uses the input path and changes only the suffix:
  - `.txt`
  - `.srt`
  - `.vtt`
- Confirmed the builder already had local model choices including:
  - `small`
  - `distill-large-v3.5-eng-only`
  - `medium`
  - `large-v3-turbo`
  - `large-v3`

### Chinese Transcription Defaults

- Updated the command builder so first load and `Reset defaults` use:
  - Language: `zh`
  - Initial prompt: `请使用简体中文转写。`
- Updated the language dropdown so `Chinese, zh` is selected by default.
- Updated the initial prompt textarea with the Simplified Chinese prompt.
- Verified the generated command path reflects the Chinese language and prompt defaults.
- Ran a JavaScript syntax check after the change.

## 2026-06-26

### Diarization Workflow

- Added optional Pyannote diarization support to `faster-whisper-command-builder.html`.
- Added controls for:
  - Enabling diarization
  - Speaker count mode
  - Minimum and maximum speaker counts
  - Pyannote model selection
  - Custom Pyannote model id
  - Pyannote device
  - Hugging Face token
  - Diarization output folder
- Generated diarization commands call the separate environment:
  - `C:\Users\YourName\Desktop\Tools\faster-whisper-diarization\.venv\Scripts\python.exe`
  - `C:\Users\YourName\Desktop\Tools\faster-whisper-diarization\diarize_with_faster_whisper.py`
- Added generated flags for:
  - `--model`
  - `--device`
  - `--compute-type`
  - `--beam-size`
  - `--pyannote-model`
  - `--pyannote-device`
  - `--language`
  - `--min-speakers`
  - `--max-speakers`
  - `--output-dir`
- Added user-facing warnings for Hugging Face token requirements, translation limitations, SRT/VTT limitations, and batch output-folder behavior.

### Diarization Speed And Device Options

- Added a Pyannote device selector with `auto`, `cuda`, and `cpu`.
- Added `--pyannote-device` to generated diarization commands.
- Added `--no-word-timestamps` to generated diarization commands for faster segment-level diarization.
- Removed the visible fast-diarization checkbox after deciding fast segment-level mode should always be on.
- Changed visible wording from CUDA to GPU while keeping `cuda` as the internal command value.
- Added warnings for:
  - Hugging Face token requirements
  - Diarization not supporting translation output in the same way as normal transcription
  - Diarized output using `diarized.txt` and `diarized.json`, not SRT/VTT
  - GPU memory risk on GTX 1050
  - Pyannote GPU use on GTX 1050
  - Batch diarization ignoring custom output folder and using each input file folder

### Dependency Notes

- Verified the HTML is self-contained and does not depend on external CDN/script/link/http imports.
- Kept runtime dependencies outside the HTML:
  - faster-whisper environment
  - faster-whisper-diarization environment
  - Hugging Face token/model access
  - Local model folders

## 2026-06-25

### Manual Restructure

- Rewrote the original Faster-Whisper plug-and-play manual into a modular copy-as-you-go manual.
- Structured the manual around:
  - Python path and input file setup
  - Model choice
  - Transcript or translation task
  - Optional add-ons
  - Final run block
- Fixed PowerShell-to-Python option passing by base64-encoding the JSON options in PowerShell and decoding them in Python.
- Dry-tested the Python invocation, argument order, and encoded options.

### Command Builder Creation

- Created `faster-whisper-command-builder.html` as a standalone local HTML/CSS/JavaScript command builder.
- Built the first version from the CPU and GPU manuals found on the Desktop.
- Added a two-column UI with:
  - Input controls on the left
  - Generated PowerShell command and summary on the right
- Added controls for:
  - Input file path
  - CPU/GPU runtime profile
  - Model name
  - Compute type
  - Task: normal transcript or translate to English
  - Output suffix
  - Beam size
  - Language forcing
  - Silence filtering
  - More aggressive silence removal
  - Minimum silence duration
  - Repeated-text reduction
  - Initial prompt/names/technical terms
  - Hotwords
  - Word timestamps
  - Copy-ready PowerShell output
- Added `Copy command` and `Reset defaults` behavior.

### Builder Usability Updates

- Replaced the model free-text/preset setup with one model dropdown.
- Added simplified helper text for optional add-ons.
- Clarified:
  - `Minimum silence duration, ms`
  - `Names or technical terms`
  - `Hotwords`
- Reworked the Hotwords explanation to clarify:
  - Hotwords act as a recognition hint.
  - They can improve matching when audio sounds similar.
  - They do not force words to appear if not spoken.
  - They are useful for names, brands, acronyms, technical terms, and unusual phrases.

### Model And Output Updates

- Added local model folder mapping for downloaded models, including:
  - `small`
  - `medium`
  - `large-v3`
  - `large-v3-turbo`
  - `distill-large-v3.5-eng-only`
- Added output support for:
  - Plain text `.txt`
  - SubRip subtitles `.srt`
  - WebVTT subtitles `.vtt`
- Added command-side text cleanup before writing output.

### Multi-File Input

- Added single-file and multi-file input modes.
- Added support for adding and removing multiple input path rows.
- Added paste handling that can split multiple copied file paths into separate rows.
- Added cleanup for pasted Windows paths wrapped in quotes.
- Updated generated commands to loop through all selected input files.

### PowerShell Command Reliability And GPU Runtime Fixes

- Installed or verified CUDA 12 cuBLAS/NVRTC runtime packages inside the faster-whisper environment.
- Confirmed `cublas64_12.dll`, `cublasLt64_12.dll`, and related CUDA runtime DLLs were present.
- Added the cuBLAS package folder to the Windows user `PATH`.
- Updated generated commands to prepend the cuBLAS `bin` folder to `$env:PATH` for each run.
- Derived the cuBLAS folder from the configured Python path.
- Switched generated PowerShell execution to pipe the embedded Python code into Python with:
  - `$Code | & "...python.exe" -`
- Added instructions warning that the command must be run in PowerShell, not Command Prompt.
- Verified CUDA model loading with the local `large-v3-turbo` model.

### Progress Output

- Tested that faster-whisper can emit progress while consuming generated segments.
- Changed generated code so it collects segments while reporting progress, instead of waiting until all segments are converted to a list.
- Added live percentage progress output.
- Simplified progress output to just the current percentage after user feedback.
- Later added elapsed processing time next to progress:
  - `Progress:  12% | Elapsed: 03:41`
- Added `time.monotonic()` timing and a `format_elapsed()` helper in the generated Python code.
