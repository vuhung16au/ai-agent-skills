# LANGGRAPH-SKILL.md

## Purpose

Guide AI coding agents in designing and implementing LangGraph workflows. This skill covers graph structure, state design, node responsibilities, deterministic transitions, observability, and error recovery.

---

## Scope / When to Use

Apply this skill when generating or modifying code that uses:
- `langgraph` for agent or multi-step workflow orchestration.
- `StateGraph` or `MessageGraph` graph definitions.
- Conditional edges and branching logic.
- Human-in-the-loop patterns with `interrupt_before` or `interrupt_after`.
- LangGraph persistence via checkpointers.

---

## Core Principles

- **State is the single source of truth.** All data flowing through the graph must be represented in the typed state schema.
- **Nodes are pure transformers.** Each node should receive state, produce a partial state update, and have no hidden side effects.
- **Edges define control flow.** Business logic about what happens next belongs in edge functions, not in nodes.
- **Prefer determinism.** Use conditional edges with explicit conditions over open-ended agent decisions where the control flow is predictable.
- **Observability is required.** Non-trivial graphs must be traceable.

---

## Required Behavior

### Graph and State Definition
- Define the state as a `TypedDict` with explicit field types.
- Use `Annotated` fields with `operator.add` (or custom reducers) for list fields that accumulate values across nodes.
- Define the graph with `StateGraph(StateType)` and compile before use.

```python
from typing import Annotated, TypedDict
import operator
from langgraph.graph import StateGraph, END

class AgentState(TypedDict):
    messages: Annotated[list, operator.add]
    context: str
    step_count: int

graph = StateGraph(AgentState)
```

### Node Design
- Give each node a single, clearly named responsibility.
- Node functions must accept `state: StateType` and return a `dict` containing only the fields being updated.
- Do not mutate the input state directly — return a new partial state dict.
- Keep nodes stateless with respect to external mutable objects; use the state schema for persistence.

```python
def call_model(state: AgentState) -> dict:
    response = llm.invoke(state["messages"])
    return {"messages": [response]}
```

### Edge Design
- Use `graph.add_edge(source, destination)` for unconditional transitions.
- Use `graph.add_conditional_edges(source, condition_fn, mapping)` for branching.
- Condition functions must return a string key that maps to a defined destination node or `END`.
- Define the `mapping` dict explicitly — do not rely on implicit fallthrough.

```python
def should_continue(state: AgentState) -> str:
    if state["step_count"] >= 10:
        return "end"
    if state["messages"][-1].tool_calls:
        return "tools"
    return "end"

graph.add_conditional_edges("agent", should_continue, {"tools": "tool_node", "end": END})
```

### Entry and Exit Points
- Always call `graph.set_entry_point(node_name)`.
- Every execution path must eventually reach `END` or a defined terminal node.
- Audit the graph for unreachable nodes before deployment.

### Compilation and Invocation
- Compile the graph before use: `app = graph.compile()`.
- Pass a checkpointer to `compile()` for persistent workflows: `graph.compile(checkpointer=checkpointer)`.
- Use `app.invoke(initial_state)` for synchronous execution or `app.astream()` for streaming.
- Always provide a `config` dict with a `thread_id` when using a checkpointer.

### Human-in-the-Loop
- Use `interrupt_before=["node_name"]` or `interrupt_after=["node_name"]` in `compile()`.
- After an interrupt, resume with `app.invoke(None, config={"thread_id": ...})` — do not re-invoke with the full initial state.
- Document every interrupt point with a comment explaining what human input is expected.

### Error Recovery
- Wrap node logic in try/except and return an error field in the state for recoverable errors.
- Define a dedicated error-handling node that the graph can route to on failure.
- Do not let unhandled exceptions propagate silently through the graph.

### Observability
- Enable LangSmith tracing for non-trivial graphs via `LANGCHAIN_TRACING_V2=true`.
- Use `run_name` in the config dict for identifiable traces.
- Log node entry/exit with the relevant state fields for debugging.

---

## Things to Avoid

- Storing mutable shared objects (database connections, file handles) directly in state.
- Writing nodes that have multiple responsibilities (violates single-responsibility principle).
- Using unbounded loops in the graph without a clear termination condition.
- Hardcoding LLM model names or API keys inside node functions.
- Building graphs without any error recovery path.
- Skipping the `set_entry_point` call.
- Using `MessageGraph` for workflows that need richer state than a message list.

---

## Output Expectations

Generated LangGraph code must:
- Define state as a `TypedDict` with type annotations.
- Use `Annotated` with reducers for accumulating fields.
- Have each node function return a partial state dict.
- Include a condition function and explicit mapping for every conditional edge.
- Include `graph.set_entry_point()` and ensure all paths reach `END`.
- Include a checkpointer reference if persistence is needed.
- Be compilable and invocable without modification.

---

## Optional Checklist

- [ ] State defined as `TypedDict` with typed fields.
- [ ] Accumulating fields use `Annotated[list, operator.add]` or a custom reducer.
- [ ] Every node returns a partial state dict (no direct mutation).
- [ ] All conditional edges have an explicit mapping.
- [ ] `set_entry_point` is called.
- [ ] Every execution path reaches `END`.
- [ ] Checkpointer configured for persistent workflows.
- [ ] LangSmith tracing enabled or documented.
- [ ] Error recovery path defined.
