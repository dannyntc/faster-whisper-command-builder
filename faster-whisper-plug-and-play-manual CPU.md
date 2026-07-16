# <span style="color:#2563eb">Faster-Whisper CPU Plug-and-Play Manual</span>

<blockquote>
<b>Matches:</b> <code>C:\Users\YourName\Desktop\faster-whisper-command-builder\faster-whisper-command-builder.html</code><br>
<b>For:</b> Windows setup at <code>C:\Users\YourName\Desktop\Tools\faster-whisper\faster-whisper</code><br>
<b>Installed version checked:</b> <code>faster-whisper 1.2.1</code><br>
<b>Default mode:</b> <code>device="cpu"</code>, <code>compute_type="int8"</code>, <code>model="medium"</code><br>
<b>Main official docs:</b> <a href="https://github.com/SYSTRAN/faster-whisper">https://github.com/SYSTRAN/faster-whisper</a>
</blockquote>

---

## <span style="color:#16a34a">1. What Changed In The Command Builder</span>

The command builder now defaults to CPU everywhere it matters:

| Setting | Current Default |
|---|---|
| Runtime profile | `CPU` |
| Device passed to faster-whisper | `cpu` |
| Model | `medium` |
| Compute type | `int8` |
| Task | Normal transcript |
| Output suffix | `.txt` |
| Forced language | `zh` |
| Initial prompt | `请使用简体中文转写。` |
| Silence filtering | On |
| Pyannote diarization device | `cpu` |

Use this CPU manual when you want the safest command path and do not want CUDA/GPU memory issues.

---

## <span style="color:#16a34a">2. Start Here</span>

Open PowerShell. Change only `$Audio` if your file is not here:

```powershell
$Python = "C:\Users\YourName\Desktop\Tools\faster-whisper\faster-whisper\Scripts\python.exe"
$Audio = "C:\Users\YourName\Desktop\audio.mp3"
```

Common replacements:

```powershell
$Audio = "C:\Users\YourName\Desktop\meeting.mp4"
```

```powershell
$Audio = "C:\Users\YourName\Desktop\recording.wav"
```

---

## <span style="color:#9333ea">3. Choose One Model Block</span>

The command builder uses local model folders. Copy exactly one block.

### Default CPU Setting

This matches the command builder default.

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\medium"
$Device = "cpu"
$ComputeType = "int8"
```

### Faster Draft

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\small"
$Device = "cpu"
$ComputeType = "int8"
```

### Faster Large-Style Model

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\large-v3-turbo"
$Device = "cpu"
$ComputeType = "int8"
```

### Highest Accuracy

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\large-v3"
$Device = "cpu"
$ComputeType = "int8"
```

### English-Only Distilled Model

Use this for English transcription, not translation workflows.

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\distil-large-v3.5"
$Device = "cpu"
$ComputeType = "int8"
```

---

## <span style="color:#dc2626">4. Choose One Output Block</span>

### Plain Text Default

```powershell
$OutputSuffix = ".txt"
```

### SRT Subtitles

```powershell
$OutputSuffix = ".srt"
```

### WebVTT Subtitles

```powershell
$OutputSuffix = ".vtt"
```

---

## <span style="color:#dc2626">5. Transcription Options</span>

This block matches the command builder defaults: Chinese language forced, silence filtering on, and a Simplified Chinese prompt.

```powershell
$TranscribeOptions = @{
    beam_size = 5
    vad_filter = $true
    language = "zh"
    initial_prompt = "请使用简体中文转写。"
}
```

For auto language detection, remove the `language = "zh"` line.

For translation to English, add:

```powershell
$TranscribeOptions.task = "translate"
```

Optional add-ons:

```powershell
$TranscribeOptions.vad_parameters = @{
    min_silence_duration_ms = 500
}
```

```powershell
$TranscribeOptions.condition_on_previous_text = $false
```

```powershell
$TranscribeOptions.hotwords = "project term, OpenAI, faster-whisper"
```

---

## <span style="color:#2563eb">6. Final Run Block</span>

Copy this after Sections 2, 3, 4, and 5.

```powershell
$OptionsJson = $TranscribeOptions | ConvertTo-Json -Compress
$OptionsJsonBase64 = [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($OptionsJson))

