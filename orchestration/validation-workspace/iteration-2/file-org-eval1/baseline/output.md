# File Organizer Compliance Report
**Date:** 2026-05-13
**Scope:** 4 new agents added to the agency
**Evaluated by:** file-organizer agent (baseline — no skill)

---

## SUMMARY

| Severity | Count |
|---|---|
| CRITICAL | 1 |
| HIGH | 2 |
| MEDIUM | 1 |
| LOW | 1 |
| INFO | 1 |

Total issues found: **6**

---

## ISSUE 1 — Frontmatter `name:` uses underscore, filename uses hyphen

**File:** `orchestration/security-guardian.md`
**Severity:** HIGH
**Category:** Frontmatter mismatch

**Description:**
The filename is `security-guardian.md` (kebab-case, correct), but the `name:` field in the YAML frontmatter is `security_guardian` (underscore). This creates a two-identity problem: any agent or orchestrator referencing this agent by name from the frontmatter will use the wrong identifier. The `director`, `task-router`, and `project-manager` all route by the `name:` field. A mismatch means this agent could be unreachable by name lookup.

**Evidence:**
- Filename: `security-guardian.md`
- Frontmatter: `name: security_guardian`

**Recommended action:**
Update frontmatter to `name: security-guardian`. No file rename needed.

---

## ISSUE 2 — Filename uses underscores instead of kebab-case

**File:** `design/ui_designer_v2.md`
**Severity:** HIGH
**Category:** Filename convention violation + potential duplicate

**Description:**
The agency naming convention requires kebab-case filenames for all agent `.md` files. The file `ui_designer_v2.md` uses underscores throughout the filename. This breaks filesystem glob patterns used by the file-organizer, security-guardian review queue, and any tooling that scans `design/*.md`.

Additionally, a file `design/ui-designer.md` already exists with `name: ui-designer`. The new file's description explicitly states "Updated version of ui-designer with new Phase 1 workflow," which signals it may be a versioned replacement rather than a distinct agent. Running two agents with overlapping scope in the same folder without a clear deprecation or merge plan creates routing ambiguity.

**Evidence:**
- `design/ui_designer_v2.md` — `name: ui-designer-v2`, description references parent agent
- `design/ui-designer.md` — `name: ui-designer`, established agent

**Recommended actions (in order of preference):**
1. If `ui_designer_v2` supersedes `ui-designer`: rename to `ui-designer.md`, merge improvements into the existing file, and delete the old version. Update any memory references.
2. If both must coexist as distinct agents: rename to `ui-designer-v2.md` (kebab-case), update description to remove the "updated version of" language and instead define the explicit scope difference versus `ui-designer`.
3. If `ui_designer_v2` is experimental/WIP: move to a `design/drafts/` subfolder until ready for promotion.

**Do not leave both files active under overlapping descriptions.**

---

## ISSUE 3 — Suspicious file size (possible skeleton/incomplete agent)

**File:** `orchestration/token-saver.md`
**Severity:** MEDIUM
**Category:** Incomplete file / deployment risk

**Description:**
The reported file size is 180 bytes — far below the minimum expected for a production-ready agent. A properly structured agent file requires at minimum: YAML frontmatter (name + description), a role statement, LEARNING SYSTEM section, and HARMONY PROTOCOL section. These sections alone typically occupy 800–2,000 bytes. A 180-byte file likely contains only a stub or placeholder content.

Activating a skeleton agent creates a silent failure: the `director` or `task-router` may invoke `token-saver` expecting full behavior, but receive incomplete or incoherent responses. This is particularly dangerous for `token-saver` because it operates at session start and affects all downstream agents.

**Note:** A read of the actual file on disk (2026-05-13) shows the file IS fully developed (130+ lines, all required sections present). The 180-byte figure appears to be from a prior state or incorrect metadata. If the on-disk state has already been corrected, this issue can be closed. If the 180-byte version was deployed to a different environment or cache, that version must be replaced.

**Recommended action:**
Verify the deployed/active version matches the on-disk content. If a skeleton version exists in any environment or cache, replace it immediately with the full file. Add a minimum file-size check (>500 bytes) to the file-organizer's validation pipeline.

---

## ISSUE 4 — Project memory file stored in wrong folder

**File:** `memory/brand-analysis-cafe-vermut.md`
**Severity:** CRITICAL
**Category:** Taxonomy error / data isolation violation

