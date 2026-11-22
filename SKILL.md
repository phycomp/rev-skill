---
name: rev-skill
description: Alias for audit-skill plus skill rename and top-to-bottom logic consistency checking. Use when the user asks to "review a skill", "audit a skill", "check skill security", "check whether rules are logically consistent", "review rules top to bottom", or "rename a skill" — delegates security workflow to audit-skill, provides rename plus logic-consistency workflows that write corrections into the skill, and hooks viz-dboard to verify doc-layer and code-layer demo compliance. Prefer audit-skill directly for pure security reviews.
---

# rev-skill (Alias → audit-skill)

This skill delegates all **security** review to `audit-skill`. It adds two local capabilities: **Rename** and **Logic Consistency Check** (top-to-bottom rule audit that writes corrections into the skill).

## When to Use

- User asks to "review a skill" or "audit a skill" and mentions `rev-skill` by name
- User says "檢查規則是否符合邏輯" / "由上至下" / "將之寫入" — run the Logic Consistency Check below
- Backward compatibility with existing workflows that reference `rev-skill`

## Rename Capability

rev-skill can **rename** an existing skill (directory + frontmatter + repo) in one workflow:

### Rename Workflow

1. **Copy** the source skill directory to the new name:
   ```bash
   cp -r ~/.agents/skills/<old-name> ~/.agents/skills/<new-name>
   ```

2. **Update frontmatter** in `SKILL.md`:
   - `name:` → new name
   - `description:` → update if needed (keep "Use when..." format)

3. **Update `README.md`** (if exists): title, install command, contents list.

4. **Re-audit** with `audit-skill`:
   ```bash
   python3 ~/.agents/skills/audit-skill/scripts/audit_skill.py ~/.agents/skills/<new-name>
   ```

5. **Git setup** (if publishing):
   ```bash
   cd ~/.agents/skills/<new-name>
   rm -rf .git
   git init -b main
   git config user.name <name>
   git config user.email <email>
   git add -A
   git commit -m "Initial commit: <new-name> skill"
   gh repo create <owner>/<new-name> --public --source=. --remote=origin --push
   ```

6. **Update parent-child references** in any skill that referenced the old name.

7. **Deprecate the old skill** (optional): replace old SKILL.md with a thin alias pointing to the new name, or remove entirely.

### Rename Checklist

- [ ] Directory renamed/copied
- [ ] `name:` field updated in SKILL.md
- [ ] `description:` field updated (still starts with "Use when...")
- [ ] README.md updated (title, install command)
- [ ] Audit passed (APPROVE or CAUTION with consent)
- [ ] Drift manifest generated
- [ ] Git repo created and pushed (if publishing)
- [ ] Parent-child references updated in dependent skills
- [ ] Old skill deprecated or removed

## Logic Consistency Check (由上至下檢查規則是否符合邏輯，並寫入)

Security scanning answers "is this skill safe?" — it does **not** answer "are its rules logically coherent?" A skill can score 0/100 LOW SAFE and still teach stale or contradictory behavior. This capability checks the rules from top to bottom and **writes the corrections into the SKILL.md**.

### When to use
- A bug fix changed behavior, but the skill documenting it may still teach the old behavior.
- User says "檢查規則是否符合邏輯", "由上至下", or "將之寫入".
- Before re-audit: logic errors survive a security scan.

### Procedure — read in order, do not skip
1. **Frontmatter**: `name:` matches the directory; `description:` starts with "Use when..." and mentions **every** capability the body claims. A capability present in the body but absent from the description is inconsistent.
2. **Core invariant / overview**: identify the single invariant. Every later rule must be a consequence of it — if a rule contradicts it, the rule is stale or the invariant is wrong.
3. **When to Use**: every listed symptom must be resolvable by some rule later in the document.
4. **Rules, in order**: for each rule ask —
   - Does it contradict an earlier rule or the core invariant?
   - Does its code example actually implement the rule it claims to?
   - Did behavior change while this rule kept the superseded approach (e.g. post-hoc `st.info` correction still documented after switching to option filtering)?
