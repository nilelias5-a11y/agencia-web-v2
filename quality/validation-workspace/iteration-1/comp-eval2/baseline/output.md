# Technical Comparison: Blog for La Nonna Restaurant

**Project:** La Nonna — Italian restaurant, Barcelona  
**Stack:** Next.js 15, deployed on Vercel  
**Client profile:** Non-technical owner, uses Word/WhatsApp  
**Volume:** 3–5 posts/month  
**Constraint:** No dev retainer after launch

---

## Evaluation Criteria

1. Usability for the client (can the owner publish independently?)
2. Total cost of ownership
3. Implementation effort
4. Maintenance burden post-launch
5. Content workflow fit
6. Technical risk

---

## Option A — MDX Files in the Repository

### How it works
Posts live as `.mdx` files in `/content/blog/`. Publishing requires: creating a file in VS Code, writing valid MDX, committing to GitHub, and waiting for Vercel to deploy.

### Criteria Assessment

| Criterion | Assessment |
|---|---|
| Client usability | Fails. The owner cannot use VS Code, Git, or MDX syntax. Every post requires a developer. |
| Total cost | Zero infrastructure cost, but every post incurs dev time at agency rates. 3–5 posts/month × developer time = ongoing hidden cost. |
| Implementation effort | Low (~5–8h). Simple to build. |
| Post-launch maintenance | High dependency on dev. Owner is blocked without technical help. |
| Content workflow fit | Completely mismatched with Word/WhatsApp user profile. |
| Technical risk | Low for the codebase. High for the client relationship — owner frustration is near-certain. |

### Summary
Option A is a developer-friendly solution applied to a non-developer problem. The zero infrastructure cost is misleading: the owner cannot publish a single post without calling the agency. At 3–5 posts/month, this either creates a permanent unpaid support burden or a frustrated client who stops using the blog.

---

## Option B — Sanity.io Headless CMS

### How it works
Sanity provides a hosted visual editor (Studio). The owner logs into a web interface, writes posts in a rich-text editor comparable to Notion or Google Docs, adds images via drag-and-drop, and publishes. Next.js fetches content via GROQ queries. No redeployment needed on publish.

### Criteria Assessment

| Criterion | Assessment |
|---|---|
| Client usability | Strong fit. Visual editor maps directly to the owner's mental model (Word/Notion-like). Publish without touching code, Git, or terminal. |
| Total cost | Sanity free tier (up to 3 users, 10GB assets, 10k API req/day) is well within this volume. Effectively zero ongoing cost at 3–5 posts/month. |
| Implementation effort | ~20–25h. Meaningful upfront investment. Includes schema definition, Studio config, GROQ queries, Next.js integration, and preview mode. |
| Post-launch maintenance | Low. Owner operates independently. Agency involvement only needed for design changes or schema evolution. |
| Content workflow fit | Direct match. Owner can publish from a phone browser if needed. |
| Technical risk | Low. Sanity is a mature, well-documented platform with official Next.js integration. Vercel + Sanity is a standard production stack. |

### Summary
Option B has a higher upfront cost but transfers publishing ownership to the client permanently. After delivery, the owner publishes recipes and news without contacting the agency. This is the correct solution for a client with no technical background and no dev retainer.

---

## Head-to-Head Comparison

| Criterion | MDX (A) | Sanity (B) | Winner |
|---|---|---|---|
| Client can publish independently | No | Yes | B |
| Infrastructure cost | Zero | Zero (free tier) | Tie |
| Dev hours to build | ~5–8h | ~20–25h | A |
| Dev hours post-launch | High (every post) | Near zero | B |
| Content workflow match | Poor | Excellent | B |
| Maintenance dependency | Permanent | Minimal | B |
| Preview/drafts | No | Yes | B |
| Technical maturity | High | High | Tie |

---

## Recommendation: Option B — Sanity.io

**Sanity is the correct choice for this client.**

The fundamental requirement is not "blog posts in a codebase" — it is "the restaurant owner can publish content without developer involvement." Option A does not satisfy this requirement at all. Option B satisfies it completely.

### Justification

1. **Client independence is non-negotiable.** With no dev retainer after launch, Option A creates a permanent bottleneck. The owner cannot publish without a developer. This either kills the blog within months or generates ongoing unpaid support requests.

2. **The cost argument for MDX is false economy.** Zero infrastructure cost means nothing if every post requires 30–60 minutes of developer time. Sanity's free tier is genuinely free for this volume. The real cost difference is the ~15–17 additional implementation hours — a one-time investment that eliminates all future publishing friction.

3. **The 20–25h implementation cost is proportionate.** This is a standard Sanity + Next.js integration. The return is a client who can run their blog independently for years. Amortized over 12 months at 3–5 posts/month (36–60 posts), the per-post dev cost of Option A will far exceed the Option B implementation cost.

4. **Preview mode and real-time updates are genuine value adds.** The owner can draft posts, review them before publishing, and correct mistakes without triggering a Vercel redeploy. This is a meaningful quality-of-life feature for a non-technical user.

5. **Sanity is a proven production stack with Next.js 15.** Official `next-sanity` package, active maintenance, and extensive documentation reduce integration risk to near zero.

### Delivery Notes

- Configure Sanity Studio as a `/studio` route within the Next.js project (single deployment, single domain).
- Define a minimal schema: `post` (title, slug, body, mainImage, publishedAt, category). Avoid over-engineering the schema at launch.
- Set up `@sanity/image-url` for optimized image delivery through Sanity's CDN — removes the need to manage images in Git.
- Enable GROQ-powered search and tag filtering only if the client requests it after launch.
- Provide a 30-minute onboarding session with the owner showing how to publish, add images, and save drafts. Document it with a short Loom video for future reference.

---

*Evaluation date: 2026-05-13*