**Description:**
The `memory/` folder is designated for agent runtime memory files — persistent state written by individual agents during sessions (e.g., `ui-designer-memory.md`, `token-saver-memory.md`, `global-memory.md`). A brand analysis document for a specific client project (`cafe-vermut`) is project data, not agent memory. Storing it here violates the agency's data taxonomy in two ways:

1. **Scope contamination:** Any agent reading `global-memory.md` could accidentally traverse adjacent files and ingest client-specific data as if it were agency-wide context. This is especially dangerous for agents like `token-saver` and `security-guardian` that read memory at session start.
2. **Discoverability failure:** Project deliverables stored in `memory/` are invisible to the standard project file structure. The `project-manager` and `director` scan project folders, not the memory folder, for brand documents.

**Recommended action:**
Move `memory/brand-analysis-cafe-vermut.md` to the correct project folder, e.g., `projects/cafe-vermut/brand-analysis.md` or the equivalent path under the active project structure. If a `projects/` directory does not exist, create it. Update any memory references in `brand-designer-memory.md` or similar that point to the current misplaced path.

---

## ISSUE 5 — No `ui_designer_v2.md` file found at reported path (filesystem discrepancy)

**File:** `design/ui_designer_v2.md` (reported)
**Severity:** LOW
**Category:** Filesystem discrepancy / possible deployment gap

**Description:**
The scenario states `design/ui_designer_v2.md` exists. A filesystem scan of `design/` finds the following files: `brand-designer.md`, `design-system-manager.md`, `mood-board-creator.md`, `prototype-designer.md`, `ux-designer.md`, `ui-designer.md`. The file `ui_designer_v2.md` is not present on disk.

This may indicate: (a) the file was described but not yet created, (b) it was created in a different environment or branch, or (c) the filename was already corrected to kebab-case before this audit ran.

**Recommended action:**
Confirm whether the file exists in the intended deployment environment. If it was never created, no action needed. If it exists elsewhere (different machine, branch, or environment), apply the remediation from Issue 2 before merging.

---

## ISSUE 6 — `brand-analysis-cafe-vermut.md` not found in `memory/` on disk

**File:** `memory/brand-analysis-cafe-vermut.md` (reported)
**Severity:** INFO
**Category:** Filesystem discrepancy

**Description:**
The scenario reports this file exists in `memory/`. A filesystem scan of `memory/` finds only `fullstack-developer-memory.md`. The `brand-analysis-cafe-vermut.md` file is not present.

If the file was already moved to the correct location prior to this audit, this is resolved. If it was never created, no action is needed. Logged for traceability.

**Recommended action:**
Confirm current location. If already moved, mark Issue 4 as resolved. If it was never created, close both issues.

---

## COMPLIANT FILES (no issues)

| File | Status |
|---|---|
| `specialists/hero-specialist.md` | PASS — kebab-case filename, `name: hero-specialist` matches, has LEARNING SYSTEM + HARMONY PROTOCOL |
| `specialists/footer-architect.md` | PASS — kebab-case filename, `name: footer-architect` matches, has LEARNING SYSTEM + HARMONY PROTOCOL |

---

## REMEDIATION PRIORITY ORDER

1. **CRITICAL — Move `brand-analysis-cafe-vermut.md` out of `memory/`** (Issue 4)
2. **HIGH — Fix frontmatter `name:` in `security-guardian.md`** (Issue 1)
3. **HIGH — Rename `ui_designer_v2.md` to kebab-case and resolve duplication with `ui-designer.md`** (Issue 2)
4. **MEDIUM — Verify `token-saver.md` deployed version matches on-disk content** (Issue 3)
5. **LOW/INFO — Confirm filesystem discrepancies for Issues 5 and 6**

---

## CONVENTIONS REFERENCE (for future agents)

- All agent filenames: `kebab-case.md` (no underscores, no spaces, no version suffixes unless distinct scope)
- Frontmatter `name:` must exactly match filename stem (e.g., `security-guardian.md` → `name: security-guardian`)
- `memory/` folder: agent runtime memory only (`[agent-name]-memory.md`, `global-memory.md`)
- Project deliverables (brand analysis, briefs, research): go in the project folder, not `memory/`
- Versioned agents must have non-overlapping descriptions; if scope overlaps, merge or deprecate the older file
