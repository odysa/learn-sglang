# Interactive Project Tutorial Generator: Full Workflow

Generate a comprehensive, publish-ready interactive HTML tutorial that explains how any software project works — from first principles to implementation details. The output is a single-file interactive website with animated visualizations, suitable for GitHub Pages.

## When to Use

- Explaining how a codebase works to newcomers (onboarding, open-source docs)
- Creating educational material for a framework, library, or system
- Producing a "how does X work under the hood" deep-dive for any project
- Building interactive documentation that goes beyond static READMEs

## Prerequisites

- Target project codebase checked out locally (or accessible via URL)
- User provides: project path/URL + target audience level (beginner / intermediate / expert)
- Optional: specific aspects to focus on (e.g., "focus on the scheduler" or "explain the plugin system")

## Inputs

```
PROJECT_PATH=<path to codebase or GitHub URL>
AUDIENCE=<beginner|intermediate|expert>
FOCUS=<optional: specific subsystem or aspect to emphasize>
OUTPUT=<output filename, default: tutorial.html>
```

---

## Phase 1: Deep Codebase Investigation

### Step 1.1: Architecture Discovery

Dispatch **1 subagent** to perform a broad architecture survey:

- Map the directory structure and identify major modules
- Read README, ARCHITECTURE.md, CONTRIBUTING.md, and any design docs
- Identify the entry points (main, CLI, server startup)
- Trace the primary data flow end-to-end (e.g., request → response, input → output)
- List the key abstractions: core data structures, interfaces, protocols
- Identify external dependencies and what role they play

**Output**: `architecture_survey.md` containing:
- High-level architecture diagram (text-based)
- Module map with purpose of each major directory
- Primary data flow trace
- List of 6-12 core concepts that need explaining

### Step 1.2: Concept Dependency Graph

From the architecture survey, build a **concept dependency graph** — determine the order in which concepts must be taught so each builds on the previous:

```
Example for a web framework:
  HTTP basics → Routing → Middleware → Request/Response → ORM → Templates → Auth
Example for an ML system:
  Tensors → GPU → Model → Attention → KV Cache → Batching → Scheduling
```

This graph determines the chapter order. Each concept becomes one chapter. Target: **8-16 chapters**.

### Step 1.3: Deep Dives per Concept

Dispatch **3 parallel subagents** to investigate the codebase in depth, split by concept groups:

- **Agent A**: Concepts 1–N/3 (foundational). Trace through code: key classes, methods, data structures. Record file paths and line numbers. Note non-obvious design decisions and WHY they were made.
- **Agent B**: Concepts N/3+1–2N/3 (intermediate). Same approach.
- **Agent C**: Concepts 2N/3+1–N (advanced). Same approach.

Each agent produces a structured trace:
```markdown
## Concept: <Name>

### What It Does
One-paragraph summary a newcomer can understand.

### Why It Exists
The problem it solves. What goes wrong without it.

### How It Works
Step-by-step explanation with code references (file:line).
Key data structures and their fields.
Key algorithms and their complexity.

### Source Code Snippets
For each core mechanism, extract the **actual source code** (not pseudocode) that
implements it. Each snippet should be:
- 10-40 lines (the essential logic, not boilerplate)
- Trimmed of irrelevant branches, error handling, and logging
- Annotated with inline comments explaining non-obvious lines
- Tagged with exact file path and line range: `# source: scheduler/prefill.py:142-178`

Include at minimum:
- The primary data structure definition (class/struct with key fields)
- The core algorithm (the "hot loop" or main dispatch logic)
- One non-obvious helper that reveals a design decision

Example format:
```python
# source: sglang/srt/managers/schedule_batch.py:45-62
@dataclass
class Req:
    input_ids: torch.Tensor      # prompt tokens (CPU)
    cached_len: int              # prefix already in KV cache (reused, not recomputed)
    output_len: int              # max tokens to generate
    cache_handle: Handle         # pointer into RadixCache
    sampling_params: SamplingParams

    @property
    def extend_len(self):
        # Only the NEW tokens need computation — this is where
        # RadixCache savings materialize
        return len(self.input_ids) - self.cached_len
```

