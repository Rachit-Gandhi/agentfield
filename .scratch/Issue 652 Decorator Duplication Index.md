# Issue #652 — Reasoner/Skill Decorator Duplication

Trace map for [Agent-Field/agentfield#652](https://github.com/Agent-Field/agentfield/issues/652).

## Artifacts

- [[Issue 652 Decorator Duplication.canvas]] — Obsidian canvas (same nodes/edges)
- [issue-652-trace-graph.html](./issue-652-trace-graph.html) — draggable HTML graph (open in browser)

## Checkpoint

The Python SDK has **four decorator entry points** for reasoners/skills:

1. **`decorators.reasoner`** — module-level, tracking-only, no FastAPI routes
2. **`Agent.reasoner`** — full registration (endpoint + registry + tracked_func)
3. **`Agent.skill`** — parallel to reasoner with `/skills/` paths
4. **`AgentRouter.reasoner/skill`** — deferred staging; `include_router` delegates to Agent methods

**Four duplication hotspots** (red/issue edges in the graph):

| # | Pattern | Sites |
|---|---------|-------|
| 1 | `@decorator` vs `@decorator()` direct-registration | Agent, AgentRouter (reasoner + skill), decorators |
| 2 | Trigger merge + `accepts_webhook` resolution | `decorators.py:166`, `agent.py:1876` |
| 3 | Type-hint → `input_fields` loop | `agent.py` reasoner + skill decorators |
| 4 | FastAPI `@self.post` endpoint closure (~200 lines) | `agent.py` reasoner + skill decorators |

`Agent.include_router` is the bridge: it calls `self.reasoner(...)(func)` / `self.skill(...)(func)` and wires `router._tracked_functions` for lazy delegation.

**Proposed fix (issue):** extract a mixin with shared decorator factory logic, testable without instantiating `Agent(FastAPI)`.