$Code = @'
from faster_whisper import WhisperModel
import base64
import json
import pathlib
import re
import sys

audio = sys.argv[1]
model_name = sys.argv[2]
device = sys.argv[3]
compute_type = sys.argv[4]
output_suffix = sys.argv[5]
options_json = base64.b64decode(sys.argv[6]).decode("utf-8")
options = json.loads(options_json)

def clean_text(text):
    return re.sub(r"\s+", " ", text).strip()

def srt_time(seconds):
    milliseconds = int(round(seconds * 1000))
    hours, remainder = divmod(milliseconds, 3600000)
    minutes, remainder = divmod(remainder, 60000)
    secs, millis = divmod(remainder, 1000)
    return f"{hours:02}:{minutes:02}:{secs:02},{millis:03}"

def vtt_time(seconds):
    return srt_time(seconds).replace(",", ".")

def write_output(audio, segments):
    out = pathlib.Path(audio).with_suffix(output_suffix)
    if output_suffix.lower() == ".srt":
        lines = []
        for index, segment in enumerate(segments, start=1):
            text = clean_text(segment.text)
            if text:
                lines.extend([str(index), f"{srt_time(segment.start)} --> {srt_time(segment.end)}", text, ""])
        out.write_text("\n".join(lines), encoding="utf-8")
    elif output_suffix.lower() == ".vtt":
        lines = ["WEBVTT", ""]
        for segment in segments:
            text = clean_text(segment.text)
            if text:
                lines.extend([f"{vtt_time(segment.start)} --> {vtt_time(segment.end)}", text, ""])
        out.write_text("\n".join(lines), encoding="utf-8")
    else:
        out.write_text("\n".join(clean_text(segment.text) for segment in segments if clean_text(segment.text)), encoding="utf-8")
    return out

model = WhisperModel(model_name, device=device, compute_type=compute_type)
segments, info = model.transcribe(audio, **options)
collected_segments = list(segments)
out = write_output(audio, collected_segments)

print("Detected language:", info.language)
print("Language probability:", round(info.language_probability, 3))
print("Saved output to:", out)
'@

& $Python -c $Code $Audio $ModelName $Device $ComputeType $OutputSuffix $OptionsJsonBase64
```

---

## <span style="color:#16a34a">7. Complete Default Example</span>

```powershell
$Python = "C:\Users\YourName\Desktop\Tools\faster-whisper\faster-whisper\Scripts\python.exe"
$Audio = "C:\Users\YourName\Desktop\audio.mp3"

$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\medium"
$Device = "cpu"
$ComputeType = "int8"
$OutputSuffix = ".txt"

$TranscribeOptions = @{
    beam_size = 5
    vad_filter = $true
    language = "zh"
    initial_prompt = "请使用简体中文转写。"
}
```

Then run Section 6.

---

## <span style="color:#dc2626">8. Notes</span>

| Situation | What To Do |
|---|---|
| Wrong language detected | Change `language = "zh"` to the correct code, or remove it for auto detect |
| Need English output from non-English speech | Add `$TranscribeOptions.task = "translate"` |
| Need subtitle files | Set `$OutputSuffix` to `.srt` or `.vtt` |
| Repeated text appears | Add `$TranscribeOptions.condition_on_previous_text = $false` |
| CPU is too slow | Use the GPU manual only if CUDA is working reliably |

---

## <span style="color:#2563eb">9. Official References</span>

- Official project: <https://github.com/SYSTRAN/faster-whisper>
- Python package: <https://pypi.org/project/faster-whisper/>
- Model downloads: <https://huggingface.co/Systran>
