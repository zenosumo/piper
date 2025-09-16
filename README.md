# Piper TTS CLI Toolkit

This repository provides helper scripts and wrapper commands for running [Piper TTS](https://github.com/rhasspy/piper) from the command line.

## Installation Guides

- [macOS CLI setup](./INSTALL-MACOS.md)
- [Ubuntu / Debian CLI setup](./INSTALL-UBUNTU.md)

> Have instructions for another platform? Open a PR or issue so we can link it here.

## Quick Usage

Once you have installed Piper and downloaded at least one voice model via the steps above, the wrapper script makes it easy to generate speech:

```bash
piper say "Hello from Piper"
```

Choose a specific model:
```bash
piper say -m ~/piper-voices/en_US-amy-medium.onnx "Using the medium model."
```

List available voices:
```bash
piper list local
piper list all en_US
```

If you need to work directly with the underlying CLI, you can still call `piper` manually:
```bash
echo "Hello" | piper \
  --model ~/piper-voices/en_US-amy-low.onnx \
  --output_file hello.wav
```

Generated audio defaults to your temporary directory (override with `$TMPDIR`). The wrapper periodically cleans `piper_*.wav` files older than an hour.

## Updating

When a new Piper release ships, activate your virtual environment and upgrade:

```bash
source ~/.local/piper-venv/bin/activate
pip install --upgrade piper-tts
```

Refresh downloaded voices when needed:

```bash
piper download VOICE_ID --force-redownload
```
