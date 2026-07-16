# <span style="color:#2563eb">Faster-Whisper GPU Plug-and-Play Manual</span>

<blockquote>
<b>Matches:</b> <code>C:\Users\YourName\Desktop\faster-whisper-command-builder\faster-whisper-command-builder.html</code><br>
<b>For:</b> Windows setup at <code>C:\Users\YourName\Desktop\Tools\faster-whisper\faster-whisper</code><br>
<b>Installed version checked:</b> <code>faster-whisper 1.2.1</code><br>
<b>GPU profile:</b> <code>NVIDIA GeForce GTX 1050, 3 GB VRAM</code><br>
<b>Important:</b> The command builder now defaults to CPU. Use this manual only when you intentionally choose GPU.<br>
<b>GPU mode:</b> <code>device="cuda"</code>, <code>compute_type="int8"</code><br>
<b>Main official docs:</b> <a href="https://github.com/SYSTRAN/faster-whisper">https://github.com/SYSTRAN/faster-whisper</a>
</blockquote>

---

## <span style="color:#16a34a">1. How This Differs From The Builder Default</span>

The command builder default is CPU:

| Setting | Builder Default | GPU Manual Override |
|---|---|---|
| Runtime profile | CPU | GPU |
| Device | `cpu` | `cuda` |
| Model | `medium` | `medium`, but use `small` if VRAM runs out |
| Compute type | `int8` | `int8` |
| Forced language | `zh` | `zh` |
| Initial prompt | `请使用简体中文转写。` | `请使用简体中文转写。` |
| Pyannote device | `cpu` | Keep `cpu` unless you intentionally test GPU diarization |

On a 3 GB GTX 1050, `medium`, `large-v3-turbo`, `large-v3`, and `distill-large-v3.5-eng-only` may run out of GPU memory. If that happens, switch back to CPU or use `small`.

---

## <span style="color:#16a34a">2. Start Here</span>

Open PowerShell. Change only `$Audio` if your file is somewhere else:

```powershell
$Python = "C:\Users\YourName\Desktop\Tools\faster-whisper\faster-whisper\Scripts\python.exe"
$Audio = "C:\Users\YourName\Desktop\audio.mp3"
```

---

## <span style="color:#9333ea">3. Choose One GPU Model Block</span>

### Builder-Matching GPU Setting

This keeps the builder's model default but changes the device to CUDA.

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\medium"
$Device = "cuda"
$ComputeType = "int8"
```

### Safer GTX 1050 Setting

Use this if `medium` gives a GPU memory error.

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\small"
$Device = "cuda"
$ComputeType = "int8"
```

### Larger Models To Try Carefully

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\large-v3-turbo"
$Device = "cuda"
$ComputeType = "int8"
```

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\large-v3"
$Device = "cuda"
$ComputeType = "int8"
```

```powershell
$ModelName = "C:\Users\YourName\Desktop\Tools\faster-whisper\models\distil-large-v3.5"
$Device = "cuda"
$ComputeType = "int8"
```

---

## <span style="color:#dc2626">4. Output And Options</span>

These match the command builder defaults.

```powershell
$OutputSuffix = ".txt"

$TranscribeOptions = @{
    beam_size = 5
    vad_filter = $true
    language = "zh"
    initial_prompt = "请使用简体中文转写。"
}
```

For SRT or VTT output:

```powershell
$OutputSuffix = ".srt"
```

```powershell
$OutputSuffix = ".vtt"
```

For translation to English:

```powershell
$TranscribeOptions.task = "translate"
```

---

## <span style="color:#2563eb">5. Final Run Block</span>

Copy this after Sections 2, 3, and 4.

```powershell
$CublasBin = $Python -replace "\\Scripts\\python\.exe$", "\Lib\site-packages\nvidia\cublas\bin"
$env:PATH = "$CublasBin;$env:PATH"
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

## <span style="color:#dc2626">6. Troubleshooting</span>

| Problem | What To Try |
|---|---|
| GPU memory error | Use the `small` GPU block, or switch to the CPU manual |
| CUDA/driver error | Use the CPU manual |
| Wrong language detected | Change `language = "zh"` to the correct code, or remove it for auto detect |
| Need English output from non-English speech | Add `$TranscribeOptions.task = "translate"` |
| Repeated text appears | Add `$TranscribeOptions.condition_on_previous_text = $false` |
| Need diarization | Keep Pyannote device on `cpu` first; GPU diarization can use extra VRAM |

---

## <span style="color:#2563eb">7. Official References</span>

- Official project: <https://github.com/SYSTRAN/faster-whisper>
- Python package: <https://pypi.org/project/faster-whisper/>
- Model downloads: <https://huggingface.co/Systran>
