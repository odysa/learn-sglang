---
name: tutorial
description: "Generate a publish-ready interactive HTML tutorial that explains how any software project works — from first principles to implementation details. The output is a single-file interactive website with animated visualizations, suitable for GitHub Pages. Use when asked to create a tutorial, explainer, or interactive docs for a codebase."
when_to_use: "generate tutorial, create tutorial, explain this codebase, interactive docs, how does this project work, teach me this codebase"
argument-hint: "[project_path] [audience: beginner|intermediate|expert] [--design brand] [focus area]"
effort: max
---

# Interactive Project Tutorial Generator

Generate a comprehensive, publish-ready interactive HTML tutorial that explains how any software project works — from first principles to implementation details. The output is a single-file interactive website with animated visualizations.

## Parse Arguments

Extract from `$ARGUMENTS`:

- **PROJECT_PATH**: Path to codebase or GitHub URL (required). If omitted, use the current working directory.
- **AUDIENCE**: `beginner`, `intermediate` (default), or `expert`
- **DESIGN**: Optional design system brand (default: `claude`). Use `--design <brand>` to apply a different brand. Run `npx -y getdesign@latest add <brand>` to install, then read the resulting `DESIGN.md` for tokens. If the brand is `claude` or omitted, use the built-in Claude design tokens below.
- **FOCUS**: Optional specific subsystem to emphasize (e.g., "the scheduler", "the plugin system")
- **OUTPUT**: Output filename (default: `tutorial.html`)

If the project is a GitHub URL and not cloned locally, clone it to `/tmp/<repo-name>` first.

### Design System Resolution

1. If `--design claude` or no `--design` flag: use the **Built-in Claude Design Tokens** in Appendix A below.
2. If `--design <other-brand>`: run `npx -y getdesign@latest add <brand>` in `/tmp`, read the resulting `DESIGN.md`, and extract color palette, typography, spacing, shadows, border-radius, and do's/don'ts. Apply those tokens instead of the Claude defaults in Phase 4.

---

## Phase 1: Deep Codebase Investigation

### Step 1.1: Architecture Discovery

Dispatch **1 subagent** (type: `Explore`, thoroughness: "very thorough") to survey the codebase:

- Map the directory structure and identify major modules
- Read README, ARCHITECTURE.md, CONTRIBUTING.md, and any design docs
- Identify entry points (main, CLI, server startup)
- Trace the primary data flow end-to-end (e.g., request → response, input → output)
- List key abstractions: core data structures, interfaces, protocols
- Identify external dependencies and what role they play
- **Find associated papers and references**: search README, docs/, comments, and `CITATION.cff` for arXiv links, DOIs, paper titles, or blog posts. Many open-source projects are implementations of research papers — finding these is critical for the tutorial's credibility and depth. Also web-search for `"<project-name>" paper site:arxiv.org` to find papers the repo doesn't link directly.

### Step 1.2: Concept Dependency Graph

From the architecture survey, build a **concept dependency graph** — the order in which concepts must be taught so each builds on the previous:

```
Example for a web framework:
  HTTP basics → Routing → Middleware → Request/Response → ORM → Templates → Auth
Example for an ML system:
  Tensors → GPU → Model → Attention → KV Cache → Batching → Scheduling
```

This graph determines chapter order. Target: **8-16 chapters**.

### Step 1.3: Deep Dives per Concept

Dispatch **3 parallel subagents** (type: `Explore`) to investigate the codebase, split by concept groups:

- **Agent A**: Concepts 1–N/3 (foundational)
- **Agent B**: Concepts N/3+1–2N/3 (intermediate)
- **Agent C**: Concepts 2N/3+1–N (advanced)

Each agent produces for every concept:

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
For each core mechanism, extract the **actual source code** (not pseudocode):
- 10-40 lines (essential logic, not boilerplate)
- Trimmed of logging, error handling, irrelevant branches
- Annotated with inline `# ←` comments on non-obvious lines
- Tagged with exact file path and line range: `# source: path/to/file.py:L42-L78`

