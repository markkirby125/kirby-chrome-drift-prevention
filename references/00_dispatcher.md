# Standard Operating Procedure (SOP): Chrome Drift Prevention — Partials-as-Source-of-Truth for Multi-Page Static Sites - Dispatcher

**Document ID:** SOP-ARC-DRIFT-001
**Origin:** Extracted from a production fix on a 78-page flat HTML site (September 2026), where a nav item added to 77 of 78 hand-replicated pages produced permanent drift that was discovered weeks later by the site owner in a browser.
**Target Scope:** Any static multi-page site (flat HTML, Cloudflare Pages/Workers, S3, or similar) where shared chrome — navigation, header, footer, legal blocks, sitewide CTAs — is physically replicated in every page file. **Not** for platforms with native templating/SSR (those already solve this).
**Related:** Hide-until-date / edge publish gating is `kirby-scheduled-content`. Do not use this skill for that.

---

## 1. The Failure Mode This SOP Prevents ("Chrome Drift")

**Anatomy:** Shared chrome is copy-pasted into every page as a frozen photocopy. There is no connection between the copies. Any chrome change must be applied N times (once per page) by hand or script. A missed copy = **drift**: one logical element, N divergent physical versions, no mechanism to detect or repair the inconsistency.

**Why it is dangerous:**
- Drift is **invisible until a human notices it** — often weeks later, on the most important page (the one nobody re-checks is frequently the homepage).
- The failure compounds: every un-templated element (nav today, footer tomorrow, sitewide CTAs next) multiplies the mutation surface.
- Mass repairs themselves introduce bugs: invalid nesting (block elements inserted inside `<p>`), variant mismatches, escaped clauses, and double-applied edits.

**Economic signature that you need this SOP:**
- Chrome changes require scripted multi-file mutations with exact-string anchors.
- The site has **2+ chrome "variants"** that must each be handled in every mass edit.
- Any mass edit in the last quarter produced a bug that needed a follow-up fix.
- A chrome element differs between pages and nobody knows which version is canonical.

---

## 2. The Fix in One Line

**Partials-as-source-of-truth + sync script:** shared chrome lives in exactly one file per element; a deterministic script injects it into every page; a `--check` mode detects drift mechanically; hand-editing page copies is prohibited thereafter.

This is **Shape A**. Deliberately NOT Shape B (full static-site generator): Shape B forces a mandatory build for every edit — including one-line copy fixes — and requires migrating all hand-tuned page content into a content model. Reject Shape B unless the site is becoming a content-operations product.

---

## 3. Implementation Runbook

### Step 0 — Census (nav-scoped, never string-scoped)

Count which pages actually carry the chrome element **inside its structural container** (e.g. "Tech Guides" within `<nav>…</nav>`), never as a raw string anywhere in the file. A raw string count produces false PASSes: a section heading or button sharing the phrase will mask a missing nav item (this exact error occurred during the origin fix and delayed the diagnosis).

Output of Step 0: per-page presence matrix + **variant fingerprint table** — group pages by their chrome's item sequence and classes (e.g. `About | Our Pricing | [dropdown] | Tech Guides | Contact Us`). Expect 2–6 variants; enumerate every one. These become the rule table in Step 2.

### Step 1 — Extract Canonical Partials

Pick the **majority variant** as canonical and extract each chrome block into `partials/` (e.g. `partials/nav.html`, `partials/footer-rich.html`).

**⚠ Pitfall — the canonical-source anomaly:** inspect the source page's block for *local anomalies before extracting* (hardcoded `active` classes, omitted self-links, page-specific labels). Whatever is in the extraction gets **faithfully propagated to every page by the sync** — including the anomaly. This bug occurred during the origin fix: the source page marked a nav item `active` for no reason, and the first sync spread it to all 77 other pages.

Placeholder-encode per-page differences as tokens in the partial (e.g. `{{ACTIVE:blog}}` resolved to `" active"` or `""` per page by the rules table). Keep the token set small and table-driven.

### Step 2 — Encode the Variant Rules Table

One entry per page class, mapping page-path patterns to rule values:

| Rule | Example |
| --- | --- |
| Active-class keys | `/blog*`, `/guides*` → mark section item active; `/about` → mark About item |
| Footer variant | 8 named pages → `footer-inline.html`; all others → `footer-rich.html` |
| Self-link policy | Render self-link with active class (recommended, consistent), or omit (legacy) — pick one and normalise |

Rules live in the sync script as a literal map — no config files, no magic.

### Step 3 — Write the Sync Script

