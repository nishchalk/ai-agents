# Java AI Agents — Setup Guide

Two AI agent skills for Java/Maven projects:

| Skill | What it does |
|-------|-------------|
| **java-project-agent** | Interviews you about your project and generates a `PROJECT-AGENT.md` instruction file for any AI agent |
| **java-peer-review** | Performs a structured, severity-labelled peer review of Java code changes |

---

## Prerequisites

| Tool | Minimum version | Check |
|------|----------------|-------|
| [Cursor IDE](https://cursor.com) | 0.40+ | `cursor --version` |
| Java | 11 LTS (17 or 21 recommended) | `java -version` |
| Maven | 3.8+ | `mvn -version` |
| Git | any | `git --version` |

---

## Installation

### Option A — Personal (you only)

Copy both skill directories into your personal Cursor skills folder.
They will be available in **every project** you open.

```bash
# from this directory
cp -r java-project-agent  ~/.cursor/skills/
cp -r java-peer-review    ~/.cursor/skills/
```

### Option B — Project / Team (recommended for teams)

Commit the skill directories into your repository so every contributor gets
them automatically.

```bash
# from your repo root
mkdir -p .cursor/skills

cp -r path/to/java-project-agent  .cursor/skills/
cp -r path/to/java-peer-review    .cursor/skills/

git add .cursor/skills
git commit -m "chore: add Java AI agent skills"
git push
```

> Cursor auto-discovers skills in both `~/.cursor/skills/` and `.cursor/skills/`
> in the open workspace. No further configuration needed.

---

## First-time project setup

### Step 1 — Open your Java project in Cursor

```
File → Open Folder → select your project root (where pom.xml lives)
```

### Step 2 — Invoke the project agent

In Cursor chat, type:

```
Set up the Java project agent for this project
```

The agent will ask you **9 questions** to understand your project:

| # | Question | Why it matters |
|---|----------|---------------|
| 1 | Project name & one-sentence purpose | Populates the overview section |
| 2 | Java version | Enables version-specific advice (records, sealed classes, etc.) |
| 3 | Primary frameworks | Spring Boot, Quarkus, Micronaut, or plain Jakarta EE |
| 4 | Module layout | Single module, multi-module Maven reactor, or microservices |
| 5 | Test strategy | Unit, integration, contract (Pact), E2E |
| 6 | Code style enforced | Checkstyle, Spotless, Google Style, or custom |
| 7 | CI/CD pipeline | GitHub Actions, Jenkins, GitLab CI |
| 8 | Off-limits areas | Generated code, legacy modules, vendor dirs |
| 9 | Deployment target | Docker/K8s, Lambda, bare metal |

### Step 3 — Review the generated file

The agent creates `PROJECT-AGENT.md` at your project root. Open it and:

- Correct anything the agent inferred incorrectly from `pom.xml`
- Add project-specific conventions not covered by the defaults
- Commit it alongside your code

```bash
git add PROJECT-AGENT.md
git commit -m "docs: add PROJECT-AGENT.md for AI agent guidance"
```

From this point on, any AI agent working on the project will load
`PROJECT-AGENT.md` automatically and follow your project's rules.

---

## Running a peer review

In Cursor chat, type:

```
Review these changes  [or]  Do a peer review of PR #42
```

The peer review agent will ask **4 scoping questions**:

| # | Question |
|---|----------|
| 1 | What is being reviewed? (files, diff, PR number) |
| 2 | Is there a `PROJECT-AGENT.md` to load for context? |
| 3 | Any areas to focus on? (security, performance, tests…) |
| 4 | Any known trade-offs or constraints to be aware of? |

### Understanding the output

```
🔴 CRITICAL  — must fix before merge (bug, security flaw, data loss)
🟠 MAJOR     — should fix (violates standards, likely future bug)
🟡 MINOR     — advisory (style, minor best-practice deviation)
🔵 NIT       — optional (cosmetic / formatting)
🟢 PRAISE    — good pattern worth keeping
```

**Merge policy (balanced mode):** A PR is merge-ready when there are
zero 🔴 CRITICAL and zero 🟠 MAJOR items open. MINOR and NIT items
are the author's discretion.

---

## Typical daily workflow

```
1. Write / change code
          ↓
2. Ask peer-review agent to review the diff
          ↓
3. Fix any CRITICAL / MAJOR findings
          ↓
4. Commit and push
          ↓
5. CI runs mvn verify (green required)
          ↓
6. Team member or agent does final review → Approve
```

---

## Sharing with your team

### Via the repository (recommended)

```
your-repo/
└── .cursor/
    └── skills/
        ├── java-project-agent/
        │   ├── SKILL.md
        │   └── JAVA-STANDARDS.md
        └── java-peer-review/
            └── SKILL.md
```

Commit this structure. Every developer who clones the repo and opens it in
Cursor gets both skills with no extra steps.

### Via a shared link / archive

```bash
# Create a zip to send to a colleague
zip -r java-agents.zip java-project-agent java-peer-review JAVA-AGENTS-SETUP.md

# Recipient unzips and copies skills to their personal folder
unzip java-agents.zip
cp -r java-project-agent java-peer-review ~/.cursor/skills/
```

---

## Updating the skills

Skills are plain markdown files — edit them like any other file.

```bash
# Personal update
code ~/.cursor/skills/java-project-agent/SKILL.md

# Project update
code .cursor/skills/java-peer-review/SKILL.md
git add .cursor/skills && git commit -m "chore: update peer review checklist"
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Agent doesn't seem to know the skill | Restart Cursor; skills are loaded at startup |
| Agent ignores `PROJECT-AGENT.md` | Explicitly mention it: "Follow the rules in PROJECT-AGENT.md" |
| Review is too verbose | Ask: "Give me only CRITICAL and MAJOR findings" |
| Review is too lenient | Ask: "Review strictly, flag any deviation from Java best practices" |
| Skill not found in `.cursor/skills/` | Verify the directory contains a `SKILL.md` with valid YAML frontmatter |

---

## File reference

```
~/.cursor/skills/                      (personal skills root)
├── JAVA-AGENTS-SETUP.md               ← this file
├── java-project-agent/
│   ├── SKILL.md                       main agent instructions
│   └── JAVA-STANDARDS.md             detailed Java standards reference
└── java-peer-review/
    └── SKILL.md                       peer review agent instructions

your-project/
├── pom.xml
├── PROJECT-AGENT.md                   ← generated by java-project-agent
└── src/
```
