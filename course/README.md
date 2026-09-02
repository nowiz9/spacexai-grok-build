# From Zero to Agentic Engineer

<div align="center">
  <h3>A Masterclass in Systems Engineering for Autonomous AI Agents</h3>
</div>

Welcome to the definitive course on building production-grade, autonomous AI agents. I am your instructor, and my goal is to guide you—step by step, from first principles—through the architecture, execution, and philosophy of **Grok Build** (`grok`), an enterprise-grade AI Agent framework written in Rust.

By the end of this course, you will not just know *how* to use an AI framework; you will intimately understand *why* it is designed the way it is. We will demystify the magic and examine the cold, hard logic behind state machines, async runtimes, deterministic tool calling, and context bounding.

We will break down the hardest concepts into plain English. There is no magic here. Only systems.

## Course Objectives and Philosophy

### 1. First Principles Thinking
We do not use jargon just for the sake of it. If we introduce an Event Loop, we will first ask: *Why can't we just execute code sequentially?* (Spoiler: Because IO is slow, and blocking a thread waiting for an LLM to respond is a sin).

### 2. Top-Down, Zoom-In Architecture
You cannot understand a leaf without understanding the tree. We will map out the 10,000-foot view of Grok Build using ASCII diagrams before we dive into a specific Rust file or struct.

### 3. Visual Grounding
Diagrams will serve as our map. State transitions, memory bounds, and data flows will be charted out to aid your understanding.

### 4. No Hallucinations
Every lesson is tied directly to the `grok-build` repository. We will reference actual code paths (e.g., `crates/codegen/xai-grok-shell/src/waterfall.rs`) to ground theory in reality.

### 5. Hands-On Application
Theory without application is just trivia. We will present thought experiments and conceptual code snippets to lock in your understanding.

---

## Prerequisites

Before we begin, you need a basic understanding of modern software engineering. However, let's briefly review the essentials.

### Rust & Strong Typing (The Shield)
Grok Build is written in **Rust**. Rust is not just a language; it is a philosophy of safety. Unlike Python or JavaScript where type errors are discovered at runtime (often crashing your program), Rust catches them at compile time.
*   **Why it matters for AI:** LLMs are non-deterministic (they hallucinate). We need a strictly deterministic runtime to catch and handle weird LLM outputs before they cause havoc. We use Enums, Structs, and Pattern Matching to force the AI into rigid, safe boundaries.

### Asynchronous Execution (The Juggler)
You will hear about `async`, `await`, and `Tokio` (Rust's asynchronous runtime).
*   **Analogy:** Imagine you are cooking. You put water on to boil (a network request to the LLM). Do you stand there staring at the pot (blocking), or do you start chopping vegetables while you wait (asynchronous)? The `Tokio` runtime is our master chef, ensuring no thread is ever sitting idle.

---

## The Modules

We will journey through four intensive modules. Take them one at a time.

### [Module 1: The Foundations & Tech Stack](./module-01-foundations/README.md)
We begin with the bedrock. Why Rust? What is the `Tokio` event loop? How do we use structural typing and Discriminated Unions to build an impenetrable fortress against AI hallucinations? We will also explore the modular, crate-based architecture of the repository.

### [Module 2: Core Architecture & Framework](./module-02-architecture/README.md)
Here, we dissect the `xai-grok-shell` agent runtime. We will study the "Holy Trinity": Context, Services, and Waterfalls. How do we build a system that can pause, resume, intercept events, and isolate different agents safely in memory?

### [Module 3: Tooling & The Retrieval Mechanism](./module-03-tooling-retrieval/README.md)
You cannot stuff a 70,000-file codebase into a prompt. We will tackle the Context Window problem. How does the agent search? We will analyze `search_tool`, secure execution environments, and how massive outputs are bounded and paginated before being fed back to the LLM.

### [Module 4: Agent Loops & Context Management](./module-04-subagents/README.md)
The mind of the agent. We will explore the Immutable State (Append-Only Session Logs) and the Turn Flow. Finally, we will cover Subagents: how Grok spawns isolated mini-agents to solve complex problems without suffering from token bloat.

---

<div align="center">
  <b>Class is now in session. Proceed to <a href="./module-01-foundations/README.md">Module 1</a>.</b>
</div>