### Key Design Decisions
Why this approach over alternatives. Trade-offs made.

### Interactive Demo Idea
A specific visualization that would build intuition:
- What the user inputs/clicks
- What animates/changes
- What insight it delivers
```

### Step 1.4: Fill Gaps

Dispatch **1 subagent** to read all Phase 1 outputs and identify:
- Missing links in the concept chain (concept A references concept B, but B isn't covered)
- Prerequisite knowledge the audience might lack
- Cross-cutting concerns (error handling, configuration, testing) worth a chapter

Add 1-3 supplementary concepts if needed.

**Output**: Complete concept list with dependency order, deep-dive traces, and interactive demo ideas for every chapter.

---

## Phase 2: Draft Writing

### Step 2.1: Plan the Chapter Structure

Every chapter follows this template:

```markdown
## N. Chapter Title
Subtitle: one-sentence hook.

### The Problem
What real-world problem does this concept solve?
Why can't you just ignore it?

### The Idea (Simplified)
Explain the core idea in 2-3 paragraphs.
Use analogies to everyday objects if AUDIENCE=beginner.
No code yet.

### How It Works (Detailed)
Step-by-step technical walkthrough.
Include pseudocode blocks showing the key algorithm.
Reference actual data structures from the codebase.

### Source Code (from the real codebase)
Embed the actual source code snippets extracted in Phase 1 for this concept.
Every chapter that covers a core mechanism MUST include at least one real
source code block — not pseudocode, but the actual implementation.

Guidelines for which code to show:
- **Always show**: The primary data structure (class/struct with fields),
  the core algorithm loop, and one "surprising" helper that reveals a
  design trade-off.
- **Trim**: Remove logging, error handling, docstrings, and type imports
  that don't aid understanding. Keep the logic skeleton.
- **Annotate**: Add inline `# ←` comments on non-obvious lines. The
  reader should be able to follow the code without reading the prose.
- **Cite**: Every snippet must have a source tag:
  `# source: path/to/file.py:L42-L78`
- **Limit**: 10-40 lines per snippet. If longer, split into multiple
  snippets with prose between them explaining the transition.

For AUDIENCE=beginner, present each snippet AFTER the prose explanation
so the reader has context. For AUDIENCE=expert, lead with the code and
annotate the interesting parts.

### [Optional] Comparison / Trade-offs
If there are alternative approaches (e.g., "Why radix tree not hash map?"),
show a side-by-side comparison.

### Interactive Demo
Description of the interactive visualization for this chapter.
Specify: inputs, outputs, what animates, what stats to show.