Include at minimum:
- The primary data structure definition (class/struct with key fields)
- The core algorithm (the "hot loop" or main dispatch logic)
- One non-obvious helper that reveals a design decision

### Key Design Decisions
Why this approach over alternatives. Trade-offs made.

### Papers & References
For each concept, find the original paper or authoritative source that introduced
the technique. Many codebase techniques originate from research papers — the
tutorial should cite them so readers can go deeper.

Search strategy:
1. Check code comments and docstrings for paper citations, arXiv links, or "Based on..."
2. Check the project's README/docs for a references section
3. Web-search: `"<technique name>" paper site:arxiv.org` (e.g., "speculative decoding paper site:arxiv.org")
4. Web-search: `"<technique name>" <project name>` for blog posts explaining the technique

For each paper found, record:
- Title, authors, year
- arXiv or DOI link
- One-sentence summary of what the paper contributes
- Which codebase files implement the paper's ideas

Example:
```
- **Speculative Decoding**: "Fast Inference from Transformers via Speculative Decoding"
  (Leviathan et al., 2023). arXiv:2211.17192. Introduced the draft-then-verify paradigm.
  Implemented in: speculative/eagle_info.py, speculative/dflash_info.py

- **RadixAttention**: "SGLang: Efficient Execution of Structured Language Model Programs"
  (Zheng et al., 2024). arXiv:2312.07104. Introduced radix tree KV cache for prefix sharing.
  Implemented in: mem_cache/radix_cache.py

- **Flash Attention**: "FlashAttention: Fast and Memory-Efficient Exact Attention"
  (Dao et al., 2022). arXiv:2205.14135. IO-aware attention algorithm.
  Implemented in: layers/attention/flashinfer_backend.py

- **Continuous Batching**: "Orca: A Distributed Serving System for Transformer-Based
  Generative Models" (Yu et al., 2022). arXiv:2206.01698. Introduced iteration-level scheduling.
  Implemented in: managers/scheduler.py
```

### Interactive Demo Idea
A specific visualization: what the user inputs/clicks, what animates, what insight it delivers.
```

### Step 1.4: Fill Gaps

Review all Phase 1 outputs and identify:
- Missing links in the concept chain
- Prerequisite knowledge the audience might lack
- Cross-cutting concerns worth a chapter

Add 1-3 supplementary concepts if needed.

---

## Phase 2: Draft Writing

### Step 2.1: Chapter Template

Every chapter follows this structure:

```markdown
## N. Chapter Title
Subtitle: one-sentence hook.

### The Problem
What real-world problem does this concept solve?
Start with the pain — what breaks, what's slow, what's wasteful without this?

### The Idea (Simplified)
Core idea in 2-3 paragraphs.
Use analogies if AUDIENCE=beginner.

### [If applicable] What Is <Technique>?
When a chapter introduces a non-obvious data structure, algorithm, or technique
(radix tree, ring buffer, FSM, speculative execution, etc.), explain IT before
explaining how the codebase uses it. The reader needs to understand the tool
before seeing it applied.

Structure as:
1. What is it in general? (1-2 paragraphs, CS-agnostic explanation)
2. Why is it the right fit here? (compare to alternatives the reader might expect)
3. Then transition into how the codebase implements it.

Example for RadixAttention chapter:
  - First explain: "A radix tree (trie) is a compressed prefix tree where..."
  - Then explain: "Why not a hash table? Because hash lookup is O(N) per prefix
    length — you must hash the entire sequence to check if it's cached. A radix
    tree matches incrementally: once you've matched N tokens, checking token N+1
    is O(1)."
  - Then: "Here's how SGLang's RadixCache implements this..."

### Why This, Not That?
Every chapter should answer: "Why did the authors choose THIS approach over
the obvious alternative?" This is often the most valuable part of a tutorial
because it builds engineering intuition, not just knowledge.

Structure as a comparison:
  | Approach | Pros | Cons | When to use |
  | The chosen approach | ... | ... | (this codebase) |
  | The obvious alternative | ... | ... | (when X is different) |

