# Java AI Agents — Setup Guide (GitHub Copilot)

Two AI agent configurations for Java/Maven projects:

| Agent | Copilot mechanism | What it does |
|-------|------------------|-------------|
| **java-project-agent** | `.github/copilot-instructions.md` | Always-on project context loaded automatically into every Copilot chat |
| **java-peer-review** | `.github/prompts/java-peer-review.prompt.md` | On-demand peer review prompt invoked from Copilot Chat |

> **Cursor user?** See the original Cursor setup guide in the `SKILL.md` files
> under `java-project-agent/` and `java-peer-review/`.

---

## How Copilot instructions work

GitHub Copilot has no "skills" directory. It uses two mechanisms instead:

| Mechanism | File location | Scope | Loaded |
|-----------|--------------|-------|--------|
| Repository instructions | `.github/copilot-instructions.md` | Whole repo | Automatically on every chat |
| Prompt files | `.github/prompts/*.prompt.md` | Per prompt | On-demand — you reference them in chat |
| User instructions | VS Code settings (JSON) | All your repos | Always |

The **project agent** maps to repository instructions (always-on context).  
The **peer review agent** maps to a reusable prompt file (invoked when needed).

---

## Prerequisites

| Tool | Minimum version | Check |
|------|----------------|-------|
| [VS Code](https://code.visualstudio.com) | 1.90+ | `code --version` |
| [GitHub Copilot extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) | latest | Extensions panel |
| GitHub Copilot subscription | Individual, Business, or Enterprise | [github.com/settings/copilot](https://github.com/settings/copilot) |
| Java | 11 LTS (17 or 21 recommended) | `java -version` |
| Maven | 3.8+ | `mvn -version` |
| Git | any | `git --version` |

Enable prompt files in VS Code settings (required for the peer review agent):

```json
// .vscode/settings.json  OR  user settings.json
{
  "chat.promptFiles": true
}
```

---

## Installation

### Step 1 — Create the Copilot directory structure

Run this once at the root of your Java project:

```bash
mkdir -p .github/prompts
```

### Step 2 — Copy the instruction files

```bash
# Copy repository-level instructions (project agent)
cp path/to/java-project-agent/copilot-instructions.md  .github/copilot-instructions.md

# Copy the peer review prompt
cp path/to/java-peer-review/java-peer-review.prompt.md  .github/prompts/java-peer-review.prompt.md
```

### Step 3 — Commit both files

```bash
git add .github/
git commit -m "chore: add GitHub Copilot Java agent instructions"
git push
```

Every contributor who clones the repo and has Copilot installed gets both
agents with no extra steps.

---

## First-time project setup

### Step 1 — Open your Java project in VS Code

```
File → Open Folder → select your project root (where pom.xml lives)
```

### Step 2 — Generate your project instructions

Open Copilot Chat (`Ctrl+Shift+I` / `Cmd+Shift+I`) and paste:

```
I want to set up GitHub Copilot instructions for this Java/Maven project.
Please ask me the following questions one by one, then generate a
.github/copilot-instructions.md file based on my answers.
Reference #file:pom.xml for build context.
```

Copilot will ask you **9 questions** to understand your project:

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

### Step 3 — Save and verify the generated file

Copilot writes `.github/copilot-instructions.md`. Open it and:

- Correct anything inferred incorrectly from `pom.xml`
- Add project-specific conventions not covered by the defaults
- Commit it

```bash
git add .github/copilot-instructions.md
git commit -m "docs: add Copilot project instructions"
```

From this point Copilot **automatically** loads `.github/copilot-instructions.md`
at the start of every chat session in this repository — no manual referencing needed.

---

## Running a peer review

Open Copilot Chat and reference the prompt file:

```
#file:.github/prompts/java-peer-review.prompt.md

Review the changes in #file:src/main/java/com/example/PaymentService.java
```

Or use the **prompt picker**: type `/` in Copilot Chat and select
`java-peer-review` from the list.

The prompt will ask **4 scoping questions**:

| # | Question |
|---|----------|
| 1 | What is being reviewed? (files, diff, PR number) |
| 2 | Are there any areas to focus on? (security, performance, tests…) |
| 3 | Any known trade-offs or legacy constraints to be aware of? |
| 4 | Should the review apply strict or relaxed standards? |

### To review a pull request

```
#file:.github/prompts/java-peer-review.prompt.md

Review the diff for PR #42. Use @github to fetch the PR details.
```

### Understanding the output

```
🔴 CRITICAL  — must fix before merge (bug, security flaw, data loss)
🟠 MAJOR     — should fix (violates standards, likely future bug)
🟡 MINOR     — advisory (style, minor best-practice deviation)
🔵 NIT       — optional (cosmetic / formatting)
🟢 PRAISE    — good pattern worth keeping
```

**Merge policy (balanced mode):** A PR is merge-ready when there are
zero 🔴 CRITICAL and zero 🟠 MAJOR items open. MINOR and NIT are the
author's discretion.

---

## Optional — personal user-level instructions

To apply Java standards across **all** your repositories (not just this one),
add user-level instructions in VS Code:

```json
// settings.json  (Cmd+Shift+P → "Open User Settings JSON")
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "text": "Always follow Java best practices: constructor injection over field injection, SLF4J over System.out, try-with-resources for closeables, Optional instead of null returns, and immutable collections."
    }
  ]
}
```

---

## Typical daily workflow

```
1. Write / change code in VS Code
          ↓
2. Open Copilot Chat — project instructions auto-loaded
          ↓
3. Reference peer review prompt → ask for review of changed files
          ↓
4. Fix any CRITICAL / MAJOR findings
          ↓
5. Commit and push
          ↓
6. CI runs mvn verify (green required)
          ↓
7. Team member or Copilot does final review → Approve
```

---

## Sharing with your team

The entire setup lives in `.github/` — just commit it.

```
your-repo/
└── .github/
    ├── copilot-instructions.md        ← project agent (auto-loaded)
    └── prompts/
        └── java-peer-review.prompt.md ← peer review agent (on-demand)
```

Every developer who clones the repo gets both agents automatically.
No extensions to install beyond GitHub Copilot itself.

---

## Updating the instructions

Both files are plain markdown — edit them directly.

```bash
# Update project instructions
code .github/copilot-instructions.md

# Update peer review prompt
code .github/prompts/java-peer-review.prompt.md

git add .github/ && git commit -m "chore: update Copilot Java instructions"
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Copilot ignores project instructions | Verify `.github/copilot-instructions.md` exists at the **repo root** (not in a subfolder) |
| Prompt file not showing in `/` picker | Ensure `"chat.promptFiles": true` is set in VS Code settings |
| Instructions seem stale | Copilot caches context — close and reopen the chat panel |
| Review is too verbose | Add to your message: "List only CRITICAL and MAJOR findings" |
| Review is too lenient | Add: "Review strictly against the Java standards in the project instructions" |
| Copilot doesn't see `pom.xml` | Explicitly reference it in chat: `#file:pom.xml` |
| PR review not working | Ensure the `@github` extension is enabled in Copilot Chat |

---

## File reference

```
your-project/                             (repo root)
├── .github/
│   ├── copilot-instructions.md           ← project agent (always-on)
│   └── prompts/
│       └── java-peer-review.prompt.md    ← peer review agent (on-demand)
├── .vscode/
│   └── settings.json                     ← chat.promptFiles: true
├── pom.xml
└── src/
```

---

## Copilot vs Cursor — quick comparison

| Feature | GitHub Copilot | Cursor |
|---------|---------------|--------|
| Always-on project context | `.github/copilot-instructions.md` | `PROJECT-AGENT.md` (via SKILL.md) |
| Reusable prompt agents | `.github/prompts/*.prompt.md` | `SKILL.md` files (auto-discovered) |
| Personal instructions | VS Code `settings.json` | `~/.cursor/skills/` |
| Team sharing | Commit `.github/` to repo | Commit `.cursor/skills/` to repo |
| Agent auto-invocation | Instructions always loaded; prompts manual | Skills auto-triggered by description match |
