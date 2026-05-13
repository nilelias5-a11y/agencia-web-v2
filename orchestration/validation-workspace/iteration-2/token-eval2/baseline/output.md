# AI Model Recommendations — Café Vermut Web Project

**Date:** 2026-05-13
**Project:** Café Vermut
**Evaluator:** Claude Sonnet 4.6

---

## Pricing Reference (as of mid-2025)

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|---|---|---|
| Claude Opus 4 | $15.00 | $75.00 |
| Claude Sonnet 4.5 | $3.00 | $15.00 |
| Claude Haiku 3.5 | $0.80 | $4.00 |

---

## Task-by-Task Recommendations

---

### (a) Create the complete design system from scratch
**— component library, tokens, dark mode, responsive rules**

**Recommended model: Claude Opus 4**

**Justification:**
This is the highest-complexity task in the session. It requires:
- Architectural decisions that cascade across the entire project lifetime
- Simultaneous management of token schemas, component APIs, theming logic (light/dark), and responsive breakpoint strategy
- Deep reasoning about naming conventions, scalability, and inter-component dependencies
- Output that will be consumed by every other developer task downstream

Errors here compound into all future work. The cost premium is justified by the strategic nature of the output and the long-term savings from getting it right on the first pass. Sonnet would produce a workable design system but risks shallow token hierarchies and brittle dark mode logic.

**Token estimate:**
- Input: ~2,000 tokens (project brief, brand guidelines, tech stack context)
- Output: ~6,000 tokens (token file, component structure doc, theming logic, breakpoint map)
- **Subtotal: ~8,000 tokens**
- **Cost: (2K × $0.015) + (6K × $0.075) ≈ $0.03 + $0.45 = ~$0.48**

---

### (b) Generate alt text for 47 portfolio images
**— each image is a coffee product photo**

**Recommended model: Claude Haiku 3.5**

**Justification:**
Alt text for product images is a high-volume, low-complexity, highly repetitive task. Each call follows the same pattern: describe the product visually in 10–20 words, include key SEO terms (coffee type, color, texture, presentation). There is no architectural reasoning or creative originality required. Haiku handles this class of structured extraction/generation at high accuracy and near-zero cost. Using Sonnet or Opus here would waste budget with zero quality gain.

**Token estimate:**
- Per image: ~50 tokens input (image descriptor or filename + product name) + ~30 tokens output (alt text string)
- 47 images: 47 × 80 tokens = ~3,760 tokens total
- **Subtotal: ~3,800 tokens**
- **Cost: (1.9K × $0.0008) + (1.9K × $0.004) ≈ $0.0015 + $0.0076 = ~$0.009 (under $0.01)**

---

### (c) Write 3 CTA button variants for the hero
**— "Buy coffee" alternatives**

**Recommended model: Claude Haiku 3.5**

**Justification:**
Three short-copy variants for a single CTA button is a micro-copywriting task. The output is fewer than 30 words total. Even with strong brand voice requirements, Haiku produces excellent short-form marketing copy when given a clear brand brief. No multi-step reasoning, no technical complexity. Using Sonnet or Opus would be like hiring a senior architect to write a Post-it note.

**Token estimate:**
- Input: ~200 tokens (brand voice guide, hero context, examples to avoid)
- Output: ~60 tokens (3 variants × ~20 tokens each)
- **Subtotal: ~260 tokens**
- **Cost: (200 × $0.0008) + (60 × $0.004) ≈ $0.00016 + $0.00024 = ~$0.0004 (negligible)**

---

### (d) Audit the accessibility of 500 lines of finished JSX/TSX code

**Recommended model: Claude Sonnet 4.5**

**Justification:**
Accessibility auditing requires genuine technical reasoning: identifying missing ARIA roles, checking semantic HTML correctness, evaluating keyboard navigation flows, catching contrast issues in className strings, and flagging interactive element labeling gaps. This is not mechanical extraction — it demands understanding of WCAG 2.1 AA criteria and how they apply to specific component patterns. However, it does not require the deep architectural creativity of Opus. Sonnet has sufficient technical depth for a thorough WCAG audit at a fraction of Opus cost. Haiku would miss subtle issues (e.g., nested interactive elements, focus trap logic errors).

**Token estimate:**
- Input: ~6,000 tokens (500 lines of JSX/TSX at ~12 tokens/line average)
- Output: ~1,500 tokens (structured audit report: issues list, severity ratings, fix suggestions)
- **Subtotal: ~7,500 tokens**
- **Cost: (6K × $0.003) + (1.5K × $0.015) ≈ $0.018 + $0.0225 = ~$0.04**

---

### (e) Write the "Our Story" page
**— 500 words, artisanal voice**

**Recommended model: Claude Sonnet 4.5**

**Justification:**
A 500-word brand narrative requires genuine creative writing quality: tone consistency, emotional arc, authentic voice, and the ability to balance storytelling with conversion intent. Sonnet delivers high-quality long-form copy and handles nuanced brand voice well. Opus is justified only when the creative task has extremely high strategic stakes (e.g., a manifesto that defines the brand identity from zero). For a single interior page of a café website, Sonnet is the sweet spot between quality and cost. Haiku would produce generic, flat prose that fails to capture the artisanal character.

**Token estimate:**
- Input: ~500 tokens (brand brief, tone guide, key story beats, founder info)
- Output: ~700 tokens (500-word page copy + minor structural notes)
- **Subtotal: ~1,200 tokens**
- **Cost: (500 × $0.003) + (700 × $0.015) ≈ $0.0015 + $0.0105 = ~$0.012**

