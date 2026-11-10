# RedBlueAI

**AI-Powered Security Testing and Automated Vulnerability Remediation**

RedBlueAI uses AI-powered red-team and blue-team agents to detect, test, and remediate SQL injection vulnerabilities in a Flask/SQLite application. Built with Python, Google Gemini, and GitHub Actions, the project demonstrates automated security testing and AI-assisted code remediation.

> 🚧 **Work in Progress:** RedBlueAI is under active development, with ongoing improvements to agent reliability, automated testing, and infrastructure.

> **Security Notice:** The target application is intentionally vulnerable and must only be run locally in an authorized testing environment.

## Features

- **Red-Team Agent:** Analyzes Python source code to detect SQL injection vulnerabilities and tests findings against a local Flask application.
- **Blue-Team Agent:** Generates secure code fixes using parameterized SQL queries while preserving the original application.
- **AI Agent Automation:** Uses Google Gemini with structured tool calls to automate vulnerability detection and remediation.
- - **OWASP-Aligned Security Testing:** Detects and tests SQL injection vulnerabilities based on security risks covered by the OWASP Top 10, using AI agents to automate vulnerability identification and remediation.
- **GitHub Actions CI/CD:** Automatically scans pull requests and posts security findings as PR comments.
- **Restricted Tool Access:** Limits agent file operations to project directories and HTTP requests to localhost.
- **Testing and Verification:** Supports comparing vulnerable and corrected application behavior to evaluate security fixes.

## Tech Stack

| Category | Technologies |
|---|---|
| Languages | Python, SQL |
| AI | Google Gemini, OpenAI-compatible API |
| Backend | Flask, SQLite |
| CI/CD | GitHub Actions, Git |
| Security | SQL Injection, Parameterized Queries, Restricted Agent Tools |

## Architecture

```text
                  RedBlueAI
                      |
             Google Gemini API
                      |
              AI Agent System
                      |
             +--------+--------+
             |                 |
       Red-Team Agent    Blue-Team Agent
             |                 |
       Scan Source Code   Analyze Findings
             |                 |
       Test SQL Injection Generate SQL Fixes
             |                 |
       Report Findings    Save app_fixed.py
             |                 |
             +--------+--------+
                      |
               Flask / SQLite
               Test Application
```

## Project Structure

```text
RedBlueAI/
├── .github/
│   └── workflows/
│       └── security-scan.yml
├── agent/
│   ├── attack_agent.py
│   ├── defense_agent.py
│   ├── agent_loop.py
│   ├── ci_scan.py
│   ├── llm.py
│   └── tools.py
├── target_app/
│   └── app.py
├── .env.example
├── requirements.txt
└── README.md
```

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/rishitram/RedBlueAI.git
cd RedBlueAI
```

### 2. Install Dependencies

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

For Windows PowerShell, activate with:

```powershell
.\.venv\Scripts\Activate.ps1
```

### 3. Configure Environment Variables

Create a `.env` file using `.env.example` and configure your API credentials:

```env
LLM_API_KEY=your_api_key
LLM_BASE_URL=your_openai_compatible_endpoint
LLM_MODEL=your_model_name
```

Never commit your `.env` file or API credentials.

### 4. Start the Flask Application

```bash
python target_app/app.py
```

The test application runs at `http://127.0.0.1:5000`.

### 5. Run the Red-Team Agent

In a separate terminal:

```bash
python agent/attack_agent.py
```

The agent scans the application, identifies potential SQL injection vulnerabilities, and reports its findings.

### 6. Run the Blue-Team Agent

```bash
python agent/defense_agent.py
```

The agent generates a corrected version of the application at `target_app/app_fixed.py`.

### 7. Verify Remediation

Stop the original application, review the generated code, and start the fixed version:

```bash
python target_app/app_fixed.py
```

Compare the original and corrected application's behavior. The scanner targets the original source by default, so testing the generated file requires directing the source analysis to that file or checking the fixed application's responses separately.

## GitHub Actions Integration

The CI/CD workflow in `.github/workflows/security-scan.yml` integrates automated security analysis into pull requests.

- Triggers security scans on pull requests.
- Uses `agent/ci_scan.py` to analyze source code.
- Generates vulnerability reports.
- Posts security findings as pull request comments.

The workflow requires an `LLM_API_KEY` repository secret and should only be enabled with appropriate permissions and secret protections.

