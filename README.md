# Global Pi configuration

Standalone repository for shared Pi configuration:

```text
settings.json
skills/
extensions/
prompts/
```

## Link on a new machine

Clone this repository to `~/pi-config`. Back up any existing
`~/.pi/agent/settings.json`, `skills`, `extensions`, and `prompts` first.
The commands below deliberately refuse to overwrite existing paths:

```sh
mkdir -p "$HOME/.pi/agent"
for name in settings.json skills extensions prompts; do
  ln -s "$HOME/pi-config/$name" "$HOME/.pi/agent/$name"
done
pi update --extensions
```

Directory symlinks do not merge existing local skills with repository skills.
To keep machine-specific skills, link shared skill directories individually
instead and merge the package entries into your local `settings.json`.

Only configuration and skill sources belong in Git, not `~/.pi/agent` as a whole.
External npm packages are pinned in `settings.json`; update pins deliberately.

## Vendored skills

Entire skill directories (including supporting scripts) are copied locally.

| Skill | Source | Commit |
| --- | --- | --- |
| handoff | https://github.com/mattpocock/skills/tree/main/skills/productivity/handoff | 24fe0ef7737efae15c87225755e9f6f5965e4888 |
| pdf-reader | https://github.com/espennilsen/pi/tree/main/skills/pdf-reader | 79d019bfc687e607b284fb92440c268c8dcb95ca |
| youtube-transcript | https://github.com/amosblomqvist/pi-config/tree/main/skills/youtube-transcript | f82da563ab05d66729492d64c7ed4e96db3663f3 |

`handoff` is explicit-only: `/skill:handoff`.
For PDF helper commands, resolve `scripts/extract.py` relative to the skill
folder, not the current project.

## Runtime requirements

- PDF text: Poppler (`pdftotext`, `pdfinfo`, etc.).
- PDF tables/helper script: Python with `pdfplumber`.
- Optional OCR: Tesseract plus Python `pytesseract` and `pdf2image`.
- YouTube transcript: Python and `yt-dlp`; upstream currently only fetches English captions.

Python dependencies and OCR are not installed by copying these skills.
Install system tools on Fedora:

```sh
sudo dnf install poppler-utils tesseract tesseract-langpack-deu tesseract-langpack-eng
```

Or on macOS:

```sh
brew install poppler tesseract tesseract-lang yt-dlp
```

Then create the Python environment separately on each machine:

```sh
python3 -m venv ~/.local/share/pi/venvs/pdf-reader
~/.local/share/pi/venvs/pdf-reader/bin/python -m pip install pdfplumber pytesseract pdf2image
```

Local customization: the PDF skill uses this venv's Python explicitly;
no activation is needed and system Python remains unchanged. German OCR
uses `--lang deu+eng`. The venv itself is outside Git.

Pi Web Access works without API keys using its default search providers.
Keep any provider keys outside this repository (e.g. environment variables).
