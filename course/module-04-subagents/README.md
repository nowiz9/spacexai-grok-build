# Module 4: Agent Loops & Context Management

Welcome to the final module. We have a robust runtime, strict schemas, and secure tooling. Now, we must govern the mind of the agent.

How does an agent actually "think" over time? How does it remember what it did 5 minutes ago? How does it solve complex problems without forgetting the original goal?

---

## 1. The Immutable State: Append-Only Session Logs

A common mistake beginners make is maintaining state as a mutable string that they just keep appending to and modifying.

```python
# BAD AI MEMORY
context = "User asked for a script."
context = context + " I wrote the script."
```

Grok Build uses an **Append-Only Session Log** (found in `crates/codegen/xai-grok-session-events`).

The memory of the agent is a sequence of discrete, immutable events.
*   `UserMessage`
*   `ToolCall`
*   `ToolResult`
*   `AssistantMessage`

**The Golden Invariant: "Model-Visible Means Logged"**
If the LLM sees data, that data MUST be permanently logged in the session. If the LLM sees a tool result, but the framework crashes before saving it to disk, upon reboot, the LLM will remember something the framework forgot. This causes a split-brain hallucination.

By keeping an append-only log, Grok Build can reconstruct the exact state of the agent at any point in time, allowing for seamless pausing, resuming, and time-travel debugging.

**Conceptual JSON Representation:**
```json
[
  { "type": "UserMessage", "content": "Find the bug in waterfall.rs" },
  { "type": "ToolCall", "tool": "search_tool", "args": {"query": "waterfall"} },
  { "type": "ToolResult", "output": "..." }
]
```

---

## 2. The Turn Flow (The Agent Loop)

The agent does not just "run." It executes in a rigid state machine known as the **Turn Flow**.

1.  **Ingest:** New events (user input, tool results) are appended to the session.
2.  **Compress (Context Building):** The framework reads the massive Session Log. It knows the LLM cannot handle 1 million tokens. So it *compresses* the history. It truncates old tool outputs, summarizes old conversations, and creates a dense, optimized prompt.
3.  **Generate:** The optimized prompt is sent to the LLM (via the Tokio runtime).
4.  **Parse:** The strictly-typed response is validated against our schemas.
5.  **Execute:** If a tool was called, execute it. Loop back to step 1.

---

## 3. Subagents: Conquering Token Bloat

Imagine you ask the agent: *"Refactor the entire networking stack."*

If a single agent tries to do this, its session log will fill up with thousands of `read_file` and `bash` commands. After 10 minutes, the context window will be maxed out (Token Bloat), and the agent will forget the original goal.

**The Solution: Hierarchical Subagents** (`crates/codegen/xai-grok-subagent-resolution`)

Instead of doing the work itself, the Main Agent acts as a Manager. It spawns a Subagent.

### How a Subagent Works
1.  **Isolation:** The Main Agent spawns a Subagent with a *fresh, empty context window*.
2.  **Delegation:** The Main Agent says: *"Subagent, here is your only goal: Find the IP routing bug in `src/net/`. Do not worry about anything else."*
3.  **Execution:** The Subagent uses tools, reads files, and makes mistakes. Its context window gets messy.
4.  **Resolution:** When the Subagent finds the bug, it writes a clean summary: *"I found the bug on line 42. It is a null pointer."*
5.  **Termination:** The Subagent is killed. Its messy context window is destroyed (Garbage Collection).
6.  **Return:** The Main Agent receives *only the clean summary*.

By using Subagents, the Main Agent's memory remains pristine. It only remembers the high-level goals and the final results, allowing it to orchestrate massive refactors that would otherwise be impossible.

---

<div align="center">
  <b>Congratulations. You now understand the deep systems engineering behind Grok Build. You are no longer a beginner; you are on the path to becoming an Agentic Engineer.</b>
</div>
