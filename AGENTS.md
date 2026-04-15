# pi-subagents

Third-party pi extension (`@tintinweb/pi-subagents`) — Claude Code-style autonomous sub-agents for pi. This is a local fork from `tintinweb/pi-subagents`.

## What It Does

Registers three LLM-callable tools (`Agent`, `get_subagent_result`, `steer_subagent`) and the `/agents` command. Agents run in isolated pi sessions with their own tools, system prompts, and models. Supports foreground, background, and parallel execution.

## Source Layout

```
src/
  index.ts              # Extension entry: tool registration, /agents command, event bus setup
  agent-manager.ts      # AgentManager — lifecycle, concurrency queue (default 4), session tracking
  agent-runner.ts       # Core run loop: spawns pi session, streams events, handles turn limits
  agent-types.ts        # Built-in agent types (explore, code, ask, etc.) + type registry
  custom-agents.ts      # Loads user-defined agents from .pi/agents/<name>.md (YAML frontmatter)
  default-agents.ts     # Default agent definitions
  model-resolver.ts     # Fuzzy model name → full model ID resolution
  context.ts            # Conversation forking (context inheritance)
  cross-extension-rpc.ts # Event bus RPC: subagents:rpc:ping, subagents:rpc:spawn
  group-join.ts         # GroupJoinManager — consolidates background agent completions
  memory.ts             # Persistent agent memory (project/local/user scopes)
  worktree.ts           # Git worktree isolation for agents
  output-file.ts        # Streams agent output to markdown files
  prompts.ts            # System prompt templates
  skill-loader.ts       # Loads .pi/skills/*.md into agent prompts
  types.ts              # Core type definitions (AgentRecord, AgentConfig, SubagentType, etc.)
  env.ts                # Environment helpers
  ui/
    agent-widget.ts     # TUI widget: spinners, token counts, status icons
```

## Build

```bash
npm install
npx tsc            # Compile to dist/
```

## Key Types

- `AgentRecord` — runtime state of a spawned agent (status, session, tokens, output)
- `AgentConfig` — agent type definition (system prompt, model, thinking level, tool restrictions)
- `SubagentType` — string enum of built-in types (explore, code, ask, etc.)

## Custom Agents

Place `.pi/agents/<name>.md` files with YAML frontmatter:
```yaml
---
model: sonnet
thinking: medium
disallowed_tools: [Bash]
skills: [my-skill]
---
System prompt content here.
```

## Extension API

- Emits lifecycle events: `subagents:created`, `started`, `completed`, `failed`, `steered`
- Other extensions can spawn agents via `subagents:rpc:spawn` event
- Emits `subagents:ready` on load
