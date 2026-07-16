# faster-whisper-command-builder

A lightweight, browser-based toolkit for generating copy-ready Windows PowerShell commands for Faster-Whisper transcription, translation, subtitle generation, and speaker diarization.

## Why I made this

I built this to make my life easier as a non-coder. Faster-Whisper is powerful, but the PowerShell and Python command blocks can become long, complicated, and easy to get wrong.

I also do not want to open and configure a separate transcription app every time I need to process audio or video. Instead, I choose the files and options once, copy one generated PowerShell script, paste it into PowerShell, and press Enter to start the work.

For multiple files, the generated script handles the batch in one run rather than making me build and execute separate commands one by one.

## What it includes

- A static, dependency-free HTML/JavaScript command builder.
- Single-file and multi-file batch input using Windows paths, including pasting multiple file paths at once.
- CPU or GPU runtime profiles, local Faster-Whisper model selection, and compute-type options.
- Transcription or translation-to-English workflows.
- Plain-text (`.txt`), SubRip subtitle (`.srt`), and WebVTT subtitle (`.vtt`) output.
- Language selection or auto-detection, with defaults tailored for Chinese transcription.
- Optional transcription cleanup controls, including silence filtering, repeated-text reduction, prompts for names or technical terms, and hotwords.
- Optional Pyannote speaker diarization with speaker-count, model, CPU/GPU, Hugging Face token, and output-folder options.
- A copy-ready PowerShell script that processes the selected files, reports progress, and saves output beside each input file.
- Matching CPU and GPU setup manuals.
