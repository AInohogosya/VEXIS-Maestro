# Configuration Guide

## Overview

VEXIS-Maestro reads an optional `config.yaml` from the project root to control runtime behavior (timeouts, safety toggles, Telegram mode, logging, etc.).

API keys and provider/model preferences are primarily supplied via environment variables or interactive selection in `run.py`. They are not persisted to disk by default (the in-memory settings manager is used during a single run).

## Quick Start

1. Copy the template:
   ```bash
   cp config.example.yaml config.yaml
   ```
2. Edit `config.yaml` to match your environment.
3. Run:
   ```bash
   python3 run.py "list files"
   ```

## config.yaml Reference

### api

```yaml
api:
  preferred_provider: "ollama"
  local_endpoint: "http://localhost:11434"
  local_model: "llama3.2:3b"
  openrouter_api_key: ""
  timeout: 30
  max_retries: 3
```

- `preferred_provider`: `ollama`, `google`, `openrouter`, `openai`, `anthropic`, `xai`, `meta`, `mistral`, `microsoft`, `amazon`, `cohere`, `deepseek`, `groq`, `together`, `minimax`, `zhipuai`
- `local_endpoint`: Ollama server URL
- `local_model`: default model name used as a fallback when a model is not explicitly selected
- `openrouter_api_key`: optional; can also be provided via environment variable
- `timeout`, `max_retries`: API request timeout and retry count

### execution

```yaml
execution:
  mode: "auto"
  safety_mode: true
  dry_run: false
  verify_commands: true
  command_timeout: 600
  task_timeout: 900
  max_iterations: 500
  auto_recovery: true
```

- `mode`: `auto`, `normal`, `telegram`
- `max_iterations`: maximum Phase 2–4 retry loops

### logging

```yaml
logging:
  level: "INFO"
  file: "vexis.log"
  json_format: false
  console: true
  max_file_size: 10485760
  backup_count: 5
```

### security

```yaml
security:
  enable_command_blocking: false
  enable_confirmation_prompts: false
  enable_sudo_warning: false
  enable_shell_pipe_warning: false
  enable_sandbox: true
```

### performance

```yaml
performance:
  max_concurrent_tasks: 1
  memory_limit_mb: 1024
  task_timeout: 900
  command_timeout: 600
  api_timeout: 30
```

### cache

```yaml
cache:
  enabled: true
  max_size: 1000
  ttl: 3600
  persist_to_disk: true
```

### cost

```yaml
cost:
  daily_budget: null
  monthly_budget: null
  per_request_budget: null
  warning_threshold: 0.8
  critical_threshold: 0.95
```

### telegram

```yaml
telegram:
  enabled: false
  bot_token: ""
  bot_username: ""
  api_id: 0
  api_hash: ""
  session_name: "vexis_telegram"
  authorized_users: []
  output_recipients: []
  enable_input_listener: true
  send_phase2_end_updates: true
  max_history_length: 50
```

### user

```yaml
user:
  name: ""
  preferred_style: "detailed"
  auto_confirm: false
  show_progress: true
```

### custom_system_prompt

```yaml
custom_system_prompt: ""
```

## Environment Variables

### API Keys

`run.py` and the providers recognize these environment variables:

```bash
GOOGLE_API_KEY=...
GROQ_API_KEY=...
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
XAI_API_KEY=...
META_API_KEY=...
MISTRAL_API_KEY=...
AZURE_API_KEY=...
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
COHERE_API_KEY=...
DEEPSEEK_API_KEY=...
TOGETHER_API_KEY=...
MINIMAX_API_KEY=...
ZHIPUAI_API_KEY=...
OPENROUTER_API_KEY=...
```

### Config Overrides

The config loader also supports a small set of `AI_AGENT_*` overrides:

```bash
AI_AGENT_LOG_LEVEL=INFO
AI_AGENT_LOG_FILE=vexis.log
AI_AGENT_LOG_JSON=false
AI_AGENT_LOCAL_ENDPOINT=http://localhost:11434
AI_AGENT_LOCAL_MODEL=llama3.2:3b
AI_AGENT_PREFERRED_PROVIDER=ollama
AI_AGENT_API_TIMEOUT=30
AI_AGENT_API_MAX_RETRIES=3
AI_AGENT_COMMAND_TIMEOUT=600
AI_AGENT_MAX_CONCURRENT_TASKS=1
AI_AGENT_TASK_TIMEOUT=900
```

## Validation

### Validate YAML Syntax

```bash
python3 -c "import yaml; yaml.safe_load(open('config.yaml'))"
```

### Environment Check

```bash
python3 run.py --check
```
