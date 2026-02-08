---
name: Request to Append Source Repository
about: Ask platform team to add a new allowed source repo to your AppProject
title: "[APPEND-REPO] Add <repo-name> to np-<your-apm>"
labels: ["append-repo", "team-request"]
assignees: []   # optional: auto-assign to platform team
---

### Request to Append Source Repo

**Your APM code / project name**
(e.g. `team1`, `np-finance`, `np-ecomm`)
:

**Repo URL to add**
(full https:// URL, must be organization-owned or approved)
:

**Why is this repo needed?**
(what kind of manifests / charts / base resources does it contain? Security justification if external.)

**Target environment(s)** (optional)
dev / staging / prod / all :

**Any additional context / risks**

---

**Platform team checklist** (for reviewers)

- [ ] Repo is internal / trusted
- [ ] No secrets / credentials visible in repo
- [ ] Owner agrees to maintain quality & security
- [ ] No wildcard / overly broad paths needed