Examples of "Why this, not that?" questions worth answering:
  - Why radix tree, not hash table for prefix caching?
  - Why separate processes via ZMQ, not async tasks in one process?
  - Why ring shadows instead of drop shadows? (design)
  - Why chunked prefill instead of just priority queuing?
  - Why CUDA graph replay instead of torch.compile?
  - Why FSM-based constrained decoding instead of retry-on-invalid?

### How It Works (Detailed)
Step-by-step technical walkthrough.
Include pseudocode showing key algorithms.
Reference actual data structures from the codebase.

### Source Code (from the real codebase)
Embed actual source code snippets from Phase 1.
- Present AFTER prose for beginners (context first)
- Lead WITH code for experts (annotate interesting parts)
- Trim to 10-40 lines per snippet
- Always include `# source:` tag with file path

### Further Reading
Link to the original papers and references that introduced the techniques
used in this chapter. Not every chapter needs this — only chapters where the
technique has a clear research origin (most do in systems/ML codebases).

Format as a compact list:
- Paper title (Authors, Year) — one-line summary. [arXiv link]

### Interactive Demo
Description of the interactive visualization.

### Key Takeaway
One callout box summarizing the essential insight.
```

### Step 2.2: Write Chapters in Parallel

Dispatch **3 parallel subagents** to write chapter content:

- **Agent 1**: Introduction + Chapters 1–N/3 (~3000-5000 words)
- **Agent 2**: Chapters N/3+1–2N/3 (~3000-5000 words)
- **Agent 3**: Chapters 2N/3+1–N + Conclusion (~3000-5000 words)

Each agent receives the full concept dependency graph, the deep-dive traces for their concepts, the chapter template, and the target AUDIENCE level.

---

## Phase 3: Review

### Round 1: Technical Accuracy

Dispatch **2 parallel subagents** to cross-reference the draft against actual source code:
- Verify class/function names match the codebase
- Verify data structure descriptions are accurate
- Verify source code snippets are verbatim (re-read actual file and diff)
- Fix any drift, stale references, or renamed code

### Round 2: Pedagogy

Review the assembled draft sequentially:
- Flag undefined terms used before introduction
- Flag leaps in complexity without scaffolding
- Ensure "why should I care" motivation exists per chapter
- This review can be done inline (no need for a separate agent if the draft is solid)

---

## Phase 4: Interactive HTML Generation

### Step 4.1: HTML Architecture

Write a **single self-contained HTML file** with zero external dependencies. Must work by double-clicking in a browser.

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
│  │  Source code blocks (highlighted)   │    │
│  │  Callout boxes (info/warn/success)  │    │
│  │  Chapter navigation (prev/next)     │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

**Required UI features**:
- Sidebar with numbered chapter list, active/completed state, progress bar
- Mobile responsive (sidebar collapses to hamburger menu)
- Smooth chapter transitions (fade + slide animation)
- Code blocks with keyword highlighting (inline, no external deps)
- Chapter-to-chapter navigation (prev/next buttons)
- Light theme by default with dark via `prefers-color-scheme: dark`
- All styling via CSS custom properties so the design system is a single block of overrides

**Apply the resolved design system** from the Parse Arguments step. Use the design tokens for:
- CSS custom properties (colors, surfaces, text, borders)
- Typography (headline font family, body font family, weights, line-heights)
- Border radius scale (from the design system's component stylings)
- Shadow/elevation system (ring shadows vs drop shadows)
- Button styles, callout colors, interactive element styling
- Do's and Don'ts as hard rules

**Source code block styling** — must be visually distinct from pseudocode:

```html
<div class="source-code">
  <div class="source-header">
    <span class="source-tag">SOURCE CODE</span>
    <span class="source-path">path/to/file.py:L42-L78</span>
  </div>
  <pre><code><!-- syntax-highlighted real code --></code></pre>
</div>
```

- Left border accent (3px solid, using the design system's brand/accent color)
- Header bar with `SOURCE CODE` tag + file path, on dark surface
- Inline annotations (`# ←`) in dimmer italic font, using the accent color
- Code blocks always use a dark surface background regardless of page theme

