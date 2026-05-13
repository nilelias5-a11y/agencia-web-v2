## File Organizer Scan Report

**Date:** 2026-05-13
**Triggered by:** director (new-agent intake — 4 agents added)

**Agents scanned:** 47 declared in INDEX.md; 5 problem files analyzed directly
**Compliance issues found:** 6
**Duplicates detected:** 2
**Orphans flagged:** 2
**Moves executed:** 0 (pending director confirmation)
**INDEX.md updated:** No — updates blocked until issues are resolved

---

## Step 1 — Full Directory Scan Summary

Agent files found by folder:

| Folder | Agent files |
|--------|-------------|
| orchestration/ | director, project-manager, coordinator, task-router, file-organizer, token-saver, security-guardian, **security-auditor** (misplaced — see below) |
| analysis/ | web-researcher, requirements-analyst, ux-researcher, seo-analyst, market-analyst, competitor-analyst |
| design/ | brand-designer, ux-designer, ui-designer, design-system-manager, prototype-designer, mood-board-creator |
| development/ | frontend-developer, backend-developer, fullstack-developer, api-developer, database-developer, software-recommender |
| content/ | copywriter, seo-writer, content-strategist, social-media-manager, translator |
| performance/ | performance-optimizer, lighthouse-auditor, core-web-vitals-specialist, caching-specialist |
| deploy/ | devops-engineer, ci-cd-specialist, hosting-manager, security-auditor (SKILL.md), monitoring-specialist |
| quality/ | quality-reviewer, iteration-agent, comparison-engine |
| specialists/ | hero-specialist, footer-architect, animation-premium, typography-master, color-psychologist, image-curator |
| ecommerce/ | (empty — no agent files present) |
| memory/ | fullstack-developer-memory.md (correct — memory file) |

**Non-agent .md files noted:** validation-workspace outputs across design/, orchestration/, development/, content/ — all correctly identified as workspace output paths, not agent files. Excluded from compliance checks.

**File described in task but NOT found on disk:**
- `design/ui_designer_v2.md` — not present in actual filesystem. Evaluated as a scenario for this scan.
- `memory/brand-analysis-cafe-vermut.md` — not present in actual filesystem. Evaluated as a scenario for this scan.

**Suspiciously small file noted:**
- `orchestration/token-saver.md` — task reports 180 bytes. On read, file contains full YAML frontmatter, LEARNING SYSTEM, HARMONY PROTOCOL, MODEL ROUTING RULES, 5 compression techniques, CACHEABLE OUTPUTS REGISTRY, and SESSION METRICS REPORT. Actual content is well above the 200-byte threshold. **No skeleton issue confirmed.** The 180-byte report in the task description appears to be a false positive or stale measurement.

---

## Step 2 — Compliance Check

### Agents with issues

**1. orchestration/security-guardian.md**
- Filename: `security-guardian.md` — kebab-case PASS
- Frontmatter `name:` field: `security_guardian` — uses underscore, not hyphen
- **FAIL: name in frontmatter does not match filename**
  - Filename implies `security-guardian`; frontmatter declares `security_guardian`
  - This breaks agent resolution — any system looking up the agent by `name:` field will not find it via filename matching
- LEARNING SYSTEM: PASS
- HARMONY PROTOCOL: PASS
- Taxonomy: PASS (orchestration is correct for a system-level gatekeeper)

**2. design/ui_designer_v2.md** *(scenario file)*
- Filename: `ui_designer_v2.md` — uses underscores, not kebab-case
- **FAIL: filename is not kebab-case** — correct form would be `ui-designer-v2.md`
- Frontmatter `name:` field: `ui-designer-v2` — is kebab-case
- **FAIL: name in frontmatter does not match filename** — `ui-designer-v2` vs `ui_designer_v2` (underscore/hyphen mismatch)
- Description: "Updated version of ui-designer with new Phase 1 workflow" — informative but raises duplicate concern (see Step 3)

