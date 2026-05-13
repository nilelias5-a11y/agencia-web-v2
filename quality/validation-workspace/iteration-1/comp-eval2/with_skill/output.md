## Comparison Report — MDX Files vs Sanity.io CMS

**Date:** 2026-05-13
**Project:** La Nonna — Italian restaurant in Gràcia, Barcelona
**Decision type:** Technical
**Framework used:** C — Technical Comparison
**Requested by:** Director / Agency (Nil)

---

### Variant A — MDX Files in Repository
Blog posts stored as `.mdx` files under `/content/blog/`. Processed at build time with `next-mdx-remote`. No CMS or admin interface. Publishing workflow: agency edits/creates `.mdx` files in VS Code, commits to GitHub, Vercel auto-deploys. Images committed to `/public/images/blog/`. Zero ongoing cost.

### Variant B — Sanity.io Headless CMS
Posts written in Sanity Studio — a visual editor comparable to Notion/Word. Owner publishes without touching code. `GROQ` queries fetch posts in Next.js via API. Draft preview mode available. Content updates are live without redeploy. Cost: Sanity free tier (10,000 API requests/month, 20GB bandwidth). Implementation: ~20–25h.

---

### Scoring

| Criterion | Weight | Variant A (MDX) | Variant B (Sanity) | Notes |
|---|---|---|---|---|
| Performance impact | 25% | 9 | 7 | MDX compiles at build time — zero runtime API calls, static HTML output, optimal Core Web Vitals. Sanity requires API fetches at runtime (or ISR), adds network dependency and minor latency. |
| Developer experience | 20% | 6 | 8 | MDX is clean for developers but places the entire publishing burden on the agency. Sanity Studio abstracts content ops entirely; schema setup is well-documented and straightforward on Next.js 15. |
| Scalability | 20% | 5 | 9 | MDX files scale poorly for non-technical editors. Every new post requires a developer touch, a git commit, and a deploy. Sanity decouples content from code — the owner can operate independently as volume grows or ownership changes. |
| Ecosystem maturity | 15% | 8 | 9 | `next-mdx-remote` is stable and widely used, but Sanity has a larger community, first-class Next.js App Router support, and active long-term development with a proven free tier. |
| Cost | 10% | 10 | 8 | MDX: zero cost. Sanity free tier covers 3–5 posts/month with significant headroom; cost becomes relevant only above ~500 posts or high traffic volumes — not a realistic concern for this client. |
| Time to implement | 10% | 4 | 6 | MDX requires less infrastructure setup (~8–10h) but generates recurring agency time every time a post is published (estimate: 30–60 min per post including review, commit, deploy). Sanity requires ~20–25h upfront but eliminates per-post agency involvement entirely. |

**Weighted Total calculation:**

| Criterion | Weight | A score | A weighted | B score | B weighted |
|---|---|---|---|---|---|
| Performance impact | 0.25 | 9 | 2.25 | 7 | 1.75 |
| Developer experience | 0.20 | 6 | 1.20 | 8 | 1.60 |
| Scalability | 0.20 | 5 | 1.00 | 9 | 1.80 |
| Ecosystem maturity | 0.15 | 8 | 1.20 | 9 | 1.35 |
| Cost | 0.10 | 10 | 1.00 | 8 | 0.80 |
| Time to implement | 0.10 | 4 | 0.40 | 6 | 0.60 |
| **Weighted Total** | **100%** | — | **7.05** | — | **7.90** |

---

### Analysis

**Where Variant A leads:** Performance is MDX's strongest argument. Static compilation at build time means no API roundtrips, no external dependency at serve time, and near-perfect Core Web Vitals by default. For a restaurant site where page speed matters for local SEO and mobile users, this is a genuine advantage. Cost is also an absolute zero — no vendor dependency, no account to manage.

**Where Variant B leads:** Sanity wins decisively on scalability and developer experience because those criteria map directly to the actual operational model of this project. The owner is non-technical and works in Word and WhatsApp — she cannot use git or a text editor. Variant A means every single post requires agency involvement: an estimated 30–60 minutes of billable or goodwill time per publish event. At 3–5 posts/month over 12 months, that is 18–60 agency hours that are not in scope. Sanity eliminates this entirely. The owner publishes herself on day 1.

**The decisive factor:** The constraint "no developer on retainer, max 2h/month agency maintenance" is the axis on which this decision turns. Variant A structurally violates that constraint — it makes the developer mandatory for every content update. Variant B was designed for exactly this scenario: content editors who are not engineers. The 25h implementation investment is a one-time cost that pays back immediately against recurring per-post agency time.

---

### Bias Check

- **Recency bias:** Not detected. Sanity (Variant B) was evaluated second and scored higher on several criteria, but the scores reflect documented operational differences, not order of presentation.
- **Novelty bias:** Not detected. Sanity is not being favored because it is "more sophisticated" — it is being favored because the client's operational profile (non-technical owner, capped agency hours) directly matches what Sanity solves.
- **Length bias:** Not detected. Variant B had a longer description in the task brief, but scoring was based on criteria, not description length.
- **Confirmation bias:** Not detected. No prior client preference was provided that could anchor scores.
- **Anchoring:** Not detected. Variant A's high performance score (9) did not distort the rest of the evaluation — Variant B's lower performance score (7) was independently justified by the runtime API dependency, not by contrast with A.

---

### Verdict

**Recommended: Variant B — Sanity.io**

**Reasoning:** The project constraint — non-technical owner, agency capped at 2h/month — makes Variant A operationally unworkable regardless of its technical merits. MDX requires developer involvement for every publish event; at 3–5 posts/month, this silently consumes agency capacity that is not in scope and creates a dependency that will eventually cause friction or missed posts. Sanity's 20–25h upfront investment is a one-time cost with a clear payback: the owner becomes self-sufficient on day 1, and the agency's 2h/month budget is preserved for actual maintenance rather than content operations.

**Conditions:** Variant B wins given the actual operational context of this project. If the scenario were reversed — a technical founder who prefers owning all infrastructure, with no non-technical editors — Variant A's simplicity and zero cost would be compelling.

**If you choose Variant A instead:** You must budget ~30–60 min of agency time per published post, document this explicitly in the maintenance agreement, and accept that the owner will have zero autonomy over her own blog. Content delays will occur whenever the agency is unavailable. This is a recurring operational risk, not a one-time trade-off.

---

### Implementation Note (informational, not part of scoring)

For Variant B, the recommended implementation path on Next.js 15 + App Router:
- Define `post` schema in Sanity with fields: `title`, `slug`, `publishedAt`, `mainImage`, `excerpt`, `body` (Portable Text), `category`
- Use `@sanity/client` and `next-sanity` for GROQ queries with ISR (`revalidate: 3600` or on-demand revalidation via webhook)
- Blog route: `/blog` (list) and `/blog/[slug]` (detail)
- Image handling: `@sanity/image-url` or native `next/image` with Sanity CDN
- Studio can be embedded at `/studio` or hosted separately on sanity.io — embedding at `/studio` is recommended to keep the workflow simple for the owner
- Total implementation aligns with the 20–25h estimate; no external services beyond Sanity free tier are required