**Callout boxes**: use the design system's accent for info, success color for success, accent for warning

**"What Is X?" explanation boxes** — when a chapter introduces a non-obvious technique:

```html
<div class="callout callout-info">
  <strong>What is a radix tree?</strong> A radix tree (also called a compressed
  trie) stores sequences by sharing common prefixes. Each node holds a segment
  of the sequence, not a single character...
</div>
```

Style these with the info callout color and a bold lead question. Place them BEFORE the "How It Works" section so the reader has the prerequisite knowledge.

**"Why This, Not That?" comparison tables** — for design decision explanations:

```html
<table class="compare-table">
  <thead><tr><th>Approach</th><th>Lookup Cost</th><th>Prefix Sharing</th><th>Verdict</th></tr></thead>
  <tbody>
    <tr class="chosen"><td>Radix tree</td><td>O(1) incremental</td><td>Natural</td><td>Chosen</td></tr>
    <tr><td>Hash table</td><td>O(N) per lookup</td><td>None</td><td>Too slow for long prefixes</td></tr>
  </tbody>
</table>
```

Style `.compare-table` with the design system's card surface, border, and radius. Highlight the `.chosen` row with a subtle accent background. These tables are high-value — they build engineering intuition, not just knowledge.

**"Further Reading" paper references** — link to the original research for each technique:

```html
<div class="further-reading">
  <h4>Further Reading</h4>
  <ul>
    <li>
      <a href="https://arxiv.org/abs/2211.17192" target="_blank" rel="noopener">
        Fast Inference from Transformers via Speculative Decoding
      </a>
      <span class="paper-meta">Leviathan et al., 2023</span>
      — Introduced the draft-then-verify paradigm that SGLang implements in its speculative module.
    </li>
  </ul>
</div>
```

Style `.further-reading` as a subtle card at the bottom of each chapter (before the key takeaway callout). Use the design system's secondary surface with a border. Paper links use the accent color. `.paper-meta` is dimmer/smaller text for authors and year. This section is the bridge between "how the code works" and "why the field works this way."

### Step 4.2: Build Interactive Demos

For each chapter, implement the interactive visualization. Every demo follows this pattern:

```html
<div class="interactive">
  <span class="label">TRY IT</span>
  <h4>Demo Title</h4>
  <p class="demo-desc">What to do and what to watch for.</p>
  <div class="controls"><!-- buttons, sliders --></div>
  <div class="viz"><!-- visualization area --></div>
  <div class="stat-row">
    <div class="stat-box"><div class="stat-value">0</div><div class="stat-label">Label</div></div>
  </div>
</div>
```

**Demo design principles**:
1. **Input → Visual Feedback → Insight**. Every demo must have a clear "aha moment."
2. **Immediate response**. Animations start within 100ms of user action.
3. **Stat counters** make abstract concepts concrete (e.g., "Cache hit rate: 73%").
4. **Reset buttons** let users replay. No dead-end states.
5. **Progressive complexity**: first interaction is a single button click.

**Common demo types**:

| Demo Type | Good For | Implementation |
|-----------|----------|---------------|
| **Timeline** | Parallel/serial execution, pipeline stages | Horizontal colored bars |
| **Grid/Memory** | Memory allocation, caching | CSS grid of colored cells |
| **Flow animation** | Request lifecycle, data pipelines | Components + arrows |
| **Tree** | Caches, routing, decision trees | Nested divs or SVG |
| **Bar chart** | Probability distributions, comparisons | Animated CSS bars |
| **Step-through** | Algorithms, state machines | "Next Step" button |
| **Simulation** | Queues, schedulers, batching | Auto-running with stats |
| **Calculator** | Memory budgets, performance | Sliders + computed output |
| **Side-by-side** | Before/after, with/without optimization | Two panels animating |
| **Tokenizer** | Text processing | Text input → colored output |

### Step 4.3: Polish

- Verify every chapter is reachable via sidebar navigation
- Verify every demo has a reset button and no broken states
- Verify no external URLs are fetched (fully offline)
- Verify mobile layout works (sidebar collapses, demos resize)
- Add `<meta>` tags for social sharing (title, description)

