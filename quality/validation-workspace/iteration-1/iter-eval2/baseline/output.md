# Iteration Round 2 Summary — Café Vermut

**Project:** Café Vermut — Web
**Feedback received:** 2026-05-13
**Source:** Client (free-form text)
**Processed by:** iteration-agent (baseline — no skill)
**Round:** 2

---

## STATUS CARRY-OVER FROM ROUND 1

| ID | Description | Status |
|----|-------------|--------|
| R1-001 | Hero background color changed | ✅ Done |
| R1-002 | Logo size increased | ✅ Done |
| R1-003 | Photos added | ✅ Done |
| R1-004 | Featured coffee origins section added | ✅ Done |
| R1-005 | CTA button color changed | ✅ Done |
| R1-006 | Hero typography — Playfair Display | ⏳ PENDING — waiting for licensed font file from client |

R1-006 carries over as an open blocker. No action can be taken by the agency until the client delivers the font file.

---

## ROUND 2 RAW FEEDBACK

> "El color que pusisteis está bien pero todavía queda algo frío comparado con lo que tenía en mente. ¿Se puede añadir una textura de papel o algo así de fondo? También quiero poner un contador que muestre los años que llevamos abiertos (abrimos en 2003). Y otra cosa importante: quiero que el botón de 'Comprar' aparezca flotante en móvil cuando haces scroll por la tienda, como hacen las tiendas grandes."

---

## PROCESSED FEEDBACK ITEMS

### R2-001 — Paper texture background

| Field | Value |
|-------|-------|
| Category | `visual` |
| Priority | P2 — This iteration |
| Assigned to | `ui-designer` |
| Scope | In scope — extension of R1-001 (background treatment, same surface) |
| Effort estimate | Low–Medium (1–3h: source/create texture asset, apply as CSS background-image or overlay, test across breakpoints and light/dark conditions) |

**Description:**
Client confirms the color approved in Round 1 is close but still reads as "cold." Requesting a paper or tactile texture overlay to add warmth and character — consistent with a café/vermut brand identity that trades in analogue, artisanal feel.

**Notes for ui-designer:**
- Texture should be subtle (low opacity overlay or noise layer) so it does not compromise readability of text or buttons placed over it
- Must be tested at all breakpoints — textures can create performance issues on mobile if not optimized (use WebP or SVG-based patterns)
- If using a raster texture, max file size: 50–80 KB; prefer a tileable pattern
- Options to consider: CSS `noise` filter, SVG grain overlay, or a curated royalty-free paper texture (e.g. from Subtle Patterns / Transparent Textures)
- Verify contrast ratios for all text over textured background still pass WCAG AA after texture is applied
- Get a quick approval from client before finalizing — "papel o algo así" leaves room for interpretation

**Decision needed:** Confirm exact texture style with client (rough grain, linen, newsprint, watercolor paper?) before implementation. A quick 3-option visual mock is recommended rather than building blind.

---

### R2-002 — Years-open counter (2003 to present)

| Field | Value |
|-------|-------|
| Category | `structural` + `functional` |
| Priority | P2 — This iteration |
| Assigned to | `frontend-developer` + `ui-designer` (layout placement) |
| Scope | In scope — it is a single informational UI element, not a new section |
| Effort estimate | Low (1–2h: logic is trivial, primary work is placement and design integration) |

**Description:**
Client wants a dynamic counter showing years in business. Opening year is 2003. As of 2026, that is 23 years. The counter must update automatically each year without manual intervention.

**Implementation spec for frontend-developer:**
```
yearsOpen = currentYear - 2003
// currentYear = new Date().getFullYear()
// As of 2026: 23 years
```
- Must be calculated client-side (or at build time with ISR/revalidation) — never hardcoded
- If SSR/SSG: use `new Date().getFullYear()` at request time or revalidate annually
- Display format TBD (e.g. "23 años sirviendo vermut" or "+ de 20 años" or animated odometer) — confirm with client
- Placement: likely near the About section, hero tagline, or footer — `ui-designer` to confirm placement in layout

**Notes:**
- Counter does not need to be animated unless client requests it (animation = scope creep risk — do not add unbidden)
- Simple text display is fully in scope; an animated number ticker would be P3/scope-check territory

---

### R2-003 — Floating "Comprar" button on mobile (sticky CTA)

| Field | Value |
|-------|-------|
| Category | `scope-new` |
| Priority | Escalate to `director` |
| Assigned to | → `director` decision required before proceeding |
| Scope | OUT OF SCOPE — new interactive feature not in original brief |
| Effort estimate | Medium–High (4–8h: component build, scroll event logic, z-index/overlay management, mobile-only visibility, cart state integration if button must reflect cart count, QA across iOS Safari + Android Chrome) |

**Description:**
Client is requesting a sticky/floating "Comprar" (Buy) button that appears on mobile viewports when the user scrolls through the shop page. This pattern (sometimes called a "sticky CTA" or "floating action button") is common in e-commerce (Zara, ASOS, Amazon mobile) and provides a persistent purchase path.

**Why this is out of scope:**

