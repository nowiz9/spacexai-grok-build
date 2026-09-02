# Module 3: Tooling & The Retrieval Mechanism

Welcome to Module 3. Our agent has a brain (the Framework) and memory (the Context). But right now, it is trapped in a void. It cannot see the codebase, and it cannot write files.

It is time to give our agent tools.

---

## 1. The Context Window Problem

A junior AI engineer often asks: *"Why don't we just write a script to concatenate every file in the repository into one giant string, and paste it into the LLM prompt?"*

Here is why:
1.  **Hard Limits:** Even with a 128k or 200k token window, enterprise codebases (like `grok-build` with 70,000+ files) will instantly blow past the limit.
2.  **The "Needle in a Haystack" Degradation:** Even if an LLM *can* accept 200k tokens, its reasoning degrades. The more irrelevant information you feed it, the more likely it is to hallucinate or miss the actual bug.
3.  **Cost and Latency:** Passing massive prompts costs money and takes seconds (or minutes) to process.

We must treat the LLM like a human engineer. A human doesn't read the whole repo; they use a **Search Engine**.

---

## 2. The Search Engine: `search_tool` and `grep`

If you look in `crates/codegen/xai-grok-tools/src/implementations/grok_build/`, you will see tools like `bash`, `read_file`, `grep`, and `search_tool`.

Let's look at `search_tool/types.rs` again:
```rust
pub struct SearchToolInput {
    pub query: String,
    pub limit: Option<u8>,
}
```

When the LLM needs to find how "waterfalls" work, it doesn't ask for the codebase. It emits a JSON payload calling `search_tool` with the query `"waterfall"`.

Grok Build uses tools like `ripgrep` (a blazing fast rust search utility) and AST (Abstract Syntax Tree) parsing under the hood. It searches the files, finds the matches, and returns *only the relevant snippets*.

---

## 3. Output Bounding: Handling Massive Results

What happens if the LLM calls `grep "pub fn"`, which matches 10,000 lines of code?

If we return all 10,000 lines to the LLM, we will crash the context window (Token Bloat). We need **Output Bounding**.

Grok Build dynamically intercepts the stdout of tools.

**ASCII Pipeline: Bounding Output**
```text
[LLM] -> { "tool": "grep", "args": "pub fn" }
           |
           v
      (Subprocess Execution)
           |
           v
[Raw Output: 10,000 lines]
           |
           v
 (Output Truncation / Bounding Mechanism) -> "Result truncated. Showing first 100 lines..."
           |
           v
[LLM Context Window]
```
If a tool output is too large, the system truncates it, saves the full output to a temporary "Spill Store" or file (like an artifact), and tells the LLM: *"The output was too large. I showed you the first 100 lines. If you want more, refine your search."*

---

## 4. Secure Execution: Sandboxing

When the LLM calls the `bash` tool (`crates/codegen/xai-grok-tools/src/implementations/grok_build/bash.rs`), it is literally asking your computer to execute arbitrary shell commands.

**This is terrifying.**

An LLM hallucinating `rm -rf /` or executing a malicious shell injection via a poisoned repo is a critical security vulnerability.

How do we securely execute?
1.  **Pty/Subprocesses:** Commands are not blindly piped into `sh -c`. They are carefully spawned as isolated subprocesses.
2.  **Sandboxing:** In advanced deployments, these commands run in isolated Docker containers or sandboxed environments (notice crates like `xai-grok-sandbox`).
3.  **No Shell Injection:** By strict parsing of arguments (using arrays of arguments rather than a single raw string where possible, or tightly controlled PTY sessions), the system mitigates injection attacks.

*Remember: The agent is a loaded weapon. The framework must be the safety catch.*

---

<div align="center">
  <b>The agent can now see and interact with the world safely. Proceed to <a href="../module-04-subagents/README.md">Module 4: Agent Loops & Context Management</a>.</b>
</div>