### Key Takeaway
One callout box summarizing the essential insight.
```

### Step 2.2: Write Chapters in Parallel

Dispatch **3 parallel subagents** to write chapter content:

- **Agent 1**: Introduction + Chapters 1–N/3 (~3000-5000 words)
- **Agent 2**: Chapters N/3+1–2N/3 (~3000-5000 words)
- **Agent 3**: Chapters 2N/3+1–N + Conclusion (~3000-5000 words)

Each agent receives:
- The full concept dependency graph
- The deep-dive traces for their assigned concepts
- The chapter template
- The target AUDIENCE level

### Step 2.3: Assemble

Concatenate all agent outputs. Normalize chapter numbering. Verify:
- Every concept from the dependency graph has a chapter
- Chapters reference each other correctly (no forward references without explanation)
- Consistent terminology throughout

**Output**: `tutorial_draft.md` (~10,000-20,000 words depending on project complexity)

---

## Phase 3: Multi-Round Review

### Round 1: Technical Accuracy Review

Dispatch **2 parallel reviewer subagents** that cross-reference the draft against the actual source code:

- **Reviewer A**: Chapters 1–N/2. For each code reference and source snippet, verify:
  - Class/function names match the actual codebase
  - Data structure descriptions are accurate
  - Algorithm descriptions match the implementation
  - File paths and line numbers are correct (or close)
  - **Source code snippets are verbatim from the codebase** (not paraphrased or hallucinated). Re-read the actual file and diff against the snippet. Fix any drift.
  - Inline annotations in snippets are accurate and helpful

- **Reviewer B**: Chapters N/2+1–N. Same checks.

Produce structured feedback:
```
ERRORS: [factual mistakes with corrections]
MISSING: [important details glossed over]
STALE: [code references that point to renamed/moved code]
```

**Apply fixes**: Dispatch **2 parallel fix agents** (split by chapter range) to apply targeted edits.

**Output**: `tutorial_rev1.md`

### Round 2: Pedagogy Review

Dispatch **1 reviewer subagent** role-playing the target audience (beginner/intermediate/expert):

- Read the entire draft sequentially
- Flag every point where understanding breaks down:
  - Undefined terms used before they're introduced
  - Leaps in complexity without scaffolding
  - Missing "why should I care" motivation
  - Analogies that don't land or are confusing
- Suggest where additional diagrams or examples would help

**Apply fixes**: Dispatch **1 fix agent**.

**Output**: `tutorial_rev2.md` (final markdown)

---

## Phase 4: Interactive HTML Generation

### Step 4.1: Generate the Base HTML Shell

Write the HTML app as a single self-contained file. Architecture:

```
┌─────────────────────────────────────────────┐
│  Sidebar (chapter nav, progress tracking)   │
├─────────────────────────────────────────────┤
│  Main content area                          │
│  ┌─────────────────────────────────────┐    │
│  │  Chapter title + subtitle           │    │
│  │  Prose sections                     │    │
│  │  ┌──────────────────────────────┐   │    │
│  │  │  INTERACTIVE demo box        │   │    │
│  │  │  (animations, controls,      │   │    │
│  │  │   stats counters)            │   │    │
│  │  └──────────────────────────────┘   │    │
│  │  More prose                         │    │
│  │  Code blocks (syntax-highlighted)   │    │
│  │  Callout boxes (info/warn/success)  │    │
│  │  Comparison columns                 │    │
│  │  Chapter navigation (prev/next)     │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

**Required UI features**:
- Dark theme with CSS custom properties (easy to restyle)
- Sidebar with numbered chapter list, active/completed state
- Mobile responsive (sidebar collapses to hamburger menu)
- Smooth chapter transitions (fade + slide animation)
- Code blocks with keyword highlighting (no external deps)
- **Source code blocks** with distinct styling from pseudocode (see below)
- Comparison columns (side-by-side, good/bad styling)
- Stat counter boxes (big number + label)
- Callout boxes (info blue, success green, warning orange)
- Chapter-to-chapter navigation (prev/next buttons)

**Zero external dependencies**. Everything inline: CSS, JS, SVG. The file must work by double-clicking in a browser.

**Source code block styling**:

Real source code snippets (from the codebase) must be visually distinct from pseudocode
and explanatory code blocks. Use a dedicated `.source-code` component:

```html
<div class="source-code">
  <div class="source-header">
    <span class="source-tag">SOURCE CODE</span>
    <span class="source-path">sglang/srt/managers/scheduler.py:142-178</span>
  </div>
  <pre><code><!-- syntax-highlighted real code with inline annotations --></code></pre>
</div>
```

Styling rules:
- **Border**: Left border accent (e.g., 3px solid var(--cyan)) to distinguish from
  pseudocode blocks (which use a plain background with no accent border)
- **Header bar**: Shows `SOURCE CODE` tag + file path + line range. Monospace,
  smaller font. The file path should be visually prominent so the reader can
  find it in the actual codebase.
- **Inline annotations**: Comments starting with `# ←` or `// ←` are styled
  with a dimmer, italic font to separate them from the original code.
- **Line numbers**: Optional but recommended. Use CSS counters, not hardcoded numbers.
- **Copy button**: Top-right corner, copies the code without annotations.

This is critical because the source code is the **ground truth** — readers should
be able to tell at a glance whether they're looking at the real implementation or
a simplified explanation.

### Step 4.2: Build Interactive Demos

