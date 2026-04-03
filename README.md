# Hollowgreen Code

Personal agentic coding assistant. Planner + Coder + Verifier pipeline over a local LM Studio model. Web UI + terminal. No cloud.

---

## What it does

You give it a terse command. A **Planner** agent breaks it into steps. A **Coder** agent executes each step using file and shell tools. A **Verifier** agent checks the work. The whole thing runs against whatever model you have loaded in LM Studio — fully local, no API keys.

```
hgc "add a /health endpoint to app.py"

[RUN    ] Orchestrator › Command: add a /health endpoint to app.py
[PLAN   ] Orchestrator › 3 step(s): (1) Read app.py → (2) Add /health route → (3) Write updated file
[STEP   ] Orchestrator › Step 1/3: Read app.py...
[TOOL   ] Coder › read_file({"path": "app.py"})
...
[VERIFY ] Verifier › VERDICT: PASS
```

---

## Requirements

- Python 3.10+
- [LM Studio](https://lmstudio.ai) running locally with a model loaded
- Flask (`pip install flask`)

Recommended models: Qwen3 32B, Qwen3 30B MoE, anything with solid tool-call support.

---

## Install

```powershell
git clone https://github.com/ThatReid23/Hollowgreen-coder
cd Hollowgreen-coder
pip install -e .
```

---

## Usage

**Interactive terminal (default):**
```powershell
python -m hollowgreen_code
```

**Single command:**
```powershell
python -m hollowgreen_code "build a Flask todo API with JSON storage" --mode python
```

**Web UI** *(half baked — not fully working yet)*:
```powershell
python -m hollowgreen_code --web
# opens http://localhost:7821
```

**Flags:**

| Flag | Default | Description |
|---|---|---|
| `--mode` | `python` | Coding mode: `python` `html` `react` `powershell` `cpp` `bash` |
| `--model` | *(picker)* | Model ID to use, skips interactive selector |
| `--url` | `http://localhost:1234/v1` | LM Studio base URL |
| `--dir` | `.` | Project working directory |
| `--web` | off | Launch web UI on port 7821 *(WIP — not fully working)* |
| `--port` | `7821` | Port for web mode |
| `--no-verify` | off | Skip the Verifier pass |

---

## Tools available to agents

| Tool | What it does |
|---|---|
| `read_file` | Read a file |
| `write_file` | Write a file (creates dirs) |
| `append_file` | Append to a file |
| `patch_file` | Replace a specific string in a file |
| `delete_file` | Delete a file |
| `list_dir` | List directory contents |
| `find_files` | Glob search for files |
| `grep_files` | Regex search across files |
| `run_command` | Run a shell command (PowerShell on Windows) |

All file tools are sandboxed to the working directory. `run_command` has a whitelist and a 30s timeout.

---

## Skills

Drop a `skills/` folder into your project directory with `SKILL.md` files to inject project-specific context into agent prompts. Built-in skills exist for `python`, `react`, `flask`, and `always` (loaded every run).

```
your-project/
  skills/
    python/
      SKILL.md   ← loaded when mode=python
    always/
      SKILL.md   ← always loaded
```

---

## Session files

Every run saves a `hgc_session_<timestamp>.json` to your working directory with the full plan, step summaries, and verification verdict.

---

## Structure

```
hollowgreen_code/
  lm_client.py        ← LM Studio API wrapper
  skills.py           ← skill loader
  core/
    agent.py          ← base agent + tool-call loop
    agents.py         ← Planner, Coder, Verifier
    orchestrator.py   ← coordinates the pipeline
  tools_pkg/
    tools.py          ← all tool implementations + schemas
  ui/
    main.py           ← terminal entry point
    web.py            ← Flask web server
    hgc_ui.html       ← web UI
    colors.py         ← terminal color codes
  skills/             ← built-in skill files
```

---

## License

MIT