Requirements (a ~80-line Node or Python script; match the repo's existing script culture):

1. **Deterministic:** same input → same output. No randomness, no timestamps in output.
2. **Scoped replacement:** replace only the recognised block spans (`<nav…</nav>`, `<footer…</footer>`) via bounded regex or DOM parse. Nothing outside the blocks may change.
3. **`--check` mode (mandatory):** regenerate in memory, diff against disk, report drift per file, **exit non-zero if any drift exists** — no writes. This is the drift alarm; wire it into CI or the deploy pre-flight if the project has one.
4. **Idempotence (mandatory):** running the sync twice must make the second run a no-op. Verify explicitly.
5. **Report:** per-page `SYNC`/`DRIFT`/`clean` lines plus a summary count.
6. **Skip-guard:** pages missing a chrome block are reported for manual attention, never silently skipped or partially written.

### Step 4 — Review the `--check` Divergence Report BEFORE Writing

The check-mode report is the safety payoff: every page that would change is listed, and every divergence is a fact about your current site (omitted items, variant differences, local anomalies). Review the list and confirm each change is intended **before** running write mode. Surprises found here are the bugs you would otherwise have shipped.

### Step 5 — Migration Run + Mechanical Gates

Run write mode, then pass all gates before commit:

- [ ] **Content preservation:** for every changed file, visible text *outside the chrome blocks* is byte-identical to the pre-change version (scripted check — strip the blocks, compare). Any drift outside chrome = the script touched something it must not.
- [ ] **Anchor/element counts:** chrome items present on N/N pages (e.g. nav item 78/78, footer link 78/78).
- [ ] **Tag balance:** every touched file's tag open/close counts balance (a parser walk, not a regex eyeball).
- [ ] **Block-nesting legality:** no `<h2>/<h3>/<table>/<ul>` opened inside a `<p>` (browsers auto-close `<p>` before block elements — source and DOM silently diverge; this defect occurred during the origin fix).
- [ ] **Active-class audit:** per-page active items match the rules table exactly (scripted).
- [ ] **Idempotence:** second `--check` reports zero pending updates.
- [ ] Project-standard gates (typecheck, tests) if the project has them.

### Step 6 — Commit, Deploy, Live-Verify

Commit partials + script + migrated pages together (one atomic change). After deploy, live-verify with a browser/DOM check on one page per variant class — computed-style or DOM query, not just HTTP 200.

### Step 7 — The Ongoing Protocol (this is the actual prevention)

- **Chrome changes are partial edits.** Edit `partials/nav.html`, run sync, commit. Never hand-edit a page's chrome copy — the sync script is the only writer.
- **Run `--check` on every deploy pre-flight** (and in CI). Any drift report is investigated, never silenced.
- **New pages are created with chrome slots and immediately synced.**
- **Re-run the Step 0 census (nav-scoped) after any major content generation** to confirm the invariant holds.

---

## 4. Pitfalls (all encountered during the origin fix)

1. **String-scoped census fallacy** — counting a phrase anywhere in the file instead of inside its structural container produces false "all clear" results and hides real drift. Always census nav-scoped.
2. **Canonical-source anomaly** — the extraction source's local quirks (stray `active` classes, omitted self-links) are treated as canon and mass-propagated. Inspect before extracting.
3. **Segment over-extension in scripted edits** — regex/string operations bounded by "next occurrence of X" can swallow *multiple* unrelated tasks/sections when the boundary marker is missing; looped fix-scripts then modify unrelated blocks. Re-scan from scratch per iteration; verify every changed line after.
4. **Block elements inside `<p>`** — injecting `<h3>/<table>/<ul>` after an opening `<p>` without closing it first produces invalid HTML whose DOM silently diverges from source. Close the paragraph before the injected block; re-open a new one after.
5. **Variant blindness in fingerprints** — class-based item extraction misses items whose class differs (`class="nav-link active"` vs `class="nav-link"`), undercounting coverage and misclassifying which pages have an element. Fingerprint on link href + text, not class.
6. **Exact-string anchors and whitespace** — scripted replacements assert on strings that differ by a `<code>` wrapper, an entity (`&amp;`), or a newline. Always re-extract the exact current bytes before asserting; prefer operating on the containing line/element.
7. **Two footer variants is not "basically one"** — variant counts of 2 tempt "close enough" handling. Each variant needs its own partial and an explicit page→variant rule.

---

## 5. Verification Checklist

```
================================================================================
          CHROME DRIFT PREVENTION — PRE-FLIGHT CHECKLIST
================================================================================
[ ] Step 0: nav-scoped census run; variant fingerprint table complete
[ ] Step 1: partials extracted from majority variant; source inspected for anomalies
[ ] Step 2: variant rules table encoded in the sync script (literal map)
[ ] Step 3: sync script deterministic, scoped, --check mode, idempotent, skip-guarded
[ ] Step 4: --check divergence report reviewed and every change confirmed intended
[ ] Step 5: content-preservation check passed (zero drift outside chrome blocks)
[ ] Step 5: anchor counts N/N; tag balance clean; block-nesting legality verified
[ ] Step 5: active-class audit matches rules table
[ ] Step 5: idempotence proven (second --check = zero updates)
[ ] Step 6: atomic commit; deployed; one live DOM verification per variant class
[ ] Step 7: ongoing protocol documented in the project's agent instructions
================================================================================
```
