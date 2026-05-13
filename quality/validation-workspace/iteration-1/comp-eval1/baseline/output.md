# Hero Section Comparison: Café Vermut
**Date:** 2026-05-13
**Brand:** Café Vermut — artisanal specialty coffee, Barcelona
**Primary goal:** Drive online purchases

---

## Evaluation Criteria & Scoring

| Criterion | Weight | Rationale |
|---|---|---|
| Conversion potential (CTA clarity & urgency) | 25% | Primary goal is online purchases |
| Brand alignment (palette, tone, personality) | 20% | Artisanal, Barcelona-rooted identity |
| Trust & credibility signals | 20% | Specialty coffee buyers are discerning; trust drives purchase |
| Visual hierarchy & scannability | 15% | Users scan before committing; clarity accelerates decision |
| Emotional resonance | 10% | Coffee is sensory — emotion closes the gap to purchase |
| Technical risk & implementation quality | 10% | Animation bugs and layout fragility delay launch |

---

## Variant A — "Centered Power"

### Description
Full-bleed dark olive (#1A2016) background, coffee texture overlay, centered layout, single headline "El café que merece tu mañana" at 72px Playfair Display in cream, one terracotta pill CTA "Comprar café", no imagery, staggered word-reveal animation on load.

### Scores (1–10)

| Criterion | Score | Notes |
|---|---|---|
| Conversion potential | 6/10 | Single CTA eliminates choice paralysis — good. But no supporting information means a cold visitor has zero reason to trust the purchase prompt yet. |
| Brand alignment | 9/10 | Dark olive + cream + terracotta is exactly the palette. Playfair Display headline at large scale feels premium and artisanal. Typography-first approach suits a brand that wants to feel editorial. |
| Trust & credibility signals | 3/10 | Zero differentiators, zero proof. No mention of origin, craft, or local identity. A visitor who doesn't already know the brand gets nothing to anchor trust. |
| Visual hierarchy | 8/10 | One focal point, one action. Nothing competes for attention. |
| Emotional resonance | 7/10 | The headline is evocative and poetic. The dark, moody palette creates atmosphere. But without a sensory image the coffee-craving trigger is verbal only. |
| Technical risk | 6/10 | Staggered word-reveal animation introduces paint delay, potential jank on mobile, and a flicker risk if fonts don't load before JS fires. Requires careful implementation. |

**Weighted score: (6×0.25)+(9×0.20)+(3×0.20)+(8×0.15)+(7×0.10)+(6×0.10)**
= 1.50 + 1.80 + 0.60 + 1.20 + 0.70 + 0.60 = **6.40 / 10**

---

## Variant B — "Split with Hero Image"

### Description
Left column (55%) cream background, left-aligned text, 60px headline + subheadline + 3 bullet USPs (single origin, small batch, Barcelona), primary CTA "Comprar café" + secondary ghost CTA "Ver orígenes". Right column (45%) full-height photo of coffee beans in warm light. No animation.

### Scores (1–10)

| Criterion | Score | Notes |
|---|---|---|
| Conversion potential | 8/10 | Two-CTA hierarchy caters to two buyer stages: ready-to-buy and still-researching. USPs give cold visitors a reason to click before fully committing. The secondary CTA keeps explorers in the funnel. |
| Brand alignment | 8/10 | Cream background and left-aligned editorial layout match the artisanal positioning. The warm coffee-bean photo reinforces the craft angle. Slightly less bold than A on pure palette drama, but more complete as a brand story. |
| Trust & credibility signals | 9/10 | Three concrete USPs (single origin, small batch, Barcelona) are exactly the proof points specialty coffee buyers evaluate before purchasing. "Barcelona" roots the brand geographically, which matters for local pride and quality signals. |
| Visual hierarchy | 7/10 | Split layout works well on desktop. More elements mean slightly more scanning effort, but headline → USPs → CTA is a well-understood F-pattern flow. Mobile requires careful stacking to preserve hierarchy. |
| Emotional resonance | 8/10 | The warm-lit coffee-bean photo activates the sensory imagination in a way pure typography cannot. Visual cues for aroma and texture are powerful in food/drink commerce. |
| Technical risk | 9/10 | No animation. Static split layout is battle-tested. The only risk is image optimisation (Next.js `<Image>` with priority solves it) and responsive stacking — both are routine. |

**Weighted score: (8×0.25)+(8×0.20)+(9×0.20)+(7×0.15)+(8×0.10)+(9×0.10)**
= 2.00 + 1.60 + 1.80 + 1.05 + 0.80 + 0.90 = **8.15 / 10**

---

## Head-to-Head Summary

| Criterion | Variant A | Variant B | Winner |
|---|---|---|---|
| Conversion potential | 6 | 8 | **B** |
| Brand alignment | 9 | 8 | A (marginal) |
| Trust & credibility | 3 | 9 | **B** |
| Visual hierarchy | 8 | 7 | A (marginal) |
| Emotional resonance | 7 | 8 | **B** |
| Technical risk | 6 | 9 | **B** |
| **Weighted total** | **6.40** | **8.15** | **B wins clearly** |

---

## Verdict: Implement Variant B

**Variant B is the stronger choice by a significant margin (8.15 vs 6.40).**

### Why B wins

1. **The trust gap in A is fatal for e-commerce.** A cold visitor landing on a dark, image-free page with only a headline and a "buy now" button has no information to justify a purchase. Specialty coffee is a considered buy — customers want to know provenance, craft, and why this brand is worth their money. Variant B supplies all three in three scannable bullet points.

2. **The dual-CTA structure is strategically sound.** "Comprar café" captures high-intent visitors immediately. "Ver orígenes" keeps discovery-phase visitors engaged rather than bouncing. This widens the funnel without diluting the primary action.

3. **The hero image does work that words cannot.** Warm-lit coffee beans trigger a sensory response — the imagined aroma and texture — that is a documented driver of food and beverage purchase intent. A pure-typography hero forces the headline to carry the entire emotional load, and "El café que merece tu mañana," while good copy, is not enough on its own.

4. **Lower technical risk means faster, safer launch.** The animation in A requires font-load coordination, scroll-safe implementation, and mobile testing. Variant B ships with zero animation risk, letting the team focus on conversion optimisation rather than debug cycles.

### When A would be appropriate

Variant A is a stronger concept for:
- A brand awareness campaign where purchase intent is already primed (retargeting)
- A landing page for an existing customer base who knows the product
- A secondary page (e.g., a seasonal drop) where the drama of full-bleed type fits the moment

For the primary homepage hero driving cold traffic to purchase, B is the correct choice.

### Recommended refinement for B

- Ensure the headline in B is tightened to match the poetic register of A's copy. "El café que merece tu mañana" is stronger than a generic product headline — consider adapting it for B.
- Run the hero image through Next.js `<Image priority />` with a blurred placeholder to eliminate any LCP penalty.
- On mobile, stack image above text (not below) to keep the sensory hook visible before the CTA.

---

*Evaluation produced by Claude agent — Café Vermut hero comparison, 2026-05-13*
