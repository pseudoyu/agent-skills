# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

### solidity-gas-optimization

Solidity smart contract gas optimization guidelines based on RareSkills. Contains 80+ techniques across 8 categories, prioritized by impact and safety.

**Use when:**
- Writing new Solidity smart contracts
- Reviewing or auditing existing contracts
- Optimizing gas costs for deployment or execution
- Refactoring contract storage layouts
- Implementing cross-contract interactions

**Categories covered:**
- Storage Optimization (Critical)
- Deployment Optimization (High)
- Calldata Optimization (High)
- Design Patterns (High)
- Cross-Contract Calls (Medium-High)
- Compiler Optimizations (Medium)
- Assembly Tricks (Medium)
- Dangerous Techniques (Avoid)

### operators-notebook

A complete design register for dense BI / admin / operator dashboards. Cool off-white OKLCH ink-blue tokens, Bricolage Grotesque display + Sometype Mono numerics, hairline dividers instead of card shadow, 60/30/10 accent discipline. The visual opposite of "default shadcn SaaS."

**Use when:**
- Designing or migrating admin dashboards, internal tools, BI consoles
- Building Next.js + Tailwind v4 + shadcn/ui apps that need editorial discipline
- Rewriting "default shadcn SaaS-flavored" UIs into a disciplined operator voice
- Establishing a coherent visual language across multiple operator-facing routes

**What you get:**
- Complete OKLCH token contract (`h~250`, `--radius: 0.25rem`)
- Three-font stack via `next/font/google` (Bricolage + body + Sometype Mono)
- Notebook-tuned shadcn/ui primitive variants
- Shared primitives (`PageHeader`, `StatCard`, `SectionLabel`, `EmptyState`)
- Layout chrome (hairline sidebar with ink-dot active, solid header, CSS slide-in mobile nav)
- Chart typography contract (flat bars, dashed grid, hue-locked series)
- Step-by-step big-bang migration playbook
- 10 universal BANs + grep commands to enforce them in CI

## Installation

Install skills using the CLI:

```bash
npx skills add pseudoyu/agent-skills
```

### Codex

Follow the [Codex skills guide](https://developers.openai.com/codex/skills/) and place the skill under `$CODEX_HOME/skills`:

```bash
# from the repo root
# defaults to ~/.codex if CODEX_HOME is unset
cp -r skills/solidity-gas-optimization "$CODEX_HOME/skills/"
```

Codex will auto-discover `SKILL.md` files in that directory on the next start.

### OpenCode

OpenCode discovers skills from `~/.claude/skills/<name>/SKILL.md` automatically. See [OpenCode Skills docs](https://opencode.ai/docs/skills/) for more details.

### Claude Code

Use the `/install-skill` slash command to install directly from GitHub:

```
/install-skill https://github.com/pseudoyu/agent-skills/tree/main/skills/solidity-gas-optimization
```

Add `--personal` to install to `~/.claude/skills/` (available across all projects) or `--project` for `.claude/skills/` (project-specific, default).

See the [Claude Code Skills docs](https://code.claude.com/docs/en/skills) for more details.

### claude.ai

Add the skill to your project knowledge or paste the contents of `SKILL.md` into your conversation.

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**
```
Review this Solidity contract for gas optimizations
```
```
Help me optimize the storage layout for this struct
```
```
What's the most gas-efficient way to implement this ERC721?
```
```
Migrate this admin app to the Operator's Notebook register
```
```
Audit this dashboard for raw color literals and SaaS chrome
```

## Skill Structure

Each skill contains:
- `SKILL.md` - Instructions for the agent
- `references/` - Supporting documentation and detailed rules

## License

MIT
