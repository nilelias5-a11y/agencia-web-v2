# Model Routing Analysis — Café Vermut Project

**Session date:** 2026-05-13
**Project:** Café Vermut
**Skill applied:** token-saver (Model Routing Rules + Session Metrics Report template)

---

## Per-Task Model Recommendations

### (a) Create the complete design system from scratch — component library, tokens, dark mode, responsive rules

**Recommended model:** Claude Opus

**Justification:** This is a high-complexity creative and architectural task requiring deep reasoning: designing a coherent token system, anticipating dark-mode contrast ratios, defining responsive breakpoint logic, and structuring a component library that will be reused across the entire project. The output must be internally consistent and forward-compatible — errors here cascade to every other task. Per the Model Routing Rules, "brand analysis, creative direction, complex strategy → Claude Opus."

**Estimated token consumption:**
- Input: ~1,500 tokens (project brief, brand palette, design constraints)
- Output: ~6,000 tokens (token definitions, component inventory, responsive rules, dark mode mappings)
- **Subtotal: ~7,500 tokens**

---

### (b) Generate alt text for 47 portfolio images (each image is a coffee product photo)

**Recommended model:** Claude Haiku

**Justification:** Highly repetitive, low-complexity formatting task. Each alt text follows a predictable pattern: [product name] + [visual descriptor] + [context]. No strategic reasoning required. Per the Model Routing Rules, "repetitive formatting, translation, simple summaries → Claude Haiku." Significant cost savings vs. Sonnet or Opus for 47 near-identical calls.

**Optimization note:** Batch all 47 images in a single structured prompt (one call with a list of image descriptions) rather than 47 individual calls. Use Technique 5 (Chunked Delivery) — pass only image metadata, not full product records.

**Estimated token consumption:**
- Input: ~2,350 tokens (~50 tokens per image × 47)
- Output: ~1,410 tokens (~30 tokens per alt text × 47)
- **Subtotal: ~3,760 tokens**

---

### (c) Write 3 CTA button variants for the hero ("Buy coffee" alternatives)

**Recommended model:** Claude Haiku

**Justification:** Three short copy variants is a minimal creative task — 3 to 8 words each. No structural reasoning, no codebase analysis, no strategy required. Haiku handles short-form copy generation reliably. Per the Model Routing Rules, "simple summaries → Claude Haiku." Using Sonnet or Opus here would be a direct waste of budget.

**Estimated token consumption:**
- Input: ~200 tokens (brand tone, hero context, existing headline)
- Output: ~80 tokens (3 variants with brief rationale)
- **Subtotal: ~280 tokens**

---

### (d) Audit the accessibility of 500 lines of finished JSX/TSX code

**Recommended model:** Claude Sonnet

**Justification:** Code auditing requires technical reading comprehension, pattern recognition (ARIA roles, semantic HTML, contrast, keyboard navigation), and structured output of findings. This is not creative strategy (no Opus needed) but not trivial formatting either. Per the Model Routing Rules, "code generation, SEO, content writing → Claude Sonnet." Sonnet handles 500 lines of code efficiently with high accuracy.

**Optimization note:** Apply Technique 3 (Code Context Summarization) — pass only the JSX/TSX code, not surrounding project context. No brand brief needed.

**Estimated token consumption:**
- Input: ~6,500 tokens (500 lines of JSX/TSX + audit instruction prompt)
- Output: ~1,200 tokens (structured findings, WCAG references, fix recommendations)
- **Subtotal: ~7,700 tokens**

---

### (e) Write the "Our Story" page (500 words, artisanal voice)

**Recommended model:** Claude Sonnet

**Justification:** Mid-complexity content writing task. The "artisanal voice" requires tonal consistency and narrative skill, but this is a well-defined creative writing task — not brand strategy or architectural decisions. Per the Model Routing Rules, "content writing → Claude Sonnet." Opus would be overkill; Haiku would likely produce generic output that doesn't hold the artisanal tone at 500 words.

**Estimated token consumption:**
- Input: ~400 tokens (brand brief, tone guidelines, key story beats)
- Output: ~700 tokens (~500 words output)
- **Subtotal: ~1,100 tokens**

---

### (f) Format metadata (name, description, price, weight) for 30 products in the catalog

**Recommended model:** Claude Haiku

**Justification:** Pure repetitive formatting task. Each product follows an identical schema: name, description, price, weight. No creative or strategic judgment needed — just structured transformation of raw data into a consistent format. Per the Model Routing Rules, "repetitive formatting → Claude Haiku."

**Optimization note:** Use Technique 5 (Chunked Delivery) — send all 30 products in one batched prompt with a clear schema template. Do not send brand context or design system details.

**Estimated token consumption:**
- Input: ~2,100 tokens (~70 tokens per product × 30, including schema instruction)
- Output: ~1,800 tokens (~60 tokens formatted output per product × 30)
- **Subtotal: ~3,900 tokens**

---

