# Vendor lock-in skills for AI agents

Two [Agent Skills](https://agentskills.io) about vendor lock-in. One removes it. One stops it getting in.

They are plain Markdown plus one shell script, so they work with **any agent or tool that reads the
Agent Skills standard** — not just one vendor's assistant. Which felt like the right way to distribute
skills about not being locked in.

```
.kiro/skills/
├── devendor-project/          remove vendor / platform / scaffolding coupling
│   ├── SKILL.md
│   └── references/
│       ├── case-study.md              a full worked example, mistakes included
│       └── discovery-checklist.md     the exhaustive sweep
└── vendor-lockin-guard/       detect and prevent it
    ├── SKILL.md
    ├── references/exit-drills.md      four exercises that produce evidence
    └── scripts/detect-lockin.sh       read-only audit, by hand or in CI
```

---

## The idea they are built on

**Lock-in is rarely the dependencies. The dependencies are the easy part.**

Real lock-in has an *enforcement layer* — machinery that makes removal fail. The project these came
from had three lint rules whose only purpose was to fail the build if vendor code was deleted:

```json
{ "name": "web-app-keeps-platform-runtime",
  "paths": "src/app.tsx",
  "must": { "importFrom": ["@platform/website-runtime"] } }
```

Remove the vendor's badge from your own product and your build breaks. A reasonable developer concludes
the badge is load-bearing. It is not. It is guarded.

So `devendor-project` looks for the enforcement **before** the code. Remove code first and you end up
fighting your own tooling, and concluding the coupling is structural when it is merely defended.

The companion insight, for `vendor-lockin-guard`:

> **Lock-in is not created by using a service. It is created by the cost of stopping.**

A library you can delete in an afternoon is not lock-in even if you use it everywhere. A three-line SDK
that owns your user identities is lock-in even though it is three lines. Judge the exit, not the entry.

---

## Install

### Any agent that reads Agent Skills

Copy the skill folders into wherever your tool looks for them. Each skill is a directory containing a
`SKILL.md` with YAML frontmatter — `name` (matching the folder) and `description` (what it does and when
to use it). Nothing else is required.

### Kiro

```sh
# Project scope — shared with everyone who clones the repo
mkdir -p .kiro/skills
cp -r devendor-project vendor-lockin-guard .kiro/skills/
git add .kiro/skills && git commit -m "Add vendor lock-in skills"

# Or personal, across all your projects (not available on Kiro Web)
cp -r devendor-project vendor-lockin-guard ~/.kiro/skills/
```

Skills load by **progressive disclosure**: only each `name` and `description` are read at startup, and
the full body loads when a request matches. So they cost almost nothing until someone asks about lock-in,
vendor coupling, telemetry, or how hard it would be to leave a service.

### Just the audit script

The detector stands alone and changes nothing:

```sh
./vendor-lockin-guard/scripts/detect-lockin.sh . 'vendorname|@vendor'
```

Read-only. Exits non-zero on any HIGH finding, so it works as a CI gate. With no vendor name it still
finds the generic signals: telemetry endpoints, enforcement rules, hash-protection manifests,
"do not remove" comments, vendor state directories, generated config and unread env vars.

---

## What they actually check

**`devendor-project`** — phased, because order matters and getting it wrong is destructive:

- **STOP gates first.** Does the vendor hold data you need? *Exporting it is a prerequisite project, not
  part of this one.* Do they serve live traffic? *Delete-before-DNS is a hard outage.* Do users
  authenticate through them? *That is a user-facing migration, not a refactor.*
- **Neutralise enforcement** before touching code — rules requiring vendor imports, hash-protected files,
  comments asserting necessity.
- **Remove inside-out**: usage, then dependency, then enforcement, then manifest. Verify after each step.
- **Credentials.** Deleting a key from `.env` does not revoke it, and a committed credential lives
  forever in every clone, fork and CI cache.
- **Verify empirically.** Grep cannot prove a runtime call is gone — a transitive dependency or a build
  plugin can still phone home. Cheapest decisive test: block the vendor's hosts and see if anything breaks.

**`vendor-lockin-guard`** — a scorecard, red flags, and guardrails:

- Data ownership, identity (**including: can anyone log in during their outage?**), interface shape, build
  coupling, runtime dependency, exit cost in **hours and in money** — egress fees and auto-renewal are
  lock-in that no code review will ever show you.
- Export must be **tested, not documented.** If you cannot run it before adopting, score it red.
- The **recreatability test** for generated config: *if I delete this file, can I rebuild it from my
  repository?* If not, it holds state that exists only in a dashboard.
- CI guardrails — a denylist and an outbound-host allowlist — so coupling cannot be added silently.

---

## Two rules the skills apply to themselves

**Never do anything destructive on a user's behalf.** No rewriting git history, deleting vendor projects,
rotating live credentials, running migrations or changing DNS without asking. Both skills say so
explicitly. Explain what breaks and let the person decide.

**Say when you could not check something.** "No lockfile found, so transitive dependencies were not
checked" is useful. Silence that reads as a pass is the one genuinely harmful output.

---

## These are tested, and here is the evidence

The audit script had **four false-positive bugs**, all found by running it rather than reasoning about
it. Every one is written into its comments, because the reasoning is the reusable part:

| Bug | Symptom |
|---|---|
| Matched bare `amplitude` | Fired on a physics variable in a wave-motion test |
| Matched every `importFrom:` | Flagged six legitimate structural rules |
| Then matched *nothing* | grep is line-based; the config was pretty-printed |
| Empty argument list | With no config present, grep fell back to scanning the whole tree and printed every `@azure` and `@babel` package in a lockfile |

**Match the construct, never the vocabulary.** A detector that cries wolf gets switched off, and one
that silently matches nothing is worse than none at all.

The fourth only appeared when the skills were installed into a *second* project. If you try them
somewhere new and get noise, that is a bug worth reporting — it is how the last one was found.

---

## Contributing

Wanted, in rough order of usefulness:

1. **False positives.** A noisy check is a broken check. Say what fired and what it should have matched.
2. **Missed surfaces.** Somewhere coupling hides that the discovery checklist does not look.
3. **Other stacks.** The examples lean JavaScript because that is where this was first exercised; the
   method does not. Python, Go, Rust, JVM and .NET equivalents for the dependency, lint and CI checks
   would all be real improvements.
4. **Replacement patterns.** A vendor wrapper you replaced with a platform primitive, and what safety
   the wrapper was quietly providing that you had to keep.
5. **Case studies.** Anonymised, with the mistakes left in. The mistakes are the useful part.

Please keep the tone: verify claims rather than asserting them, name the mechanism with file and line,
and never overstate what was checked.

---

## Licence

MIT. Use them, fork them, ship them inside other tools.

The subject matter felt like it deserved a licence that cannot trap anybody.