1. The original brief covered a web presence with a shop section — it did not specify interactive mobile UX patterns or sticky navigation components
2. The feature touches the shop page architecture, which may have implications for the existing CTA button placed in Round 1 (double CTA conflict risk)
3. Depending on implementation, it may require integration with cart state (does the button say "Comprar" generically or does it need to know if items are in the cart?)
4. It requires dedicated mobile QA (iOS Safari has historically buggy `position: sticky` behavior with bottom bars; also conflicts with browser bottom chrome)
5. If the site uses Next.js App Router, this likely requires a client component with scroll event listeners — adds JS bundle weight and hydration cost

**Scope Flag for director:**
```
Scope Flag: Floating mobile "Comprar" button
Why it's out of scope: Original brief did not include sticky/floating interactive
  components or mobile-specific UX patterns beyond responsive layout.
Estimated effort: 4–8h (component + scroll logic + mobile QA)
Recommendation: Option A — Include in Round 2 at [agreed rate], delivered in [N days]
               Option B — Defer to Phase 2 as part of a broader mobile UX pass
               Option C — Implement a simplified version: CSS-only sticky footer bar
                 on mobile (no JS, no cart state) — reduces to ~2h
```

**Director note:** The client framing ("como hacen las tiendas grandes") signals aspirational scope — this is common. Option C (CSS sticky footer bar, no cart integration) is a professional middle-ground that satisfies the visual request without the full engineering cost. Recommend presenting it as the default unless the brief includes e-commerce cart functionality.

---

## DUPLICATE / RECURRING ITEM CHECK

| Item | Previous round match | Notes |
|------|----------------------|-------|
| R2-001 — Texture on background | Partial match: R1-001 (background color). Not a duplicate — it's an extension request. Client is satisfied with the color but wants additional warmth. | Flag: this is the second iteration on the same surface. If a third revision comes, consider locking down the background treatment with a formal sign-off. |
| R2-002 — Years counter | No match | New item. |
| R2-003 — Floating mobile button | No match | New item. |

---

## ROUND 2 ITEM SUMMARY

| ID | Category | Priority | Description | Assigned to | Status |
|----|----------|----------|-------------|-------------|--------|
| R1-006 | visual | P1 (carryover) | Playfair Display typography — hero | Blocked — awaiting font file | ⏳ Client pending |
| R2-001 | visual | P2 | Paper/tactile texture on background | `ui-designer` | 🔲 Needs client texture style confirmation first |
| R2-002 | structural + functional | P2 | Dynamic years-open counter (2003) | `frontend-developer` + `ui-designer` | 🔲 Ready to implement |
| R2-003 | scope-new | Escalate | Floating mobile "Comprar" button | → `director` | ⏳ Awaiting director decision |

**Processed items:** 3 new
- P2 This iteration: 2 (R2-001, R2-002)
- Scope flags: 1 (R2-003)
- Carryover open: 1 (R1-006)

---

## SCOPE FLAGS REQUIRING DIRECTOR DECISION

### Flag R2-003 — Floating mobile "Comprar" button

The client explicitly wants this. It is not in the original brief. The director must decide:

- **Option A:** Accept as Round 2 addition — quote extra hours and proceed
- **Option B:** Defer to Phase 2 — communicate to client with a clear rationale
- **Option C:** Implement CSS-only simplified version (sticky footer bar, no JS scroll logic, no cart state) as an in-scope compromise — ~2h — present to client as a "professional version of what you're describing"

**Recommended path:** Option C, pending director sign-off. Keeps client happy, stays close to in-scope effort, avoids the full e-commerce cart complexity.

---

## OPEN QUESTIONS FOR CLIENT (before implementation)

1. **R2-001 — Texture style:** What kind of texture? Options to present to client: (a) fine paper grain, (b) linen/fabric texture, (c) aged/worn paper, (d) watercolor paper. Recommend a quick 3-option visual before building.
2. **R2-002 — Counter format:** How should the years be displayed? Options: "23 años", "Desde 2003", "Más de 20 años de vermut". Client should confirm copy.
3. **R2-002 — Counter placement:** Hero? About section? Footer? `ui-designer` to propose placement; client to confirm.
4. **R2-003 — Floating button:** Director to decide scope and communicate back to client.

---

## BLOCKING STATUS

| Blocker | Item | Action required |
|---------|------|-----------------|
| Client must send licensed font file | R1-006 | Chase client — this is now Round 2, still unresolved |
| Director must approve scope or reject | R2-003 | Director decision this sprint |
| Client must confirm texture style before dev starts | R2-001 | Quick visual options to client (1 day turnaround) |

---

## NOTES FOR ITERATION LOG

- Client is iterating on the background for the second time (R1-001 → R2-001). The color was accepted, but warmth was not fully met. Consider whether the original brief captured the brand aesthetic precisely enough — future projects should probe "warm vs. cold" perception earlier in design phase.
- "Como hacen las tiendas grandes" is a recurring client mindset trigger that signals aspirational feature creep. Log this pattern in memory for future client onboarding: probe e-commerce UX ambitions before project kickoff.
- R1-006 (font) is now two rounds old and unresolved due to client inaction. Director should communicate a deadline — if the font does not arrive before Round 3, the typography item should be formally deferred to Phase 2 to avoid holding up launch.