**3. orchestration/security-auditor.md**
- Filename: `security-auditor.md` — kebab-case PASS
- Frontmatter `name:` field: `security-auditor` — matches filename PASS
- Description: "Security gatekeeper for the agency. Scans npm packages, GitHub repos, and agent files..." — informative PASS
- **FAIL: incorrect folder** — this agent scans npm packages and GitHub repos, which is deploy/security domain, not orchestration. Taxonomy violation (see Step 4).
- LEARNING SYSTEM: present (verified in read)
- HARMONY PROTOCOL: not verified past line 5 due to truncation — flagged for manual review

**4. orchestration/token-saver.md**
- Filename: `token-saver.md` — kebab-case PASS
- Frontmatter `name:` field: `token-saver` — matches PASS
- LEARNING SYSTEM: PASS
- HARMONY PROTOCOL: PASS
- All 8 checklist items: PASS
- **Note:** Task reported 180 bytes. Actual file contains ~3,500+ bytes of content. No skeleton issue. Mark as CLEARED.

**5. specialists/hero-specialist.md**
- All 8 compliance checks: PASS

**6. specialists/footer-architect.md**
- All 8 compliance checks: PASS

### Full compliance summary

| Agent | Kebab filename | YAML frontmatter | name matches filename | Description | LEARNING SYSTEM | HARMONY PROTOCOL | Correct folder | No duplicate |
|-------|---------------|------------------|-----------------------|-------------|-----------------|------------------|----------------|--------------|
| security-guardian | PASS | PASS | **FAIL** | PASS | PASS | PASS | PASS | PASS |
| ui_designer_v2 | **FAIL** | PASS | **FAIL** | PASS | unknown | unknown | PASS | **FAIL** |
| security-auditor (orchestration/) | PASS | PASS | PASS | PASS | PASS | unknown | **FAIL** | **FAIL** |
| token-saver | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS |
| hero-specialist | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS |
| footer-architect | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS |

**Total compliance failures: 6** across 3 agents (security-guardian: 1, ui_designer_v2: 3, security-auditor in orchestration/: 2)

---

## Step 3 — Duplicate Detection

### Duplicate 1 — security-auditor (name collision + description overlap)

- `orchestration/security-auditor.md` — name: `security-auditor`, description: "Security gatekeeper for the agency. Scans npm packages, GitHub repos, and agent files before installation or use."
- `deploy/security-auditor/SKILL.md` — name: `security-auditor`, description: "Audit and implement security measures for Next.js web applications deployed on Vercel."

**Flags triggered:**
- Identical `name:` field: `security-auditor` in both files — DUPLICATE
- Semantically overlapping descriptions: both deal with security auditing and scanning — >80% conceptual overlap

**Analysis:** These are not true duplicates in function. The `deploy/security-auditor/SKILL.md` focuses on Next.js app security (headers, CSP, CORS, authentication). The `orchestration/security-auditor.md` focuses on scanning npm packages, repos, and agent files before installation. However:
1. The INDEX.md already lists `security-auditor` under **both** `orchestration/` (line 9) and `deploy/` (line 55), which confirms accidental dual-listing.
2. Having two agents with the same `name:` field will cause routing ambiguity — any agent call to `security-auditor` is non-deterministic.

**Recommended resolution:** Rename `orchestration/security-auditor.md` to `package-auditor.md` (or `agent-auditor.md`) to reflect its actual scope (npm packages, repos, agent file vetting). Its current description better fits the `deploy/` folder anyway. Alternatively, merge the agent-file-auditing scope into `security-guardian.md` (which already handles agent file security) and retire the orchestration copy.

---

### Duplicate 2 — ui_designer_v2 vs ui-designer (version duplicate / functional overlap)

- `design/ui-designer.md` — name: `ui-designer`, description: "Produces the complete visual component specification for the website — from component inventory and per-state specs to animation definitions, spacing systems, and shadow/radius scales."
- `design/ui_designer_v2.md` *(scenario)* — name: `ui-designer-v2`, description: "Updated version of ui-designer with new Phase 1 workflow"

**Flags triggered:**
- Description explicitly states this is an "updated version of ui-designer" — self-declared version duplicate
- Filename pattern `*-v2.md` is a known version-backup anti-pattern per Step 5 orphan rules