---

### (f) Format metadata for 30 products in the catalog
**— name, description, price, weight**

**Recommended model: Claude Haiku 3.5**

**Justification:**
Catalog metadata formatting is a structured, rule-based transformation task. Given a schema (field names, character limits, formatting conventions) and raw product data, Haiku reliably produces clean, consistent output across all 30 records. The task is analogous to batch ETL work: high volume, low variance, deterministic output. No creative originality or deep reasoning required. Haiku is the correct and most cost-efficient choice.

**Token estimate:**
- Per product: ~80 tokens input (raw data) + ~60 tokens output (formatted metadata object)
- 30 products: 30 × 140 tokens = ~4,200 tokens
- Plus schema/instructions: ~300 tokens input
- **Subtotal: ~4,500 tokens**
- **Cost: (2.55K × $0.0008) + (1.95K × $0.004) ≈ $0.002 + $0.0078 = ~$0.01**

---

### (g) Review whether CSS breakpoints are correctly applied across 8 components

**Recommended model: Claude Haiku 3.5**

**Justification:**
Breakpoint verification is a checklist-style code review task. Given the breakpoint definitions (e.g., `sm: 640px`, `md: 768px`, `lg: 1024px`) and 8 component files, the task is to confirm each responsive class is applied at the right breakpoint and flag inconsistencies. This is pattern-matching against a known ruleset — high precision but low reasoning depth. Haiku handles code pattern analysis at this scope accurately. Sonnet would be overkill; the task does not require inferring design intent or architectural judgment.

**Token estimate:**
- Input: ~3,000 tokens (8 components × ~350 tokens average + breakpoint reference)
- Output: ~600 tokens (component-by-component checklist with pass/fail and specific line flags)
- **Subtotal: ~3,600 tokens**
- **Cost: (3K × $0.0008) + (600 × $0.004) ≈ $0.0024 + $0.0024 = ~$0.005**

---

### (h) Decide whether to use Next.js 16 App Router vs Remix for this project

**Recommended model: Claude Opus 4**

**Justification:**
This is a high-stakes architectural decision with long-term consequences that are difficult to reverse. The analysis requires:
- Deep understanding of both frameworks' routing models, data loading strategies, caching mechanisms, and deployment targets
- Reasoning about the specific project requirements (café/ecommerce, SEO needs, team expertise, hosting constraints)
- Tradeoff analysis across multiple axes: DX, performance, ecosystem maturity, Vercel lock-in, streaming support, form handling
- Producing a defensible recommendation that a technical lead could act on confidently

This is exactly the class of problem where Opus's superior reasoning depth and ability to hold multiple competing considerations in tension produces meaningfully better output than Sonnet. A shallow recommendation here could result in a costly framework migration months later.

**Token estimate:**
- Input: ~800 tokens (project requirements, team context, constraints brief)
- Output: ~1,500 tokens (structured comparison, recommendation, risk factors, migration path if needed)
- **Subtotal: ~2,300 tokens**
- **Cost: (800 × $0.015) + (1,500 × $0.075) ≈ $0.012 + $0.1125 = ~$0.125**

---

## Summary Table

| # | Task | Model | Est. Tokens | Est. Cost |
|---|---|---|---|---|
| (a) | Design system from scratch | Opus 4 | ~8,000 | ~$0.48 |
| (b) | Alt text for 47 images | Haiku 3.5 | ~3,800 | ~$0.009 |
| (c) | 3 CTA button variants | Haiku 3.5 | ~260 | ~$0.0004 |
| (d) | Accessibility audit (500 lines JSX) | Sonnet 4.5 | ~7,500 | ~$0.04 |
| (e) | "Our Story" page (500 words) | Sonnet 4.5 | ~1,200 | ~$0.012 |
| (f) | Format metadata for 30 products | Haiku 3.5 | ~4,500 | ~$0.01 |
| (g) | CSS breakpoint review (8 components) | Haiku 3.5 | ~3,600 | ~$0.005 |
| (h) | Next.js 16 vs Remix architecture decision | Opus 4 | ~2,300 | ~$0.125 |
| | **TOTAL** | | **~31,160 tokens** | **~$0.68** |

---

## Cost Distribution by Model

| Model | Tasks | Est. Cost | % of Total |
|---|---|---|---|
| Claude Opus 4 | (a), (h) | ~$0.605 | ~89% |
| Claude Sonnet 4.5 | (d), (e) | ~$0.052 | ~8% |
| Claude Haiku 3.5 | (b), (c), (f), (g) | ~$0.024 | ~3% |

---

## Key Patterns and Decision Framework

**Use Opus when:**
- The decision is architecturally irreversible or high-stakes
- Multiple competing tradeoffs must be held simultaneously
- Errors in the output will compound into downstream work
- Creative output defines the brand identity or system foundation

**Use Sonnet when:**
- Technical depth is required but the problem scope is bounded
- Quality long-form writing is needed (not just templates)
- Code analysis requires understanding of design intent, not just pattern matching
- The output is one component of a larger system, not the system itself

**Use Haiku when:**
- The task is repetitive, high-volume, or structured extraction
- The output follows a fixed schema or template
- Quality is bounded by the format (short copy, checklists, metadata fields)
- Mistakes are cheap to catch and correct

**Total estimated session cost: ~$0.68**
This is extremely cost-efficient for a full project kickoff session covering architecture, content, code review, and catalog work.