For each chapter, implement the interactive visualization specified in the chapter content. Every demo follows this pattern:

```html
<div class="interactive">
  <span class="label">TRY IT</span>
  <h3>Demo Title</h3>
  <p class="demo-desc">What to do and what to watch for.</p>

  <!-- Controls: buttons, sliders, text inputs -->
  <div class="controls">...</div>

  <!-- Visualization area: SVG, canvas, or styled divs -->
  <div class="viz">...</div>

  <!-- Live stats that update as the demo runs -->
  <div class="stat-row">
    <div class="stat-box"><div class="stat-value" id="...">0</div><div class="stat-label">Label</div></div>
    ...
  </div>
</div>
```

**Interactive demo design principles**:

1. **Input → Visual Feedback → Insight**. Every demo must have a clear "aha moment."
2. **Immediate response**. Animations start within 100ms of user action.
3. **Stat counters** make abstract concepts concrete (e.g., "Cache hit rate: 73%").
4. **Comparisons animate side-by-side** (e.g., with vs without an optimization).
5. **Reset buttons** let users replay. No dead-end states.
6. **Progressive complexity**: first interaction is a single button click. Advanced controls (sliders, dropdowns) are secondary.

**Common demo types to reuse**:

| Demo Type | Good For | Implementation |
|-----------|----------|---------------|
| **Timeline** | Showing parallel/serial execution, pipeline stages | Horizontal colored bars with time axis |
| **Grid/Memory** | Memory allocation, page tables, caching | CSS grid of colored cells, click to allocate/free |
| **Flow animation** | Request lifecycle, data pipelines | Components connected by arrows, highlight sequentially |
| **Tree** | Caches, routing, decision trees | SVG with nodes/edges, click to add/grow |
| **Bar chart** | Probability distributions, performance comparison | Animated CSS bars with labels |
| **Step-through** | Algorithms, state machines | "Next Step" button advancing through states |
| **Simulation** | Queues, schedulers, batch processing | Auto-running with pause/reset, live stats |
| **Calculator** | Memory budgets, performance estimates | Sliders + computed output fields |
| **Side-by-side** | Before/after, with/without optimization | Two panels animating simultaneously |
| **Tokenizer/Parser** | Text processing, compilation stages | Text input → colored/annotated output |

### Step 4.3: Polish and Test

- Verify every chapter is reachable via sidebar navigation
- Verify every interactive demo has a reset button and no broken states
- Verify no external URLs are fetched (fully offline)
- Verify mobile layout works (sidebar collapses, demos resize)
- Remove any local file paths from the output
- Add `<meta>` tags for social sharing (title, description, OG image placeholder)

**Output**: `tutorial.html` (single self-contained file, typically 2000-5000 lines)

---

## Phase 5: Optional GitHub Pages Deployment

```bash
# 1. Create repo (or use existing)
mkdir -p docs
cp tutorial.html docs/index.html

# 2. Push
git add docs/index.html
git commit -m "Publish: interactive tutorial for <project-name>"
git push

# 3. Enable GitHub Pages (if not already)
gh api repos/<owner>/<repo>/pages \
  -X POST -f source.branch=main -f source.path=/docs

# Site live at: https://<owner>.github.io/<repo>/
```

---

## Output Inventory

| File | Description |
|------|-------------|
| `architecture_survey.md` | High-level architecture map |
| `concept_traces/*.md` | Per-concept deep-dive traces |
| `tutorial_draft.md` | First complete draft |
| `tutorial_rev1.md` | After technical accuracy review |
| `tutorial_rev2.md` | After pedagogy review (final markdown) |
| `tutorial.html` | Publish-ready interactive website |

---

## Timing Estimate

| Phase | Parallelism | Wall Clock |
|-------|-------------|------------|
| 1. Investigation | 1 + 3 + 1 agents | ~12 min |
| 2. Draft writing | 3 agents + assembly | ~8 min |
| 3. Review round 1 | 2 reviewers + 2 fixers | ~8 min |
| 3. Review round 2 | 1 reviewer + 1 fixer | ~5 min |
| 4. HTML generation | shell + demos + polish | ~15 min |
| 5. Deployment | manual | ~2 min |
| **Total** | | **~50 min** |

