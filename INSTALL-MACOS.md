# Piper TTS Installation Guide (macOS, Apple Silicon M1/M2 Optimized)

This guide sets up Piper TTS cleanly and reproducibly on Apple Silicon (M1/M2).  
It includes phonemizer, playback tools, and Apple Silicon–friendly practices.

---

## 1. Install Homebrew (if not installed)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add Homebrew to your shell environment safely:

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

---

## 2. Update Brew & Install Dependencies

```bash
brew update
brew upgrade

# Core dependencies
brew install python espeak-ng ffmpeg
```

- `python`: Latest stable Python (>=3.9).  
- `espeak-ng`: Required for phonemization.  
- `ffmpeg`: Playback/resampling support.

---

## 3. Create & Activate Virtual Environment

```bash
$(brew --prefix python@3.11)/bin/python3.11 -m venv ~/.local/piper-venv
source ~/.local/piper-venv/bin/activate

# Upgrade pip toolchain
pip install --upgrade pip setuptools wheel
```

---

## 4. Install Piper TTS

```bash
pip install piper-tts
```

---

## 5. Install ONNX Runtime (CPU version for Apple Silicon)

```bash
pip install onnxruntime
```

> The CPU version is reliable on M1/M2. GPU acceleration for ONNX on macOS is experimental and usually not needed for Piper’s footprint.

---

## 6. Download Voices via Catalog Tool

Instead of hardcoding URLs, use the built-in downloader:

```bash
mkdir -p $HOME/Library/Application\ Support/piper/voices
python -m piper.download_voices --download-dir $HOME/Library/Application\ Support/piper/voices en_US-amy-medium
```

Available voices can be explored online:

- Catalog: https://github.com/rhasspy/piper/blob/master/VOICES.md
- Samples: https://rhasspy.github.io/piper-samples/


(Optional bulk installs)

Install **all American English voices**:

```bash
for voice in $(curl -s https://raw.githubusercontent.com/rhasspy/piper/master/VOICES.md | grep -Eo 'en_US-[a-z]+-(low|medium|high)'); do
  python -m piper.download_voices --download-dir $HOME/Library/Application\ Support/piper/voices $voice
done
```

Install **all British English voices**:

```bash
for voice in $(curl -s https://raw.githubusercontent.com/rhasspy/piper/master/VOICES.md | grep -Eo 'en_GB-[a-z]+-(low|medium|high)'); do
  python -m piper.download_voices --download-dir $HOME/Library/Application\ Support/piper/voices $voice
done
```

Install **all Italian voices**:

```bash
for voice in $(curl -s https://raw.githubusercontent.com/rhasspy/piper/master/VOICES.md | grep -Eo 'it_IT-[a-z]+-(low|medium|high)'); do
  python -m piper.download_voices --download-dir $HOME/Library/Application\ Support/piper/voices $voice
done
```


---

## 7. Test Piper

Generate and play speech:

```bash
mkdir -p "${TMPDIR:-/tmp}/piper" && \
echo "Hello from Piper on macOS M1!" | \
piper --model $HOME/Library/Application\ Support/piper/voices/en_US-amy-medium.onnx --output_file "${TMPDIR:-/tmp}/piper/piper_test.wav"
afplay "${TMPDIR:-/tmp}/piper/piper_test.wav"
```

---

## 8. Fix Audio Crackling (Optional)

If audio playback crackles, resample to 48 kHz:

```bash
ffmpeg -y -i "${TMPDIR:-/tmp}/piper/piper_test.wav" -ar 48000 "${TMPDIR:-/tmp}/piper/piper_test_48k.wav"
afplay "${TMPDIR:-/tmp}/piper/piper_test_48k.wav"
```

---

## 9. Wrappers for Easier Use (Optional)

Add shortcuts in `~/.zshrc`:

```bash
alias piper-say='mkdir -p "${TMPDIR:-/tmp}/piper" && piper --model $HOME/Library/Application\ Support/piper/voices/en_US-amy-medium.onnx --output_file "${TMPDIR:-/tmp}/piper/piper_out.wav" && afplay "${TMPDIR:-/tmp}/piper/piper_out.wav"'
```

Then:

```bash
echo "Custom voice shortcut working!" | piper-say
```

---

## 10. Cleanup Temporary Files

Piper creates `/tmp/piper_*.wav`. You can clean them manually:

```bash
rm -f "${TMPDIR:-/tmp}/piper/piper_*.wav"
```

---

✅ You now have a stable, optimized Piper TTS setup for Apple Silicon (M1/M2).
