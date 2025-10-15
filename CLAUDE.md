# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a 90-minute workshop repository teaching developers how to build **durable AI agents** using **[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) + Temporal**. The workshop is designed to run in GitHub Codespaces with zero local setup required.

[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) is critical to this entire workshop. All LLM tool calls must happen using this SDK.

All tool calls have to be real, no mocking. Use the [weather API](https://docs.temporal.io/ai-cookbook/tool-calling-python#create-the-activity-for-the-tool-invocation). All code uses **asyncio** for efficient I/O operations.

**Target Audience:** Beginner-intermediate Python developers
**Workshop Format:** 30 minutes instruction + 4×15 minute exercises

## Tech Stack

- **Python 3.11** - Primary language
- **openai agents sdk** - [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
- **temporalio** - Workflow orchestration for durability and retries
- **Temporal CLI** - Local development server
- **Development tools:** `rich`, `typer`, `pytest`, `ruff`, `mypy`
- **Interactive environment:** Jupyter notebooks for hands-on exercises
- **Deployment:** GitHub Codespaces (devcontainer-based)

## Common Commands

### Setup & Environment

```bash
make setup          # Install dependencies and set up environment
make env            # Validate environment variables (OPENAI_API_KEY)
```

### Development

```bash
make lint           # Run ruff linter
make test           # Run pytest test suite
```

### Temporal Server

```bash
temporal server start-dev     # Start Temporal local dev server (idempotent)
```

### Exercises

- All exercises and solutions live in `.ipynb` self-contained files that have all of the necessary code to work independently. No extra files should be required.

## Repository Architecture

### Four-Exercise Progression

The workshop follows a progressive learning path:

1. **Exercise 1 - Agent Hello World** (`exercises/01_agent_hello_world/`)

   - Minimal OpenAI Agents SDK usage using a real API call on [weather api](https://docs.temporal.io/ai-cookbook/tool-calling-python#create-the-activity-for-the-tool-invocation)
   - Demonstrates asyncio patterns with `async/await`

- Demonstrates basic agent patterns with built-in tools
- Available as Jupyter notebook (`exercise.ipynb`)
- No Temporal integration yet
- All asyncio from the start

2. **Exercise 2 - Temporal Hello World** (`exercises/02_temporal_hello_world/`)

   - 1 workflow + 1 activity in Python
   - Introduces Temporal concepts without AI complexity
   - Shows durability and retry mechanisms

3. **Exercise 3 - Durable Agent** (`exercises/03_durable_agent/`)

   - **Core integration:** The SAME Weather Agent from Exercise 1, now wrapped in Temporal activities for durability!
   - Workflow-based state persistence with asyncio throughout
   - Includes `trace_id` for observability correlation
   - Demonstrates retry logic and durability for AI operations
   - Demonstrates network disconnection and Temporal's ability to automatically retry and recover
   - All async: `await Runner.run()`, `async def` activities and workflows

4. **Exercise 4 - Multi-Agent Handoff** (`exercises/04_multi_agent_handoff/`)
   - Advanced: Multiple specialized agents from OpenAI Agents SDK similar to https://openai.github.io/openai-agents-python/quickstart/#put-it-all-together
   - Triage agent routes queries to specialists of the WebSearchTool and another agent that we create
   - Context maintained across agent handoffs
   - Workflow orchestration of agent-to-agent transitions

### Directory Structure

```
exercises/     # Starter code for workshop participants
solutions/     # Complete reference implementations
slides/        # Workshop presentation materials
scripts/       # Bootstrap, environment checks, Temporal startup
.devcontainer/ # Codespaces configuration
```

Each exercise directory contains:

- Jupyter notebooks for interactive learning (where applicable)
- `README.md` with: Goal, Steps (≤5), Expected Output, Stretch Goal, Timebox

## Key Architectural Patterns

### Durability Through Activities

In Exercise 3, LLM calls are wrapped in **Temporal activities** rather than called directly in workflows. This provides:

- Automatic retries on failure
- State persistence across crashes
- Replay-safe execution
- Example of stopping the internet to show Temporal's durability

### Observability Integration

The durable agent implementation connects two observability systems:

- **Temporal UI:** Shows workflow execution, retries, state
- **OpenAI Traces:** Shows LLM calls and tool usage
- **`trace_id`:** Correlation key printed to link both systems

### Teaching-First Code Style

All code follows these principles:

- **Clarity over abstraction:** Linear, readable flow
- **Verbose naming:** Self-documenting variable and function names
- **Instructional logging:** Log activity starts/ends, retries, workflow resumptions
- **Explanatory comments:** Focus on _why_ (durability, retries, state), not _what_
- **With valuable comments** A valuable comment for each line of code. No comments for imports or no comments for logging statements.

## Environment & Secrets

### Required Environment Variables

- `OPENAI_API_KEY` - Required for exercises 1, 3, 4

### Security Model

- `.env.sample` provides template (never commit `.env`)
- `scripts/check_env.py` validates environment before exercises run

## DevEx & Codespaces

### Bootstrap Flow

1. Codespaces creates container from `.devcontainer/`
2. Post-create: `scripts/bootstrap.sh` runs automatically
   - Installs Python dependencies
   - Installs Temporal CLI
   - Validates environment
3. Ready to run exercises in ≤90 seconds

### Idempotent Temporal Startup

`scripts/run_temporal.sh` checks if Temporal is already running. If yes, exits cleanly without error.

## Working with This Repository

### When Adding New Exercises

- Follow the 15-minute timebox constraint
- Include README with 5 or fewer steps
- Mirror in `/solutions/` with complete, commented implementation

### When Modifying Existing Code

- Maintain teaching clarity - avoid clever abstractions
- Add logs for observable events (retries, state changes, tool calls)
- Update corresponding solution if changing exercise starter code
- Ensure comments explain the "why" behind durability patterns
- Don't use yellow as a logging color

### When Debugging

- Check `make env` output for missing environment variables
- Verify Temporal is running: `make temporal-up`
- Exercise 3 prints Temporal UI URL and `trace_id` for correlation
- Look for activity retry logs in Temporal UI

## CI/CD

GitHub Actions workflow (`.github/workflows/ci.yml`):

- Runs on push/PR
- Tests on Python 3.11
- Executes: `ruff` → `mypy` → `pytest -q`
