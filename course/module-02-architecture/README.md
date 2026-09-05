# Module 2: Core Architecture & Framework

Welcome to Module 2. Now that we have established our foundation of Rust, strict typing, and asynchronous event loops, it is time to build the engine.

In this module, we will deconstruct the runtime architecture of Grok Build. We will examine the `xai-grok-shell` crate, which serves as the brain and nervous system of the agent.

---

## 1. The Framework: The Nervous System

When you build an AI agent, you quickly realize you are not just building a chat loop. You are building an Operating System.

An OS needs a way to manage memory, isolate processes, handle hardware IO (tools), and manage lifecycles. In the Node.js/TypeScript ecosystem, frameworks like **Cordis** or **LangChain** attempt to solve this. In Grok Build, the `xai-grok-shell` crate acts as this core framework.

The framework must solve one critical problem: **State and Event routing.** How does a tool execution deeply nested in a subagent notify the UI to render a spinner, while simultaneously logging the action to disk, without the components directly knowing about each other?

---

## 2. The Holy Trinity: Context, Services, and Waterfalls

To solve the routing problem, Grok Build uses a triad of architectural patterns.

### I. Context
The `Context` is the universe the agent lives in. It is a dependency injection container. When the agent boots, it creates a Context and injects all the necessary *Services* into it.

### II. Services
A Service is a distinct, isolated piece of logic.
*   **Examples:** The `ToolRegistry` (manages tools), the `SessionManager` (manages conversation state), the `Config` service.
Instead of passing a dozen variables to every function, functions request what they need from the Context.

*Thought Experiment:*
```rust
// Conceptually, a function only asks for the Context
async fn execute_turn(ctx: &Context) {
    // The function pulls exactly what it needs from the Context
    let tools = ctx.get::<ToolRegistry>();
    let session = ctx.get::<SessionManager>();

    // Do work...
}
```

### III. Waterfalls (Events)
The most beautiful part of the architecture is the **Waterfall** event system (found in `crates/codegen/xai-grok-shell/src/waterfall.rs`).

A Waterfall is a synchronous or asynchronous event hook. Components can *emit* events, and other components can *intercept* and modify those events as they flow down the waterfall.

**ASCII Diagram: The Waterfall**
```text
[Event: "Before Tool Execute"]
       |
       v
  (Interceptor: Permission Check - "Is this command safe?") --> [Deny! Stops Waterfall]
       |
       v
  (Interceptor: Logger - "Record command to disk")
       |
       v
[Tool Executes]
```

By examining `waterfall.rs`, we see stages like `CHILD_ENTER`, `SESSION_SPAWN`, `TURN_DONE`, and `FLUSH_DONE`. These are discrete moments in time where plugins can hook into the agent's lifecycle without modifying the core code.

---

## 3. Lifecycles & Isolation: Spatiotemporal Composability

A massive problem in agent engineering is **isolation**.

If Main Agent A spawns Subagent B to search the web, and Subagent B spawns Subagent C to run a Python script, how do we ensure Subagent C doesn't corrupt the memory of Agent A?

We need **Spatiotemporal Composability**.
*   **Spatial (Space):** Components are isolated in their own scopes.
*   **Temporal (Time):** Components have clear lifecycles (Start, Pause, Resume, Kill).

### The Forked Context
When Grok Build spawns a subagent, it doesn't give the subagent the root Context. It *forks* the Context.

The subagent gets a new, isolated sandbox. It has its own memory buffer and its own scoped tools. If the subagent crashes, or hallucinates and loops infinitely, the root Context can simply drop the subagent's Context, instantly garbage collecting it and stopping the task.

This prevents cross-contamination. The main agent maintains its pristine state, observes the failure of the subagent, and decides how to proceed.

---

<div align="center">
  <b>The engine is running. Let's give it hands. Proceed to <a href="../module-03-tooling-retrieval/README.md">Module 3: Tooling & The Retrieval Mechanism</a>.</b>
</div>
