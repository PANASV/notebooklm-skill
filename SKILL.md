---
name: notebooklm
description: "Use when the user wants Codex to work with Google NotebookLM: authenticate, create notebooks, upload local files or directories, add URLs or text sources, save notebook metadata, search the local NotebookLM library, or ask questions against NotebookLM notebooks for source-grounded answers."
---

# NotebookLM

This skill lets Codex use Google NotebookLM through local browser automation. It can authenticate, create notebooks, upload sources, manage a local notebook library, and ask questions against NotebookLM.

## When To Use

Use this skill when the user:

- Mentions NotebookLM or provides a `https://notebooklm.google.com/notebook/...` URL.
- Asks to query a NotebookLM notebook or "ask my docs".
- Wants to upload local files, folders, notes, URLs, or YouTube links to NotebookLM.
- Wants to create or organize NotebookLM notebooks from Codex.
- Needs source-grounded research from documents already stored in NotebookLM.

## Execution Rules

- Run all commands from this skill directory, the directory containing this `SKILL.md`.
- Always invoke scripts through `python scripts/run.py <script> ...`; do not run scripts directly.
- The first run creates `.venv`, installs Python dependencies, and installs Patchright browser dependencies.
- Authentication requires a visible browser and manual Google login.
- Never print, copy, commit, or summarize raw values from `data/`, `.env`, cookies, or browser state.

## Data And Privacy

Runtime data is stored under this skill directory:

- `data/library.json`: notebook metadata and active notebook settings.
- `data/auth_info.json`: authentication status metadata.
- `data/browser_state/`: browser profile, cookies, and session state.

The `data/` directory is ignored by git and should stay local. For sensitive notebooks, confirm with the user before changing share settings or uploading confidential files.

## Core Workflow

1. Check auth:

```bash
python scripts/run.py auth_manager.py status
```

2. If unauthenticated, tell the user a browser window will open for Google login, then run:

```bash
python scripts/run.py auth_manager.py setup
```

3. Inspect saved notebooks when needed:

```bash
python scripts/run.py notebook_manager.py list
```

4. Perform the requested NotebookLM task with the script reference below.

## Script Reference

### Authentication

```bash
python scripts/run.py auth_manager.py status
python scripts/run.py auth_manager.py setup
python scripts/run.py auth_manager.py reauth
python scripts/run.py auth_manager.py clear
```

### Notebook Library

```bash
python scripts/run.py notebook_manager.py list
python scripts/run.py notebook_manager.py search --query "keyword"
python scripts/run.py notebook_manager.py activate --id notebook-id
python scripts/run.py notebook_manager.py stats
python scripts/run.py notebook_manager.py remove --id notebook-id
```

Add a notebook only when you have meaningful metadata:

```bash
python scripts/run.py notebook_manager.py add \
  --url "https://notebooklm.google.com/notebook/..." \
  --name "Descriptive name" \
  --description "What this notebook contains" \
  --topics "topic1,topic2,topic3"
```

If the user provides a NotebookLM URL but no metadata, first ask NotebookLM what it contains, then save the discovered metadata:

```bash
python scripts/run.py ask_question.py \
  --question "What is the content of this notebook? What topics are covered? Provide a concise overview." \
  --notebook-url "https://notebooklm.google.com/notebook/..."
```

### Ask Questions

```bash
python scripts/run.py ask_question.py --question "Your question"
python scripts/run.py ask_question.py --question "Your question" --notebook-id notebook-id
python scripts/run.py ask_question.py --question "Your question" --notebook-url "https://notebooklm.google.com/notebook/..."
python scripts/run.py ask_question.py --question "Your question" --show-browser
```

After each answer, compare NotebookLM's response to the user's original request. Ask follow-up questions with `ask_question.py` until the answer is complete enough, then synthesize the result for the user. Do not pass along NotebookLM output blindly.

### Create And Upload

```bash
python scripts/run.py upload_sources.py create --name "Notebook name"
```

Upload files:

```bash
python scripts/run.py upload_sources.py upload \
  --files "/path/to/doc.pdf,/path/to/notes.md" \
  --notebook-url "https://notebooklm.google.com/notebook/..."

python scripts/run.py upload_sources.py upload \
  --files "/path/to/doc.pdf" \
  --create-notebook "New notebook"
```

Upload a directory:

```bash
python scripts/run.py upload_sources.py upload-dir \
  --directory "/path/to/docs" \
  --extensions "pdf,md,txt,docx,doc" \
  --create-notebook "Documentation"
```

Add URLs or YouTube links:

```bash
python scripts/run.py upload_sources.py add-urls \
  --urls "https://example.com,https://youtube.com/watch?v=..." \
  --notebook-id notebook-id
```

Add text:

```bash
python scripts/run.py upload_sources.py add-text --text "Source text" --notebook-id notebook-id
python scripts/run.py upload_sources.py add-text --file "/path/to/content.txt" --notebook-id notebook-id
```

Supported file formats: PDF, TXT, MD, DOCX, DOC. NotebookLM limits can change; if an upload fails, inspect the script output and use smaller batches.

### Cleanup

```bash
python scripts/run.py cleanup_manager.py
python scripts/run.py cleanup_manager.py --confirm
python scripts/run.py cleanup_manager.py --preserve-library
```

## Optional Configuration

Create a local `.env` file in the skill directory when needed:

```env
HEADLESS=false
SHOW_BROWSER=false
STEALTH_ENABLED=true
TYPING_WPM_MIN=160
TYPING_WPM_MAX=240
DEFAULT_NOTEBOOK_ID=
```

Do not commit `.env`.

## Troubleshooting

- If auth fails, run `python scripts/run.py auth_manager.py reauth`.
- If a selector breaks after a NotebookLM UI update, retry with `--show-browser` and inspect the browser state.
- If the local environment is corrupt, remove `.venv/` and rerun a command through `scripts/run.py`.
- For detailed command options, read `references/api_reference.md`.
- For usage patterns, read `references/usage_patterns.md`.
- For recovery steps, read `references/troubleshooting.md`.