**Output**: Write to `$OUTPUT` (default: `tutorial.html`)

---

## Key Lessons

1. **Concept order is everything.** If chapter 5 uses a term from chapter 8, the reader is lost. The dependency graph in Step 1.2 is the most important artifact.

2. **One interactive demo per chapter, minimum.** Text-only chapters lose engagement.

3. **Never have a single agent write the entire HTML file.** It will exceed context limits. Write prose chapters first, then generate the HTML shell + assemble.

4. **Interactive demos must have reset buttons.** Users WILL get the demo into a weird state.

5. **Stats counters make abstract concepts click.** "Cache hit rate: 73%" beats "the cache frequently avoids recomputation."

6. **Side-by-side comparison is the most powerful demo type.** "Without X" vs "With X" simultaneously gives immediate intuition.

7. **Real source code builds trust that pseudocode cannot.** For every core mechanism, include the actual implementation trimmed to 10-40 lines with inline annotations. Source snippets also catch the writer — if you can't find a clean snippet, you don't understand the mechanism well enough.

8. **Source code snippets must be visually distinct from pseudocode.** Use a different border, a `SOURCE CODE` tag, and the file path in the header.

9. **The final HTML must be a single file with zero external dependencies.** Works offline, loads instantly, hostable anywhere.

10. **Apply a design system, don't invent one.** Use `--design <brand>` to pull tokens from getdesign.md (68+ brands available). The Claude design is the built-in default. All colors, typography, shadows, and border-radius come from the design system — CSS custom properties make swapping trivial.

11. **Responsive layout is not optional.** Many readers open on mobile after someone shares the link.

12. **Explain the technique, not just its usage.** If a chapter introduces a radix tree, explain what a radix tree IS before explaining how the codebase uses one. If a chapter uses speculative decoding, explain why memory-bandwidth-bound inference makes it work. The reader who doesn't know the underlying concept will bounce — the reader who already knows will skim past the explanation in 5 seconds. Always err on the side of explaining.

13. **Every design decision deserves a "why not the alternative?"** The most valuable insight in any tutorial is not "the system uses X" but "the system uses X instead of Y because of Z." This is the difference between documentation (what) and education (why). Dedicate a comparison table or paragraph to the rejected alternative in every chapter where a non-obvious choice was made.

---

## Adaptation by Project Type

| Project Type | Concept Focus | Best Demo Types |
|---|---|---|
| **Web framework** | Request lifecycle, middleware, routing, ORM | Flow animation, step-through |
| **Database** | Storage engine, query planning, indexing, transactions | Tree, memory grid, timeline |
| **Compiler** | Lexing, parsing, AST, type-checking, codegen | Tokenizer, tree, step-through |
| **ML system** | Tensors, GPU, model arch, caching, batching | Grid, timeline, simulation |
| **Distributed system** | Consensus, replication, partitioning | Simulation, flow animation |
| **CLI tool** | Parsing, plugins, execution model | Step-through, flow animation |
| **OS kernel** | Processes, memory, filesystem, scheduling | Memory grid, timeline |

## Adaptation by Audience Level

| Level | First Principles | Code References | Analogies |
|---|---|---|---|
| **Beginner** | Explain everything | Pseudocode only | Heavy use |
| **Intermediate** | Explain domain concepts, skip CS basics | Real code + pseudocode | For novel concepts only |
| **Expert** | Skip basics, focus on design decisions | Full code with file:line | None, use precise terminology |

---

## Appendix A: Built-in Claude Design Tokens (Default)

When `--design claude` or no design flag is specified, use these tokens. This is the Claude (Anthropic) design system — warm, editorial, literary.

### Atmosphere
Warm parchment canvas evoking premium paper. Serif headlines for authority, sans for utility. Terracotta brand accent — earthy, deliberately un-tech. Exclusively warm-toned neutrals (every gray has a yellow-brown undertone). Ring-based shadow system. Magazine-like pacing.

