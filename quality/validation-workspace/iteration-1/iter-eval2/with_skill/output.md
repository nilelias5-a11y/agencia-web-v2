# Café Vermut — Round 2 Iteration Summary

**Project:** Café Vermut
**Feedback received:** 2026-05-13
**Source:** Client (free-form text message)
**Round:** 2 (continues sequentially from Round 1)

---

## Raw Feedback (Original)

> "El color que pusisteis está bien pero todavía queda algo frío comparado con lo que tenía en mente. ¿Se puede añadir una textura de papel o algo así de fondo? También quiero poner un contador que muestre los años que llevamos abiertos (abrimos en 2003). Y otra cosa importante: quiero que el botón de 'Comprar' aparezca flotante en móvil cuando haces scroll por la tienda, como hacen las tiendas grandes."

---

## Step 1 — Intake Notes

Three distinct requests extracted from client message. One request is a continuation of a resolved Round 1 item (R1-001 background color). The other two are new requests not present in the Round 1 history.

---

## Step 2 & 3 — Categorization and Duplicate Detection

### R2-001 — Background visual warmth / paper texture
- **Category:** `visual` + `scope-new` (texture element)
- **Client language:** "todavía queda algo frío" / "textura de papel o algo así de fondo"
- **Duplicate check:** Partially related to R1-001 (hero background color → warm cream, resolved). The warmth complaint is a continuation of that thread — the client is not fully satisfied with the color alone and now wants a texture layer. The color change itself was marked ✅ RESOLVED in Round 1, but the underlying client intent (warm, cozy visual feel) was not fully delivered. This is a **soft reopening** of R1-001, not an identical duplicate.
- **Scope assessment:** The original brief presumably specified a background color change (R1-001 was a color item). A paper texture is a new design element — it was not in the original brief. However, the effort is low (CSS background pattern or a subtle image overlay) and it serves the same stated goal as R1-001. See Scope Flag below.

### R2-002 — Years-open counter (since 2003)
- **Category:** `structural` / `functional`
- **Client language:** "un contador que muestre los años que llevamos abiertos (abrimos en 2003)"
- **Duplicate check:** Not present in any Round 1 item. New request.
- **Scope assessment:** A dynamic counter (calculated as `current year − 2003`) is a small new UI element. It likely involves a brief of hero or about section additions. The element is simple (no external integration needed — pure calculation), but it adds a new section or stat block not in the original brief. Moderate scope flag warranted.

### R2-003 — Floating "Comprar" CTA button on mobile scroll
- **Category:** `scope-new` / `functional`
- **Client language:** "botón de 'Comprar' aparezca flotante en móvil cuando haces scroll por la tienda, como hacen las tiendas grandes"
- **Duplicate check:** Not present in Round 1. New request.
- **Scope assessment:** A sticky/floating call-to-action button on mobile during scroll in the shop section is a **significant functional and UX feature**. It requires:
  - Intersection Observer or scroll event logic in JavaScript/React
  - Mobile-specific CSS positioning (fixed, z-index management)
  - Testing across mobile breakpoints and devices
  - Potential conflict with existing footer/nav elements
  This is clearly outside the original brief and constitutes new feature development exceeding 2 hours. **Full scope creep — escalate to director.**

---

## Step 4 — Priority Assignment

| ID | Category | Priority | Rationale |
|----|----------|----------|-----------|
| R2-001 | visual / scope-new | P2 + Scope flag | Client explicitly unsatisfied with R1-001 result; continuation of unmet intent. Low effort if handled as CSS texture. But texture element is out of brief — needs director green-light before execution. |
| R2-002 | structural / functional | P2 | Explicit client request; simple implementation (date math); adds trust signal to the site. Fits current sprint. |
| R2-003 | scope-new / functional | Scope — Escalate | New feature not in brief; >2 hours development; mobile-specific behavior; touches shop page interaction model. Cannot absorb without director sign-off and likely a change order. |

---

## Step 5 — Agent Assignment

| ID | Description | Assigned to | Notes |
|----|-------------|-------------|-------|
| R2-001 | Background paper texture (if approved) | `ui-designer` → `frontend-developer` | Designer proposes texture options (CSS pattern / subtle image); dev implements. Pending director approval of scope. |
| R2-002 | Years-open counter (2003 to present) | `frontend-developer` | Simple computed value `(2026 - 2003) = 23 years`. Placement recommendation: hero stats area or "Sobre Nosotros" section. `ux-designer` should advise on placement. |
| R2-003 | Floating mobile CTA in shop | → escalate to `director` | Cannot assign until scope is approved and effort is quoted. |

---

## Scope Flags Requiring Director Decision

### Scope Flag 1: Paper Texture Background (R2-001)