5. **Examples / Common mistakes**: the "Fix" column must match the **current** Rules, not an earlier version.
6. **Cross-references / parent-child links**: referenced skills exist and responsibilities do not overlap.
7. **File hygiene**: no stray terminators (`EOF`), duplicate headings, or leftover artifacts.
8. **Documentation compliance ≠ implementation compliance** (critical): when a rule mandates a backend, widget, or API, open the **actual code** and confirm it complies. A README line, a Demo-Tools table, or a `legacy` / `exception` label in documentation does **not** make a violating implementation compliant. If code contradicts a rule, either convert the code or remove it from the documented demos — never paper over it with a nuance clause.
9. **Hook `viz-dboard` after the logic check**: a passing doc review is not complete until the domain skill's demo is verified. Invoke `viz-dboard` by name — it owns Demo Tools, the backend rules, and the demo-code verification method — and have it confirm "doc compliance + code compliance":
   - **Doc layer** — every example/table/guardrail in the skill's `SKILL.md` obeys the rule.
   - **Code layer** — every listed demo's `app.py` actually uses the mandated backend.
   For `qbit-viz` that means `qbit-gate-viz/app.py` (2D matplotlib) and `bloch-evolution-viz/app.py` (PyVista + Panel), plus its own example snippets. Record the verdict in the `Logic check:` line; fix both layers if either fails. **Do not copy viz-dboard's demo rules here — hook only.**

### Write the result
- Edit `SKILL.md` in place with the corrections — do not merely report them.
- Add a `Logic check:` line under `References` recording the date and what was corrected (or "no stale rules found").
- Re-audit and refresh the drift manifest:
  ```bash
  python3 ~/.agents/skills/audit-skill/scripts/audit_skill.py ~/.agents/skills/<name>
  python3 ~/.agents/skills/audit-skill/scripts/audit_skill.py --verify ~/skill-reports/manifests/<name>.sha256 ~/.agents/skills/<name>
  ```

### Checklist
- [ ] `name` / `description` cover all capabilities in the body
- [ ] Core invariant consistent with every rule
- [ ] No stale rule contradicts a newer one
- [ ] Code examples implement their own rule
- [ ] Common mistakes match current rules
- [ ] Cross-references valid, no overlapping responsibility
- [ ] No stray `EOF` / duplicate headings
- [ ] `viz-dboard` hooked to verify doc layer + code layer (e.g. `qbit-viz` demos); no demo rules duplicated here
- [ ] No `legacy` / `exception` label used to excuse a rule violation
- [ ] Corrections written into `SKILL.md`
- [ ] Audit + drift re-run

## Delegation

**Do not duplicate audit-skill's workflow here.** Instead:

1. Load and follow `audit-skill` (SKILL.md + scripts/audit_skill.py)
2. All security scanning, manifest recording, and verdict logic is in `audit-skill`
3. This skill adds no additional **security** capabilities; its local additions are Rename and Logic Consistency Check (above)

## What audit-skill provides

- Content summary (files, commands, network, credentials)
- Static scan via SkillSpector (with noise triage)
- Manual malicious-intent scan
- False-positive discipline
- Risk assessment (APPROVE / CAUTION / REJECT)
- sha256 manifest + drift check
- Mandatory gate for all skill work

## External Trust Anchors
- NVIDIA SkillSpector (optional, for static scan)
- GitHub (for source fetching)

## Guardrails
Same as audit-skill. See `audit-skill` SKILL.md for the full list.

## References
- `audit-skill` SKILL.md — full security workflow
- `audit-skill` scripts/audit_skill.py — scanner helper
- Logic check: 2026-11-22 — added top-to-bottom Logic Consistency Check; fixed description / thin-alias claim / stray `EOF`. Second pass: added step 8 "documentation compliance ≠ implementation compliance" (source of truth, hooked from `viz-dboard` Demo Tools). Third pass: added step 9 "hook the implementation demo after the logic check" (doc layer + code layer), hooked from `viz-dboard`. Fourth pass: step 9 now explicitly hooks `viz-dboard` by name as the verifier (no demo rules copied into this skill); description updated accordingly.

> **Note for reviewers:** when auditing another skill, apply both gates — `audit-skill` for safety, then **Logic Consistency Check** above for rule coherence. A 0/100 LOW SAFE verdict is not a logic verdict.