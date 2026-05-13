# Agency Agent Index
*Last updated: 2026-05-13 by file-organizer*

## orchestration/
- **director** — Master creative and technical director of the web agency. Orchestrates all specialists across 10 phases to deliver premium-quality websites. Invoke this agent to start any new web project from scratch or manage an ongoing one.
- **project-manager** — Manages timelines, milestones, and deliverables for web agency projects. Tracks progress, detects blockers, and keeps projects on schedule. Works under the director's authority and communicates progress to both the client and the director.
- **coordinator** — Coordinates communication and output handoffs between specialist agents. Ensures each agent receives clean, complete context from the previous one. Resolves minor inter-agent conflicts and escalates major issues to the director.
- **task-router** — Optimizes model selection and token usage across all agency tasks. Decides which Claude model (Opus/Sonnet/Haiku) to use based on task complexity, compresses context before handoffs, caches repeated results, and reports token consumption per session.
- **security-auditor** — Security gatekeeper for the agency. Scans npm packages, GitHub repos, and agent files before installation or use. Issues SAFE / CAUTION / DANGER verdicts. Blocks dangerous installations automatically. Reads memory of previously verified packages to avoid duplicate work.
- **file-organizer** — Structural integrity guardian of the agency. Automatically organizes agents into correct folders, detects duplicates and orphaned files, and keeps INDEX.md up to date. Invoke when agents are added, moved, or the directory structure feels messy.
- **token-saver** — Token efficiency optimizer for the agency. Compresses inter-agent context, caches reusable analyses, routes simple tasks to cheaper models, and reports session costs. Invoke at session start or when token usage feels excessive.
- **security-guardian** — Pre-activation security gatekeeper for new agents. Tests against prompt injection, jailbreak techniques, and integrity violations before any agent goes live. Issues CLEAR / FLAGGED / BLOCKED verdicts. Distinct from security-auditor (which handles npm/repos) — this agent focuses exclusively on agent files.

## analysis/
- **web-researcher** — Researches the client's current online presence before the project begins. Captures digital footprint, social media, reviews, tone of voice, and visual style. Delivers a complete client dossier to the director. Activated at Phase 0 of every project.
- **competitor-analyst** — Analyzes the top 5 competitors of the client across 6 dimensions. Also researches award-winning websites in the sector from Awwwards, FWA, and CSS Design Awards. Delivers a competitive intelligence report with a comparison table and 5 actionable differentiation recommendations.
- **requirements-analyst** — Conducts structured client interviews to capture all project requirements. Defines tone, values, target audience, functionality needs, budget, and timeline. Delivers a formal Project Brief document. Activated at Phase 1.
- **seo-analyst** — Researches keywords, search intent, and SEO opportunities for the client's sector. Defines technical SEO requirements and quick wins. Generates an initial SEO strategy including featured snippet targets. Activated during Phase 2 alongside market analysis.
- **ux-researcher** — Defines the target user profiles for the client's website. Creates data-grounded personas, user journeys, and identifies real user language from online sources. Feeds insights directly to the ux-designer for wireframe creation.
- **content-strategist** — Develops the full content strategy for the website — messaging hierarchy, tone of voice guidelines, copywriting briefs per section, and editorial calendar. Bridges brand positioning and UX requirements to ensure every word on the site serves a conversion or trust-building purpose.

## design/
- **mood-board-creator** — Creates the visual mood board before any design or development work begins. Selects 8–12 award-winning reference websites from the client's niche, analyzes what makes each exceptional, and proposes 2–3 visual directions for client approval. No visual design starts until the client approves a direction.
- **brand-designer** — Builds the complete brand identity for a client — from visual audit and competitor research to color systems, typography, logo directions, and a professional Brand Guidelines Document. Invoke after the Project Brief is approved and before any UI or development work begins.
- **ux-designer** — Designs the complete information architecture and user experience structure of the website — sitemap, navigation, conversion flows, and section-by-section textual wireframes. Invoke after the Brand Brief is approved and before UI design or development begins.
- **ui-designer** — Produces the complete visual component specification for the website — from component inventory and per-state specs to animation definitions, spacing systems, and shadow/radius scales. Invoke after UX wireframes are approved and before development begins.
- **design-system-manager** — Architects and maintains the complete design token system and globals.css for the project. Translates brand and UI specs into a three-level token architecture, writes the full CSS foundation, and documents every component with ARIA and contrast specifications. Invoke after brand and UI specs are finalized.
- **prototype-designer** — Produces section-by-section page prototypes, the global scroll animation map, all interactive state definitions, and a complete Development Briefing with accessibility checklist. Invoke after UI specs and design system tokens are approved, immediately before frontend development begins.

