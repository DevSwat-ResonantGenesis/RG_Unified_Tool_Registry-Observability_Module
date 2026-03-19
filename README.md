<div align="center">

# RG Unified Tool Registry & Observability Module

### One canonical tool format for every system · Real-time observability for every call

**Built entirely by AI, orchestrated by [Louie Nemesh](https://resonantgenesis.xyz)**

[![License: RG Source Available](https://img.shields.io/badge/License-RG%20Source%20Available-blue.svg)](LICENSE.txt)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-3776AB.svg)](https://python.org)
[![Tools](https://img.shields.io/badge/Tools-130+-green.svg)](src/rg_tool_registry/builtin_tools.py)
[![Platform](https://img.shields.io/badge/Platform-resonantgenesis.xyz-purple.svg)](https://resonantgenesis.xyz)

</div>

---

## What is this?

**RG Unified Tool Registry** is the canonical tool definition and observability layer for the entire Resonant Genesis platform. It replaces **4 incompatible tool formats** with a single `ToolDef` dataclass that auto-converts to any provider format, and wraps every tool call with real-time latency tracking, success/failure metrics, and structured logging.

### The Problem It Solved

Before this module, the platform had 4 separate tool definition formats:

| System | Old Format | File |
|--------|-----------|------|
| Agentic Chat (registered users) | `TOOL_DEFS` dict | `routers_agentic_chat.py` |
| Public Chat (guests) | `GUEST_TOOLS` dict | `routers_public_chat.py` |
| Agent Executor | `tool_handlers` dict | `executor.py` |
| Resonant IDE | `ToolDef[]` TypeScript | `toolDefinitions.ts` |

Adding a new tool meant editing 4 files in 4 formats. Renaming a parameter meant 4 changes. Now it's **one `ToolDef`, one registration, available everywhere**.

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│             RG Unified Tool Registry                      │
│                                                           │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │  ToolDef    │  │  ToolRegistry │  │  ToolObserver   │  │
│  │  (canonical │  │  (central     │  │  (per-call      │  │
│  │   format)   │  │   store)      │  │   metrics)      │  │
│  └──────┬──────┘  └──────┬───────┘  └────────┬────────┘  │
│         │                │                    │           │
│  ┌──────┴──────────────────┴────────────────────┐        │
│  │              Format Converters                │        │
│  │                                               │        │
│  │  to_openai()    → OpenAI/Groq native FC      │        │
│  │  to_anthropic() → Anthropic native tools      │        │
│  │  to_prompt_text()→ JSON-mode text fallback    │        │
│  │  to_typescript() → IDE ToolDef[] codegen      │        │
│  └───────────────────────────────────────────────┘        │
│                                                           │
│  ┌───────────────────────────────────────────────────┐    │
│  │              Subsystems                            │    │
│  │                                                    │    │
│  │  NativeFCClient  — Multi-provider function calling │    │
│  │  ToolStreamer    — Streaming partial results        │    │
│  │  RegistryBuilder — Pre-configured per-system sets  │    │
│  └────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
         │              │               │             │
    ┌────▼────┐  ┌──────▼─────┐  ┌─────▼────┐  ┌────▼────┐
    │ Agentic │  │  Public    │  │  Agent   │  │Resonant │
    │  Chat   │  │   Chat    │  │ Executor │  │   IDE   │
    │(~100 t) │  │ (14 t)    │  │ (~25 t)  │  │(~61 t)  │
    └─────────┘  └───────────┘  └──────────┘  └─────────┘
```

---

## 130+ Built-in Tools (18 Categories)

| Category | Count | Examples |
|----------|-------|---------|
| **Search** | 6 | `web_search`, `fetch_url`, `deep_research`, `search_memory` |
| **Memory** | 8 | `store_memory`, `recall_memory`, `hash_sphere_store`, `memory_search` |
| **Code Analysis** | 10 | `code_visualizer_scan`, `code_visualizer_trace`, `code_visualizer_governance` |
| **Agents** | 12 | `create_agent`, `list_agents`, `run_agent`, `agent_teams` |
| **Community** | 6 | `rabbit_post`, `rabbit_comment`, `rabbit_list_communities` |
| **Media** | 5 | `generate_image`, `text_to_speech`, `speech_to_text` |
| **Integrations** | 6 | `google_calendar`, `google_drive`, `figma_*`, `sigma_*` |
| **Filesystem** | 8 | `read_file`, `write_file`, `list_files`, `search_files` |
| **GitHub** | 6 | `github_repo_info`, `github_create_issue`, `github_list_repos` |
| **Git** | 11 | `git_clone`, `git_status`, `git_commit`, `git_push`, `git_diff` |
| **Developer** | 4 | `validate_code`, `format_code`, `run_tests` |
| **Utilities** | 8 | `generate_uuid`, `get_time`, `calculate`, `convert_units` |
| **Platform API** | 10 | `platform_api_search`, `platform_api_call`, `dsid_lookup` |
| **Planning** | 5 | `todo_list`, `ask_user`, `save_memory`, `code_search` |
| **Terminal** | 6 | `terminal_create`, `terminal_send`, `terminal_read`, `terminal_wait` |
| **State Physics** | 4 | `state_transition`, `state_query`, `state_history` |
| **Deploy** | 2 | `droplet_ssh_command`, `droplet_docker_status` |
| **Email** | 3 | `send_email`, `list_emails`, `read_email` |

---

## Source Files

| File | Purpose | Lines |
|------|---------|-------|
| `registry.py` | `ToolDef`, `ToolParam`, `ToolRegistry`, `ToolCategory`, `ToolAccess`, format converters | ~450 |
| `observability.py` | `ToolObserver`, `ToolCallRecord`, `ToolStats`, metrics engine, structured logging | ~406 |
| `builtin_tools.py` | All 130+ platform tool definitions organized by category | ~495 |
| `builtin_tools_ide.py` | IDE-only tools (git, terminal, web, planning) | ~88 |
| `builder.py` | Pre-configured registry builders per system | ~92 |
| `native_fc.py` | Native function calling client for Groq/OpenAI/Anthropic/Gemini | ~653 |
| `streaming.py` | Tool result streaming with progress callbacks | ~340 |
| `__init__.py` | Module exports | ~34 |

---

## Quick Start

### Define a Tool

```python
from rg_tool_registry import ToolDef, ToolParam, ToolCategory, ToolAccess, ParamType

my_tool = ToolDef(
    name="weather_lookup",
    description="Get current weather for a location",
    category=ToolCategory.UTILITIES,
    params=[
        ToolParam("location", ParamType.STRING, "City name", required=True),
        ToolParam("units", ParamType.STRING, "celsius or fahrenheit", default="celsius"),
    ],
    access={ToolAccess.REGISTERED, ToolAccess.GUEST},
    priority=50,
)
```

### Register and Convert

```python
from rg_tool_registry import ToolRegistry

registry = ToolRegistry()
registry.register(my_tool)

# Convert to any provider format
openai_tools = registry.to_openai()       # OpenAI/Groq native function calling
anthropic_tools = registry.to_anthropic()  # Anthropic native tool format
prompt_text = registry.to_prompt_text()    # JSON-mode text fallback
typescript = registry.to_typescript()      # IDE TypeScript ToolDef[]
```

### Pre-built Registries

```python
from rg_tool_registry.builder import (
    build_registered_registry,  # Agentic chat (~100 tools)
    build_guest_registry,       # Public chat (14 tools)
    build_agent_registry,       # Autonomous agents (~25 tools)
    build_ide_registry,         # Resonant IDE (~61 tools)
    build_full_registry,        # Everything (130+ tools)
)

guest_registry = build_guest_registry()
openai_format = guest_registry.to_openai()
```

### Observability

```python
from rg_tool_registry import ToolObserver

observer = ToolObserver(system="agentic_chat")

# Context manager wraps any tool call
async with observer.observe("web_search", user_id="u123", session_id="s456") as ctx:
    result = await do_search(query)
    ctx.set_result(result)
    ctx.set_tokens(input_tokens=500, output_tokens=200)

# Decorator for automatic tracking
@observer.track("web_search")
async def handle_web_search(args, **kwargs):
    return await do_search(args["query"])

# Query metrics
stats = observer.get_tool_stats("web_search")
# → {"total_calls": 142, "success_rate": 0.972, "avg_latency_ms": 340.2, ...}

summary = observer.get_summary()
# → {"total_calls": 5420, "top_tools_by_calls": [...], "slowest_tools": [...]}
```

### Native Function Calling

```python
from rg_tool_registry.native_fc import NativeFCClient

fc_client = NativeFCClient(
    registry=build_guest_registry(),
    observer=ToolObserver(system="public_chat"),
)

# Automatically handles tool_calls → execution → result injection → re-prompting
response = await fc_client.run(
    messages=[{"role": "user", "content": "Search for AI governance"}],
    provider="groq",
    api_key="gsk_...",
)
```

### Tool Result Streaming

```python
from rg_tool_registry.streaming import ToolStreamer

streamer = ToolStreamer()

@streamer.streamable("code_visualizer_scan", estimated_seconds=60)
async def scan_repo(args, progress=None):
    progress(0.1, "Cloning repository...")
    repo = await clone(args["url"])
    progress(0.5, "Running AST analysis...")
    ast = await analyze(repo)
    progress(0.9, "Building graph...")
    return build_graph(ast)

# Stream partial results via SSE
async for event in streamer.execute_streaming("code_visualizer_scan", {"url": "..."}):
    yield sse_event(event["type"], event)
```

---

## Access Control

Tools are tagged with access levels controlling which systems can use them:

| Access Level | System | Typical Count |
|-------------|--------|---------------|
| `REGISTERED` | Agentic Chat (logged-in users) | ~100 tools |
| `GUEST` | Public Chat (no auth) | ~14 tools |
| `AGENT` | Autonomous Agent Executor | ~25 tools |
| `IDE` | Resonant IDE (local execution) | ~61 tools |
| `ALL` | Available everywhere | ~10 tools |

```python
# Get tools for a specific system
guest_tools = registry.get_tools(access=ToolAccess.GUEST)
agent_tools = registry.get_tools(access=ToolAccess.AGENT)

# Filter by category
search_tools = registry.get_by_category(ToolCategory.SEARCH)

# Filter by name
specific = registry.get_tools(names=["web_search", "fetch_url"])
```

---

## Observability Metrics

The `ToolObserver` tracks per-tool and per-system metrics:

| Metric | Description |
|--------|-------------|
| `total_calls` | Total invocations |
| `success_count` / `failure_count` | Pass/fail breakdown |
| `success_rate` | Success percentage |
| `avg_latency_ms` / `min` / `max` | Response time distribution |
| `total_result_chars` | Data volume returned |
| `total_tokens` | LLM token consumption |
| `top_errors` | Most common failure reasons |
| `last_called` | Timestamp of last invocation |

### Structured Logging

Every tool call emits a structured log line:
```
[TOOL] agentic_chat/web_search user=u123 session=s456 status=OK latency=340ms result=4200chars tokens=0 loop=2 provider=groq
```

Compatible with ELK, Loki, Datadog, or any JSON log aggregator.

---

## Installation

```bash
git clone https://github.com/DevSwat-ResonantGenesis/RG_Unified_Tool_Registry-Observability_Module.git
cd RG_Unified_Tool_Registry-Observability_Module
pip install -e .

# Or copy src/rg_tool_registry/ into your project
```

---

## Used By

This module is deployed across the entire Resonant Genesis backend:

| Service | Import | Purpose |
|---------|--------|---------|
| **Chat Service** (agentic) | `from rg_tool_registry.builtin_tools import build_registry` | 100+ tools for registered users |
| **Chat Service** (public) | `from rg_tool_registry.builder import build_guest_registry` | 14 tools for guests |
| **Agent Engine** | `from rg_tool_registry.builtin_tools import build_registry` | Tool loading for autonomous agents |
| **Agent Executor** | `from rg_tool_registry.observability import ToolObserver` | Per-tool metrics tracking |
| **IDE Completions** | `from rg_tool_registry.builder import build_ide_registry` | IDE tool definitions |

Docker volume mount: `/home/deploy/rg_tool_registry:/app/rg_tool_registry:ro` across `chat_service` and `agent_engine_service` containers.

---

## About the Creator

**RG Unified Tool Registry** is part of the **Resonant Genesis** platform, built entirely by AI and architected by **Louie Nemesh** starting November 11, 2025. Every line of code was written by AI.

---

## License

Copyright (c) 2025-2026 Resonant Genesis / DevSwat. Founded and built by Louie Nemesh.

Licensed under the [Resonant Genesis Source Available License](LICENSE.txt).

- **View & study**: Free for everyone
- **Download & use**: Free with [platform registration](https://resonantgenesis.xyz/signup)
- **Contribute**: Pull requests welcome
- **Commercial use**: [Contact us](https://resonantgenesis.xyz/contact)

---

<div align="center">

**Built on Resonant Genesis technology by Louie Nemesh**

</div>
