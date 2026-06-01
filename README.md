# NotebookLM Codex Skill

Use Google NotebookLM from Codex through local browser automation.

This is a Codex-oriented port of `cclank/notebooklm-skill`. It keeps the Python automation scripts and rewrites the skill instructions so Codex can use them directly from `~/.codex/skills/notebooklm`.

## What It Can Do

- Authenticate to Google NotebookLM with a visible browser.
- Create NotebookLM notebooks.
- Upload local files and folders.
- Add URLs, YouTube links, and text sources.
- Save and search a local notebook library.
- Ask questions against NotebookLM notebooks and return source-grounded answers.

Supported upload formats include PDF, TXT, MD, DOCX, and DOC.

## Install For Codex

Clone this repository into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
cd ~/.codex/skills
git clone https://github.com/PANASV/notebooklm-skill.git notebooklm
```

Then start a new Codex conversation and ask for NotebookLM work, for example:

```text
Use $notebooklm to check my NotebookLM authentication status.
```

If you clone from another fork or local path, keep the folder name as `notebooklm` unless you also update references in your own workflows.

## First Run

Run all commands from the skill directory:

```bash
cd ~/.codex/skills/notebooklm
python scripts/run.py auth_manager.py status
```

The `scripts/run.py` wrapper creates `.venv`, installs dependencies, and runs the requested script. Do not call scripts directly.

Set up Google login:

```bash
python scripts/run.py auth_manager.py setup
```

A browser window opens. Log in manually, then the skill stores browser state locally under `data/`.

## Common Commands

Create a notebook:

```bash
python scripts/run.py upload_sources.py create --name "Project Docs"
```

Upload files:

```bash
python scripts/run.py upload_sources.py upload \
  --files "/path/to/doc.pdf,/path/to/notes.md" \
  --create-notebook "Project Docs"
```

Upload a folder:

```bash
python scripts/run.py upload_sources.py upload-dir \
  --directory "/path/to/docs" \
  --extensions "pdf,md,txt,docx,doc" \
  --create-notebook "Documentation"
```

Ask a notebook:

```bash
python scripts/run.py ask_question.py \
  --question "What are the key API authentication steps?" \
  --notebook-url "https://notebooklm.google.com/notebook/..."
```

Manage the local library:

```bash
python scripts/run.py notebook_manager.py list
python scripts/run.py notebook_manager.py search --query "api"
python scripts/run.py notebook_manager.py activate --id notebook-id
```

## Local Data

The skill stores runtime data inside this repository:

```text
data/
  library.json
  auth_info.json
  browser_state/
```

This directory can contain cookies, auth state, and personal notebook metadata. It is ignored by git and must remain private.

## Optional Configuration

Create `.env` in the skill directory:

```env
HEADLESS=false
SHOW_BROWSER=false
STEALTH_ENABLED=true
TYPING_WPM_MIN=160
TYPING_WPM_MAX=240
DEFAULT_NOTEBOOK_ID=
```

## Notes

- NotebookLM UI changes can break browser selectors. Use `--show-browser` to debug.
- NotebookLM usage limits and upload limits are controlled by Google and can change.
- For detailed commands, see `references/api_reference.md`.
- For recovery steps, see `references/troubleshooting.md`.

## Attribution

Based on:

- `cclank/notebooklm-skill`
- `PleasePrompto/notebooklm-skill`
- `PleasePrompto/notebooklm-mcp`