## development/
- **frontend-developer** — Builds the complete frontend of the website using Next.js 16+, React 19, TypeScript strict, Tailwind v4, Framer Motion, and GSAP. Implements every component, animation, responsive layout, and image optimization from the prototype-designer's Development Briefing. Invoke after the Development Briefing is approved by the director.
- **backend-developer** — Implements all server-side logic for the project — API Routes, Server Actions, authentication with NextAuth v5 and Supabase, environment variable management with T3 Env, rate limiting, and email flows with Resend. Invoke after the tech stack is confirmed and database schema is approved.
- **fullstack-developer** — Audits and validates the integration between frontend and backend — data flow, form flow, auth flow, type safety end-to-end, N+1 query optimization, and error boundary setup. Invoke after both frontend and backend are built, before quality-reviewer runs the full QA pass.
- **api-developer** — Integrates third-party services and APIs into the project — Stripe, Resend, Sendcloud, Google Maps, and others. Asks the client which services they want, recommends the best SDK, and implements each integration completely and autonomously. Invoke when a third-party service needs to be connected.
- **database-developer** — Designs and implements the PostgreSQL database schema in Supabase — naming conventions, mandatory columns, RLS policies, indexes, migrations, idempotent seed data, and TypeScript type generation. Invoke after the technical requirements are confirmed and before any backend code that touches the database is written.
- **software-recommender** — Evaluates and recommends the best third-party software and services for each project based on budget, technical complexity, expected volume, and integration quality. Invoke when the director needs to decide which service to use for payments, email, authentication, database, CMS, shipping, analytics, or any other category.

## quality/
- **quality-reviewer** — End-to-end QA specialist for the agency. Tests websites as a real user across devices, browsers, and interaction states. Produces a structured QA Report with all issues logged by severity. Activated before every client review.
- **iteration-agent** — Structured feedback processor and iteration manager. Takes client or director feedback after a review, categorizes it, prioritizes it, assigns it to the right agents, and tracks completion. Prevents feedback chaos and scope creep.
- **comparison-engine** — A/B variant evaluator for the agency. Objectively compares two design, copy, or technical variants against defined criteria. Produces a structured verdict with scoring and a clear recommendation. Used for hero sections, landing pages, CTA copy, and technical architecture decisions.

## specialists/
- **hero-specialist** — Hero section expert. Designs and implements above-the-fold experiences that capture attention within 3 seconds. Masters layout, hierarchy, CTA placement, and full-bleed visuals. Invoked during Phase 4 for any hero section creation or redesign.
- **footer-architect** — Footer design and implementation specialist. Transforms the most neglected section of websites into a conversion and trust asset. Handles navigation architecture, legal compliance, newsletter integration, and brand consistency in the footer.
- **animation-premium** — Premium animation and micro-interaction specialist. Implements Awwwards-level motion design using GSAP, Framer Motion, and CSS. Designs scroll-triggered animations, page transitions, hover effects, and loading sequences that elevate a site from good to extraordinary.
- **typography-master** — Typography specialist for premium web projects. Selects font pairings, defines the full type scale, establishes hierarchy rules, and implements variable fonts and advanced OpenType features. Invoked during brand design and before any frontend implementation begins.
- **color-psychologist** — Color strategy and implementation specialist. Builds scientifically grounded color palettes aligned with brand psychology, industry context, and cultural sensitivity. Defines the full color token system including semantic colors, dark mode, and accessibility compliance.
- **image-curator** — Visual asset specialist. Curates, optimizes, and manages all images for web projects — from sourcing high-quality stock photography to exporting favicon sets, Open Graph images, and WebP conversions. Ensures every image is visually on-brand and technically optimized.

---
Total agents: 29
Compliance: 29/29 passing