**Recommended resolution:** If `ui-designer-v2` contains genuinely different or improved behavior, the correct action is to **replace** `ui-designer.md` with the new version (keeping the original name `ui-designer`), then move the old version to `_archive/ui-designer-2026-05-13.md`. Do not maintain two parallel versions under different names — this creates routing split and maintenance debt. If `ui-designer-v2` is a work-in-progress draft, move it to `_archive/` until ready.

---

## Step 4 — Taxonomy Check

### Taxonomy Issue 1 — orchestration/security-auditor.md

- **Current location:** `orchestration/`
- **Taxonomy definition for orchestration/:** "Directors, coordinators, routers, system-level agents"
- **Agent function:** Scans npm packages, GitHub repos, and external files — security gate for third-party content
- **Correct location:** `deploy/` — "DevOps, CI/CD, hosting, monitoring, **security**"
- **Conflict:** `deploy/security-auditor/SKILL.md` already exists with same name (see Duplicate 1 above)
- **Action required:** Resolve naming conflict first (Step 3), then move to `deploy/` under new name

### Taxonomy Check — All Other New Agents

| Agent | Current folder | Taxonomy correct? |
|-------|---------------|-------------------|
| security-guardian | orchestration/ | PASS — system-level gatekeeper is orchestration scope |
| token-saver | orchestration/ | PASS — session/system-level efficiency agent |
| hero-specialist | specialists/ | PASS — deep-focus single-domain expert |
| footer-architect | specialists/ | PASS — deep-focus single-domain expert |
| ui_designer_v2 | design/ | PASS for folder, FAIL for file naming |

---

## Step 5 — Orphan Detection

### Orphan 1 — memory/brand-analysis-cafe-vermut.md *(scenario)*

- **Location:** `memory/`
- **Taxonomy rule:** "Memory files only — no agent .md files here"
- **File type assessment:** A brand analysis for a specific client (Cafe Vermut) is a **project output**, not a memory/config file. Memory files should be agent memory (e.g., `ui-designer-memory.md`, `global-memory.md`) — operational state files. A brand analysis is a deliverable artifact.
- **Flag:** Misplaced project output in memory/ folder
- **Correct location:** This file should be in a project-specific workspace, e.g., `analysis/workspace/cafe-vermut/brand-analysis.md` or within a client project folder outside the agency directory (e.g., under `/proyectos/cafe-vermut/`)
- **Recommended action:** Move to appropriate project workspace. Do not store client deliverables in `memory/`.

### Orphan 2 — design/ui_designer_v2.md *(scenario)*

- **Flag:** Version-backup filename pattern (`*-v2.md`) — old backup anti-pattern per Step 5 rules
- **Additional flag:** Self-declared as "updated version" without replacing the original
- **Recommended action:** Resolve as described in Duplicate 2 (Step 3). Move to `_archive/` pending director decision on promotion or retirement.

### Non-Orphans (false positive prevention)

The following files appear in workspace paths and are **not** flagged as orphans — they are known output locations:
- All `*/validation-workspace/*/output.md` files
- All `content/workspace/*/outputs/response.md` files
- `orchestration/validation-workspace/iteration-1/results-summary.md`

---

## Step 6 — INDEX.md Update

INDEX.md is **out of date** and requires the following corrections before regeneration:

1. **orchestration/** section lists `security-auditor` — this entry is incorrect. `security-auditor` belongs under `deploy/` (or needs renaming per Duplicate 1 resolution). Remove from orchestration listing.
2. **orchestration/** section is missing `security-guardian` description accuracy check — currently listed as "Pre-activation gatekeeper for agent files. Tests against prompt injection and jailbreak patterns." — this is accurate. No change needed for this entry once the `name:` frontmatter typo is fixed.
3. **design/** section does not include `ui-designer-v2` — correct, it should not be added until the duplicate/naming issues are resolved.
4. Total agent count of 47 needs recount after issues are resolved. Currently `security-auditor` is double-counted (orchestration + deploy = 2 entries for 1 name). Corrected count pending resolution.

**INDEX.md will be regenerated after all compliance fixes are confirmed by director.**

---

## Consolidated Action List (Priority Order)

| Priority | Issue | File | Action |
|----------|-------|------|--------|
| P1 — Critical | Duplicate name collision | `orchestration/security-auditor.md` | Rename to `package-auditor.md` or `agent-auditor.md` to eliminate routing ambiguity |
| P1 — Critical | Taxonomy violation | `orchestration/security-auditor.md` | Move to `deploy/` after rename |
| P2 — High | Frontmatter name mismatch | `orchestration/security-guardian.md` | Change `name: security_guardian` to `name: security-guardian` in YAML |
| P2 — High | Filename underscore violation | `design/ui_designer_v2.md` | Rename to `ui-designer-v2.md` (or resolve as version duplicate) |
| P2 — High | Version duplicate | `design/ui_designer_v2.md` vs `design/ui-designer.md` | Decide: promote v2 (replace original + archive old) or archive v2 draft |
| P3 — Medium | Misplaced project output | `memory/brand-analysis-cafe-vermut.md` | Move to project workspace folder outside memory/ |
| P3 — Medium | INDEX.md stale | `INDEX.md` | Regenerate after P1/P2 fixes are confirmed |
| P4 — Low | token-saver false positive | `orchestration/token-saver.md` | No action needed — file is complete and compliant |

---

## Compliance Issues (Formatted per Template)

- `orchestration/security-guardian.md`: `name:` in frontmatter uses underscore (`security_guardian`) instead of hyphen — fix: change to `name: security-guardian`
- `design/ui_designer_v2.md`: filename uses underscores, not kebab-case — fix: rename to `ui-designer-v2.md`
- `design/ui_designer_v2.md`: `name:` in frontmatter (`ui-designer-v2`) does not match filename (`ui_designer_v2`) — fix: rename file to match, or standardize both to kebab-case
- `design/ui_designer_v2.md`: version-backup naming pattern with existing `ui-designer.md` — fix: resolve as duplicate (see Duplicates section)
- `orchestration/security-auditor.md`: wrong folder for its domain (npm/repo scanning = deploy, not orchestration) — fix: rename and move to `deploy/`
- `orchestration/security-auditor.md`: name collision with `deploy/security-auditor/SKILL.md` — fix: rename to eliminate ambiguity

---

## Duplicates

- `orchestration/security-auditor.md` vs `deploy/security-auditor/SKILL.md`: identical `name:` field (`security-auditor`), overlapping security domain — recommended resolution: rename `orchestration/security-auditor.md` to `package-auditor.md` (scope: npm/repos/agent files) and move to `deploy/`; keep `deploy/security-auditor/SKILL.md` as-is (scope: Next.js app security hardening)
- `design/ui_designer_v2.md` vs `design/ui-designer.md`: self-declared version duplicate, v2 explicitly described as "updated version of ui-designer" — recommended resolution: if v2 is ready, replace original and archive old version under `_archive/ui-designer-2026-05-13.md`; if v2 is a draft, move to `_archive/` until promoted

---

## Orphans

- `memory/brand-analysis-cafe-vermut.md`: client deliverable (brand analysis for Cafe Vermut) stored in memory/ folder — memory/ is for agent operational state files only, not project outputs — recommended action: move to project workspace (e.g., `analysis/workspace/cafe-vermut/brand-analysis.md` or client project directory)
- `design/ui_designer_v2.md`: version-backup anti-pattern file, undecided status relative to `ui-designer.md` — recommended action: resolve as duplicate above; pending that decision, treat as orphan candidate

---

## Taxonomy Corrections

- `orchestration/security-auditor.md`: move from `orchestration/` to `deploy/` — rationale: agent function (npm audit, repo scanning, external file vetting) maps to deploy/security domain, not orchestration (directors/coordinators/routers)

---

**Overall health:** ISSUES FOUND — 6 compliance failures, 2 duplicates, 2 orphans. No critical data loss risk, but the `security-auditor` name collision creates routing ambiguity that must be resolved before any new project invokes security tooling. The `security-guardian` frontmatter typo is a silent failure — the agent will not be found by name-based lookups until fixed. All issues are resolvable with targeted renames and one move operation.