### CSS Custom Properties (Light Theme)
```css
:root {
  /* Surfaces */
  --bg: #f5f4ed;               /* Parchment — primary canvas */
  --bg-secondary: #faf9f5;     /* Ivory — cards, sidebar */
  --bg-tertiary: #e8e6dc;      /* Warm Sand — interactive surfaces */
  --code-bg: #141413;          /* Near Black — code blocks (always dark) */

  /* Text */
  --text: #4d4c48;             /* Charcoal Warm — primary body */
  --text-secondary: #5e5d59;   /* Olive Gray — secondary */
  --text-tertiary: #87867f;    /* Stone Gray — tertiary, metadata */
  --text-bright: #141413;      /* Near Black — headlines */

  /* Brand & Accent */
  --accent: #c96442;           /* Terracotta Brand — CTAs, brand moments */
  --accent-light: #d97757;     /* Coral — links, hover states */
  --green: #5a7a52;            /* Muted sage — success states */
  --red: #b53333;              /* Error Crimson — errors */
  --purple: #7a5c8a;           /* Muted purple — code functions */

  /* Borders & Ring Shadows */
  --border: #f0eee6;           /* Border Cream — gentlest containment */
  --border-strong: #e8e6dc;    /* Border Warm — section dividers */
  --ring: #d1cfc5;             /* Ring Warm — button hover/focus */
  --ring-deep: #c2c0b6;        /* Ring Deep — active/pressed states */
}
```

### CSS Custom Properties (Dark Theme — via prefers-color-scheme)
```css
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #141413; --bg-secondary: #1e1e1c; --bg-tertiary: #30302e;
    --code-bg: #0e0e0d;
    --text: #b0aea5; --text-secondary: #87867f; --text-tertiary: #5e5d59;
    --text-bright: #faf9f5;
    --accent: #d97757; --accent-light: #e08a6a;
    --green: #7a9a72; --red: #d46464; --purple: #c9a0dc;
    --border: #30302e; --border-strong: #3d3d3a;
    --ring: #4d4c48; --ring-deep: #5e5d59;
  }
}
```

### Typography
| Role | Font | Size | Weight | Line Height |
|------|------|------|--------|-------------|
| Page title | Georgia, 'Times New Roman', serif | 32px | 500 | 1.20 |
| Chapter subtitle | Georgia, serif | 16px | 400 italic | 1.50 |
| Section heading | Georgia, serif | 22px | 500 | 1.25 |
| Body text | Inter, system-ui, sans-serif | 16px | 400 | 1.65 |
| Sidebar logo | Georgia, serif | 20px | 500 | - |
| Demo heading | Georgia, serif | 18px | 500 | - |
| Stat value | Georgia, serif | 26px | 500 | - |
| Code | SF Mono, Fira Code, monospace | 13px | 400 | 1.65 |
| Labels/meta | sans-serif | 11px | 400 | - |

**Rules**: Serif for ALL headlines (single weight 500, never bold). Sans for ALL UI text. Body line-height 1.65 (generous, book-like). Heading line-heights tight (1.20-1.25).

### Syntax Highlighting (Warm Tones)
```css
.kw { color: #d97757; }      /* keywords — coral */
.fn { color: #c9a0dc; }      /* functions — muted lavender */
.st { color: #a8c4a0; }      /* strings — sage green */
.cm { color: #87867f; }      /* comments — stone gray, italic */
.an { color: #c96442; }      /* annotations — terracotta, italic */
.num { color: #d4a574; }     /* numbers — warm tan */
.dec { color: #c96442; }     /* decorators — terracotta */
```

### Border Radius Scale
| Context | Radius |
|---------|--------|
| Inline code | 6px |
| Buttons | 8px |
| Cards, containers | 12px |
| Code blocks | 12px |
| Interactive demos | 16px |
| Featured containers | 16-24px |

### Shadow System (Ring-Based)
```css
/* Level 1 — Buttons, inline code */
box-shadow: 0px 0px 0px 1px var(--ring);

/* Level 2 — Hover/focus */
box-shadow: 0px 0px 0px 1px var(--ring-deep);

/* Level 3 — Elevated cards, code blocks */
box-shadow: rgba(0,0,0,0.06) 0px 4px 24px;

/* Level 4 — Lit/active pipeline stages */
box-shadow: 0px 0px 0px 2px rgba(201,100,66,0.25), rgba(201,100,66,0.08) 0px 4px 16px;
```