### (g) Review whether CSS breakpoints are correctly applied across 8 components

**Recommended model:** Claude Sonnet

**Justification:** Technical code review requiring understanding of responsive design patterns, breakpoint semantics (mobile-first vs. desktop-first), and cross-component consistency. More analytical than task (f) but less complex than the full design system in (a). Per the Model Routing Rules, "code generation [and review] → Claude Sonnet."

**Optimization note:** Apply Technique 3 — pass only the CSS/Tailwind classes and breakpoint declarations for each component, not full JSX. If the design system tokens from task (a) are cached, pass only the breakpoint token reference (cache key: `cafe-vermut-tokens`), not the full design system output.

**Estimated token consumption:**
- Input: ~3,200 tokens (8 component CSS snippets + breakpoint reference)
- Output: ~600 tokens (per-component verdict + correction list)
- **Subtotal: ~3,800 tokens**

---

### (h) Decide whether to use Next.js 16 App Router vs Remix for this project

**Recommended model:** Claude Opus

**Justification:** This is a high-stakes architectural decision with long-term consequences. It requires reasoning across multiple dimensions: SSR/SSG tradeoffs, team familiarity, ecosystem maturity, hosting options, performance benchmarks, and project-specific requirements (catalog size, SEO needs, real-time features). Per the Model Routing Rules, "complex strategy → Claude Opus." Getting this wrong affects every downstream task.

**Estimated token consumption:**
- Input: ~800 tokens (project requirements, team context, current stack, constraints)
- Output: ~1,500 tokens (structured comparison, recommendation with rationale, risk flags)
- **Subtotal: ~2,300 tokens**

---

## Token Usage Report

**Session date:** 2026-05-13
**Project:** Café Vermut

### Consumption by Task

| Task | Description | Model | Input tokens | Output tokens | Subtotal |
|------|-------------|-------|-------------|--------------|----------|
| (a) | Design system from scratch | Opus | 1,500 | 6,000 | 7,500 |
| (b) | Alt text for 47 images | Haiku | 2,350 | 1,410 | 3,760 |
| (c) | 3 CTA button variants | Haiku | 200 | 80 | 280 |
| (d) | Accessibility audit, 500 lines JSX | Sonnet | 6,500 | 1,200 | 7,700 |
| (e) | "Our Story" page, 500 words | Sonnet | 400 | 700 | 1,100 |
| (f) | Metadata formatting, 30 products | Haiku | 2,100 | 1,800 | 3,900 |
| (g) | CSS breakpoints review, 8 components | Sonnet | 3,200 | 600 | 3,800 |
| (h) | Next.js vs Remix decision | Opus | 800 | 1,500 | 2,300 |

**Total input tokens:** 17,050
**Total output tokens:** 13,290
**Total tokens:** 30,340

### Estimated Cost (approximate, May 2026 pricing)

| Model | Tokens used | Approx. cost |
|-------|------------|--------------|
| Claude Opus (tasks a, h) | 9,800 total | ~$0.49 |
| Claude Sonnet (tasks d, e, g) | 12,600 total | ~$0.19 |
| Claude Haiku (tasks b, c, f) | 7,940 total | ~$0.03 |
| **Total** | **30,340** | **~$0.71** |

> Pricing basis: Opus ~$15/M input + $75/M output, Sonnet ~$3/M input + $15/M output, Haiku ~$0.25/M input + $1.25/M output (estimated mid-2026 rates). Actual cost depends on exact API tier and any prompt caching discounts.

---

### Efficiency Actions Taken

- **Model downgrades applied: 3**
  - Tasks (b), (c), (f) routed to Haiku instead of Sonnet — saved ~$0.14 vs. routing all to Sonnet
  - Task (c) especially high-efficiency: 280 tokens for 3 CTAs vs. ~280 tokens on any model, but Haiku costs ~95% less than Opus
- **Batching optimizations: 2**
  - Task (b): 47 alt texts in 1 call instead of 47 calls (saves per-call overhead)
  - Task (f): 30 products in 1 call with structured schema
- **Context stripping applied: 2**
  - Task (d): brand brief excluded — audit only needs code
  - Task (g): full design system not passed, only breakpoint tokens (cache key: `cafe-vermut-tokens`)
- **Cache opportunities identified: 1**
  - Design system output from task (a) should be cached as `cafe-vermut-tokens` for reuse in task (g) and any future responsive work

### Recommendations for Next Session

- Cache task (a) output under key `cafe-vermut-tokens` — reusable for every future CSS/component review, animation, and responsive audit in this project
- If task (b) images have a consistent naming convention, a single Haiku call with a structured image list is more efficient than individual calls — confirm image manifest format with frontend-developer before running
- Task (h) architectural decision output should be cached as `cafe-vermut-stack` — any future software recommendation or integration task can reference this without re-running the analysis
- Tasks (d) and (g) can be combined in a future session if both run on the same component — single Sonnet call with dual instructions saves ~1,500 input tokens