---

## Adaptation Guide

This skill works for any project. Here's how it adapts:

### By Project Type

| Project Type | Concept Discovery Focus | Best Demo Types |
|---|---|---|
| **Web framework** | Request lifecycle, middleware, routing, ORM, auth | Flow animation, step-through, tokenizer |
| **Database** | Storage engine, query planning, indexing, transactions, WAL | Tree, memory grid, timeline, calculator |
| **Compiler/Interpreter** | Lexing, parsing, AST, type-checking, codegen, optimization | Tokenizer, tree, step-through, side-by-side |
| **ML system** | Tensors, GPU, model arch, caching, batching, scheduling | Grid, timeline, simulation, calculator |
| **Distributed system** | Consensus, replication, partitioning, failure handling | Simulation, flow animation, side-by-side |
| **CLI tool** | Parsing, plugin system, execution model, output formatting | Step-through, flow animation, tokenizer |
| **Game engine** | Game loop, ECS, rendering pipeline, physics, asset loading | Timeline, simulation, grid, step-through |
| **OS kernel** | Processes, memory, filesystem, syscalls, scheduling | Memory grid, timeline, simulation, tree |

### By Audience Level

| Level | First Principles Depth | Code References | Analogies |
|---|---|---|---|
| **Beginner** | Explain everything (what is a token? what is a GPU?) | Pseudocode only | Heavy use of everyday analogies |
| **Intermediate** | Explain domain concepts, skip CS basics | Real code with simplified pseudocode | Analogies for novel concepts only |
| **Expert** | Skip basics, focus on non-obvious design decisions | Full code references with file:line | No analogies, use precise terminology |

---

## Key Lessons

1. **Concept order is everything.** If chapter 5 uses a term from chapter 8, the reader is lost. The dependency graph in Step 1.2 is the most important artifact.

2. **One interactive demo per chapter, minimum.** Text-only chapters lose engagement. Even a simple "click to step through" animation beats a wall of prose.

3. **Never have a single agent write the entire HTML file.** It will exceed context limits. Instead: one agent writes the HTML shell + CSS + navigation JS, then per-chapter agents write the demo JS, and a final agent assembles.

4. **Interactive demos must have reset buttons.** Users WILL get the demo into a weird state. Reset is mandatory, not optional.

5. **Stats counters make abstract concepts click.** "Cache hit rate: 73%" is more memorable than "the cache frequently avoids recomputation." Always quantify.

6. **Side-by-side comparison is the most powerful demo type.** "Without X" vs "With X" running simultaneously gives an immediate intuition for why X exists.

7. **Reviewer agents that check actual source code catch real errors.** Pure-reasoning reviewers miss renamed classes, moved files, and incorrect data structure descriptions.

8. **The final HTML must be a single file with zero external dependencies.** This ensures it works offline, loads instantly, and can be hosted anywhere (GitHub Pages, S3, email attachment).

9. **Dark theme by default, light via `prefers-color-scheme`.** Developer audiences overwhelmingly prefer dark. Use CSS custom properties so the entire theme is 10 lines of overrides.

10. **Responsive layout is not optional.** Many readers will open the tutorial on a phone after someone shares the link. Sidebar must collapse. Demos must not overflow.

11. **Real source code builds trust and depth that pseudocode cannot.** Pseudocode explains the idea; source code proves it. For every core mechanism, include the actual implementation trimmed to 10-40 lines with inline annotations. Readers can then `grep` the codebase for the exact function and keep reading. Source snippets also catch the tutorial writer — if you can't find a clean 20-line snippet, you probably don't understand the mechanism well enough to explain it.

12. **Source code snippets must be visually distinct from pseudocode.** Use a different border, a `SOURCE CODE` tag, and the file path in the header. Readers must never be confused about whether they're looking at real code or a simplification. Both are valuable, but they serve different purposes: pseudocode for understanding the algorithm, source code for understanding the implementation.