**Rule**: Use ring shadows (`0px 0px 0px 1px`) for interactive elements. Use whisper shadows (`rgba(0,0,0,0.06) 0px 4px 24px`) for elevated content. Never use heavy drop shadows.

### Comparison Table Styling
```css
.compare-table {
  width: 100%; border-collapse: collapse; margin: 20px 0;
  font-size: 14px; border-radius: 12px; overflow: hidden;
  border: 1px solid var(--border-strong);
}
.compare-table th {
  background: var(--bg-tertiary); padding: 10px 16px; text-align: left;
  font-weight: 600; color: var(--text-bright); font-size: 13px;
}
.compare-table td { padding: 10px 16px; border-top: 1px solid var(--border); }
.compare-table tr.chosen { background: rgba(201,100,66,0.06); }
.compare-table tr.chosen td:last-child { color: var(--accent); font-weight: 600; }

/* Further Reading — paper references */
.further-reading {
  background: var(--bg-secondary); border: 1px solid var(--border-strong);
  border-radius: 12px; padding: 20px 24px; margin: 28px 0;
}
.further-reading h4 {
  font-family: Georgia, serif; font-weight: 500; font-size: 16px;
  margin-bottom: 12px; color: var(--text-bright);
}
.further-reading ul { list-style: none; padding: 0; }
.further-reading li { margin-bottom: 10px; line-height: 1.55; font-size: 14px; }
.further-reading a { color: var(--accent); text-decoration: none; font-weight: 500; }
.further-reading a:hover { text-decoration: underline; }
.paper-meta { color: var(--text-tertiary); font-size: 12px; margin-left: 4px; }
```

### Do's and Don'ts
**Do**:
- Use Parchment (`#f5f4ed`) as primary background — the warm cream IS the personality
- Use Georgia/serif at weight 500 for all headlines
- Use Terracotta (`#c96442`) only for primary CTAs and highest-signal brand moments
- Keep all neutrals warm-toned — every gray has a yellow-brown undertone
- Use ring shadows for interactive states instead of drop shadows
- Maintain serif/sans hierarchy — serif for content headlines, sans for UI
- Use generous body line-height (1.65) for a literary reading experience

**Don't**:
- Don't use cool blue-grays anywhere — palette is exclusively warm
- Don't use bold (700+) on serif — weight 500 is the ceiling
- Don't introduce saturated colors beyond Terracotta — palette is deliberately muted
- Don't use sharp corners (< 6px) on buttons or cards
- Don't apply heavy drop shadows
- Don't use pure white (`#fff`) as page background — Parchment or Ivory always
- Don't reduce body line-height below 1.40

### Memory Grid Demo Colors (Warm Palette)
```javascript
const colors = ['#c96442','#5a7a52','#d97757','#7a5c8a','#b53333','#d4a574','#a8c4a0','#c9a0dc'];
```

### Available Design Brands (via getdesign.md)
To use a different design: `/tutorial myproject intermediate --design stripe`

**AI/LLM**: claude, cohere, elevenlabs, minimax, mistral.ai, ollama, opencode.ai, replicate, runwayml, together.ai, x.ai
**Dev Tools**: cursor, expo, lovable, raycast, superhuman, vercel, warp
**Backend/DB**: clickhouse, composio, hashicorp, mongodb, posthog, sanity, sentry, supabase
**SaaS**: cal, intercom, linear.app, mintlify, notion, resend, zapier
**Design**: airtable, clay, figma, framer, miro, webflow
**Fintech**: binance, coinbase, kraken, revolut, stripe, wise
**E-commerce**: airbnb, meta, nike, shopify
**Consumer**: apple, ibm, nvidia, pinterest, playstation, spacex, spotify, uber
**Auto**: bmw, bugatti, ferrari, lamborghini, renault, tesla
