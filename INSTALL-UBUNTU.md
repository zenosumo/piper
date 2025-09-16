# Piper TTS on Ubuntu (CLI Setup)

This guide explains how to install **Piper TTS** on Ubuntu (tested on 24.04 LTS), manage voices, and use it effectively from the command line.  
We assume you want a clean setup in your home directory, without Docker.

---

## 1. Install prerequisites

```bash
sudo apt update
sudo apt install -y python3-venv python3-pip espeak-ng-data alsa-utils ffmpeg
```

- **python3-venv / pip** → isolate Python environment  
- **espeak-ng-data** → phonemization library used by Piper  
- **alsa-utils / ffmpeg** → playback tools (`aplay`, `ffplay`)  

---

## 2. Create a dedicated Python environment

```bash
mkdir -p ~/.local
python3 -m venv ~/.local/piper-venv
source ~/.local/piper-venv/bin/activate

pip install --upgrade pip
pip install piper-tts
```

- **What is it?** `venv` builds a self-contained directory that holds its own Python interpreter and packages so the system Python stays clean.
- **Activate:**
  ```bash
  source ~/.local/piper-venv/bin/activate
  ```
  Your shell prompt typically changes, and `python` / `pip` now reference the virtual environment.
- **Deactivate:**
  ```bash
  deactivate
  ```
  This command is provided by the activation script and returns you to the regular shell environment.

Whenever you need Piper manually:
```bash
source ~/.local/piper-venv/bin/activate
```

---

## 3. Voice management

Piper uses voice models in **ONNX format** plus a small `.json` config.  

Create a folder for them:
```bash
mkdir -p ~/piper-voices
```

### List all available voices
```bash
python -m piper.download_voices | less
```

### Download a specific voice
```bash
python -m piper.download_voices --download-dir ~/piper-voices en_US-amy-low
```

You’ll get:
```
~/piper-voices/en_US-amy-low.onnx
~/piper-voices/en_US-amy-low.onnx.json
```

Repeat for as many voices as you want.

### Download all American English voices
```bash
python -m piper.download_voices | awk '/^en_US-/{print $1}' \
  | while read vid; do
      python -m piper.download_voices --download-dir ~/piper-voices "$vid"
    done
```

---

## 4. Basic usage (without wrapper)

Run Piper directly:
```bash
echo "Hello from Piper" | piper \
  --model ~/piper-voices/en_US-amy-low.onnx \
  --output_file hello.wav

aplay hello.wav
```

---

## 5. Using the `piper` wrapper script

We keep a `piper` wrapper in this repo (`~/.local/bin/piper`).  
It simplifies voice selection, batch demos, and cleanup.  

### Verify setup
```bash
piper which
```

### Speak a sentence
```bash
piper say "Hello Kris, Piper is ready."
```

### Choose a specific model
```bash
piper say -m ~/piper-voices/en_US-amy-medium.onnx "This is the medium model."
```

### Extra Piper flags
Pass them after `--`:
```bash
piper say -m ~/piper-voices/en_US-amy-medium.onnx \
  -- "Custom params" --noise-scale 0.3 --length-scale 1.05
```

### Batch demo (all US English voices)
```bash
piper demo-us "Testing all American English voices."
```

### List voices
- Local (downloaded):
  ```bash
  piper list local
  ```
- Full catalog:
  ```bash
  piper list all
  ```
- Filter:
  ```bash
  piper list all en_US
  ```

### Download voices from wrapper
```bash
piper download en_US-amy-low en_US-amy-medium
```

---

## 6. Temporary files and cleanup

- All generated WAVs are written to `/tmp` by default.  
- The wrapper auto-deletes any `piper_*.wav` older than **1 hour**.  
- Override location with `$TMPDIR`.

---

## 7. Customization

- Default voice: set `$PIPER_VOICE` env var.  
- Voice folder: set `$VOICE_DIR` (defaults to `~/piper-voices`).  
- Custom sample text for `demo-us`:  
  ```bash
  TEXT_EN_US="This is my default demo sentence." piper demo-us
  ```

---

## 8. Tips

- **Low vs Medium vs High voices**:  
  - Low = smaller, faster, less natural  
  - Medium = balanced  
  - High = large (~100 MB+), slower, higher quality
- Resample output if playback cracks:  
  ```bash
  ffmpeg -y -i hello.wav -ar 48000 hello48.wav
  ```
- For long texts, pipe from a file:  
  ```bash
  piper --model ~/piper-voices/en_US-amy-low.onnx --output_file story.wav < story.txt
  ```

---

## 9. Disk space

- Low voices: ~30 MB  
- Medium voices: ~60 MB  
- High voices: ~100+ MB  
- All American English voices together: ~1.5 GB

---

## 10. Updating

To update Piper:
```bash
source ~/.local/piper-venv/bin/activate
pip install --upgrade piper-tts
```

To refresh voices:
```bash
piper download VOICE_ID --force-redownload
```

---