```
Scope Flag: Client requests paper/textile texture as background treatment
Why it's out of scope: Original brief (as evidenced by R1-001) specified a color change only.
A texture layer is a new design element not in the agreed deliverables.
Estimated effort: 1–3 hours (design exploration + implementation)
Recommendation: LOW RISK to include in Round 2 — it directly resolves the unmet warmth
intent from R1-001 and is unlikely to destabilize the design. Recommend absorbing into
Round 2 at no extra cost as goodwill gesture, OR presenting 2–3 texture options to client
for approval before implementation to avoid another partial-resolution cycle.
Director action needed: Approve absorption into Round 2 sprint, or issue micro change order.
```

### Scope Flag 2: Floating Mobile CTA Button (R2-003)

```
Scope Flag: Sticky/floating "Comprar" button on mobile scroll in shop section
Why it's out of scope: Not mentioned in original brief. Introduces new interaction pattern
and mobile-specific JavaScript behavior. Changes the UX model of the shop page.
Estimated effort: 4–8 hours (logic + cross-device testing + QA)
Recommendation: This is a legitimate conversion-rate feature, but it is Phase 2 material.
Suggest presenting it to client as "confirmed for Phase 2 sprint" with a quoted timeline.
Do NOT absorb into Round 2 — effort is too large and mobile UX changes require proper QA.
Director action needed: Client communication + change order or Phase 2 scheduling.
```

---

## Duplicates / Continuations from Previous Rounds

| Current Item | Previous Round | Relationship |
|---|---|---|
| R2-001 (warmth / texture) | R1-001 (background color → warm cream ✅) | **Soft continuation.** R1-001 was technically resolved (color changed) but the underlying client goal — a warm, non-cold visual feel — was not fully achieved. The texture request is the client articulating the remaining gap. Not a duplicate: the previous fix was correct but insufficient. |

No other Round 1 items reappear in Round 2 feedback.

Round 1 pending item R1-006 (Playfair Display typography — awaiting licensed font from client) remains open and is **not addressed in this round**. Client has not mentioned it. Status unchanged: ⏳ PENDING CLIENT ACTION.

---

## Full Round 2 Item Table

| ID | Category | Priority | Description | Assigned to | Status |
|----|----------|----------|-------------|-------------|--------|
| R2-001 | visual / scope-new | P2 + Scope | Add paper texture to background (continuation of R1-001 warmth goal) | `ui-designer` + `frontend-developer` — pending director approval | ⏳ Scope review |
| R2-002 | structural / functional | P2 | Years-open counter (opened 2003, currently 23 years) | `frontend-developer` + `ux-designer` (placement) | 📋 Assigned |
| R2-003 | scope-new / functional | Scope — Escalate | Floating "Comprar" CTA on mobile scroll in shop | → `director` | 🚩 Escalated |

---

## Processed Items Summary

**Total new items this round:** 3
- P1 Immediate: 0
- P2 This iteration: 2 (R2-001 pending scope approval, R2-002 clear to proceed)
- P3 Next iteration: 0
- Scope flags: 2 (R2-001 minor / R2-003 major)

**Clear to execute immediately:** R2-002 (years counter)
**Pending director decision:** R2-001 (texture scope), R2-003 (floating CTA)

---

## Open Items Carried from Round 1

| ID | Description | Status |
|----|-------------|--------|
| R1-006 | Hero typography — Playfair Display (awaiting client font file) | ⏳ PENDING CLIENT ACTION — no update this round |

---

## Estimated Completion

- **R2-002** (years counter): Can be completed within current sprint — estimated 2–4 hours including placement decision and implementation.
- **R2-001** (texture): If approved by director, 1–3 days depending on design exploration cycle. Risk of another partial-resolution if texture options are not shown to client for pre-approval.
- **R2-003** (floating CTA): Phase 2 — not in current sprint.

**Blocking anything?** No hard blockers on the critical path. R1-006 remains a soft blocker for final typography sign-off but does not block other work. R2-003 must not enter development without a change order — implementing it unrequested would set a bad precedent for scope management.

---

## Director Briefing Note

Two items require your decision before the team can proceed on them:

1. **R2-001 — Texture background:** Low-risk scope absorption recommended. The client's warmth goal from Round 1 was technically addressed but not emotionally landed. A texture layer is a natural completion of that intent. Absorbing it avoids a third round of the same complaint. Suggest we show the client 2–3 options before implementing to close the loop properly.

2. **R2-003 — Floating mobile CTA:** This is a real conversion-optimization feature, not a cosmetic tweak. It requires proper development and QA time. Recommend communicating to the client that it is confirmed for Phase 2 with a quoted timeline. Framing it positively ("great idea, we have it on the roadmap for Phase 2") maintains client satisfaction without absorbing unbudgeted hours.

The R1-006 font file is still pending from the client. Consider a gentle reminder in your next client communication.

---

*Processed by: iteration-agent | Round 2 | 2026-05-13*
