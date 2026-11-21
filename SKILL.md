---
name: rev-skill
description: Alias for audit-skill plus skill rename capability. Use when the user asks to "review a skill", "audit a skill", "check skill security", or "rename a skill" — delegates security workflow to audit-skill and provides rename workflow (directory + frontmatter + git + publish). Prefer audit-skill directly for security reviews.
---

# rev-skill (Alias → audit-skill)

This skill is a **thin alias** for `audit-skill`. All security review logic lives in `audit-skill`.

## When to Use

- User asks to "review a skill" or "audit a skill" and mentions `rev-skill` by name
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

## Delegation

**Do not duplicate audit-skill's workflow here.** Instead:

1. Load and follow `audit-skill` (SKILL.md + scripts/audit_skill.py)
2. All security scanning, manifest recording, and verdict logic is in `audit-skill`
3. This skill adds no additional capabilities (except Rename)

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
- `audit-skill` SKILL.md — full workflow
- `audit-skill` scripts/audit_skill.py — scanner helper
EOF