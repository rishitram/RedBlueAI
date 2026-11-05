# red-vs-blue-starter

A hands-on starter project for learning how automated **code-scanning security
agents** work. You will run a small, deliberately-broken web app on your own
computer, then build an AI agent that reads the app's source code, finds a
SQL injection bug, and proves the bug is real — all on localhost.

This is the same idea as [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
or the [PortSwigger Web Security Labs](https://portswigger.net/web-security):
a safe, intentionally vulnerable target you attack in a controlled sandbox.

> ⚠️ **For local learning only.** The practice app has security holes *on
> purpose*. It binds to `127.0.0.1` (your machine only) and must **never** be
> put on a public server or the internet.

---

## What's in here

```
red-vs-blue-starter/
  README.md
  requirements.txt
  .env.example
  target_app/
    app.py            # the vulnerable practice app (the "blue" target)
  agent/
    llm.py            # talks to the AI model
    tools.py          # list_files / read_file / write_file / send_request
    attack_agent.py   # the "red team" scanner
    defense_agent.py  # the "blue team" fixer
    agent_loop.py     # a separate example of a complete agent loop
```

- **`target_app/app.py`** — a tiny Flask + SQLite app with a login route and a
  product search route. Both build their database queries by gluing your input
  straight into SQL, which is exactly how SQL injection happens.
- **`agent/`** — contains the local-model client, bounded tools, a scanner, a
  fixer, and a separate agent-loop example.

---

## Setup (total beginner, Windows or Mac)

You will need **Python 3.9+** installed. Check with `python --version`
(try `python3 --version` on Mac if `python` isn't found).

### 1. Get the code and open a terminal in this folder

```bash
cd red-vs-blue-starter
```

### 2. Create a virtual environment (a private sandbox for this project's packages)

**Windows (PowerShell):**
```powershell
python -m venv .venv
.\.venv\Scripts\Activate
```

**Mac / Linux:**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

When it works you'll see `(.venv)` at the start of your terminal line.

### 3. Install the required packages

```bash
pip install -r requirements.txt
```

### 4. Get your free Gemini key

This project defaults to **Google Gemini's free API tier** (the `openai`
library talks to it through Gemini's OpenAI-compatible endpoint, so no code
changes are needed).

1. Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey) and
   sign in with a Google account.
2. Click **Create API key**. No credit card is needed.
3. Copy the example settings file to a real one:

   **Windows (PowerShell):**
   ```powershell
   copy .env.example .env
   ```

   **Mac / Linux:**
   ```bash
   cp .env.example .env
   ```

4. Open `.env` and paste your key into `LLM_API_KEY`. The `LLM_BASE_URL` and
   `LLM_MODEL` are already set to Gemini's free defaults, so you don't need to
   change them.

> ⏱️ The free tier has **per-minute rate limits**. If you see a "rate limit"
> or `429` error, just wait a minute and run the agent again.

> 🤖 The default free model is `gemini-3.5-flash-lite`. If it's ever
> unavailable, a good backup is `gemini-3.8-flash`, or pick another Gemini
> Flash model listed as free at
> [ai.google.dev/pricing](https://ai.google.dev/pricing) and put its name in
> `LLM_MODEL`.

> 🔒 Your `.env` holds a secret key. Don't share it or commit it to git.

---

## Run it (two terminals)

You need the target app running in one terminal and the agent in another.
**Both terminals must have the virtual environment activated** (step 2 above).

### Terminal 1 — start the practice app

```bash
python target_app/app.py
```

It will start on `http://127.0.0.1:5000`. Leave this terminal running. You can
open that address in your browser to see a little help page.

### Terminal 2 — run the agent

Run from the repo root. With no argument it scans the `target_app` folder:

```bash
python agent/attack_agent.py
```

You can also point it at a different directory by passing it as an argument
(the path is relative to the repo root; anything outside the repo is refused):

```bash
python agent/attack_agent.py target_app
python agent/attack_agent.py agent
```

The agent prints which directory it's scanning, then every step so you can
watch it think: which tool it chose, what the tool returned, and its report.

The scanner is ready to use. It reads source only from the selected scan
directory and sends HTTP requests only to the local practice app.

---

## Run the attack agent

Run **`agent/attack_agent.py`** to scan the practice app, report the exact
vulnerable line, and confirm the SQL injection with a request to the local app.

---

## Build the defense agent (red team → blue team)

Run **`agent/defense_agent.py`** to build and verify a fixed copy of the app.
Its file tools are `read_file` and `write_file`.

The goal: it reads the vulnerable file, rewrites the unsafe SQL using
**parameterized queries** (the `?` placeholder style already in `seed_database`),
and writes the corrected code to a **new** file `target_app/app_fixed.py`. It
must **not** overwrite `app.py`.

Run it from the repo root (same CLI as the attack agent — the directory is a
hard boundary; it can only read/write inside it):

```bash
python agent/defense_agent.py
```

**Close the loop** — prove the fix actually works:

1. Stop the vulnerable app (Ctrl-C in Terminal 1).
2. Run the fixed app instead:
   ```bash
   python target_app/app_fixed.py
   ```
3. In Terminal 2, run the attack agent again:
   ```bash
   python agent/attack_agent.py
   ```
   The `' OR '1'='1` login bypass should now **fail** (HTTP 401) — the
   parameterized query treats the payload as literal text, not SQL.

---

## How to try the exploit yourself (to understand what the agent is doing)

Peek at the bug by hand so you know what "success" looks like. With the app
running:

**Bypass the login** (note the password turns the check into always-true):
```bash
curl -X POST http://127.0.0.1:5000/login \
  -H "Content-Type: application/json" \
  -d "{\"username\": \"admin\", \"password\": \"' OR '1'='1\"}"
```

**Reveal a hidden product** in search:
```bash
curl "http://127.0.0.1:5000/search?q=%27%20OR%201%3D1%20--%20"
```
(That URL-encoded query is `' OR 1=1 -- ` — the `OR 1=1` makes the row filter
always true and the trailing `-- ` comments out the rest of the query,
defeating the `hidden = 0` filter so the SECRET product shows up.)

If those return data they shouldn't, the injection works — and that's exactly
what your agent should discover and confirm on its own.

---

## A note on ethics

Only ever run this against the app in this repo, on your own machine. Finding
and exploiting vulnerabilities in systems you don't own or have permission to
test is illegal. The whole point of a sandbox like this is to learn safely.
