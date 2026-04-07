# Agentic AI Guide For Developers

Guidance & templates for efficient agentic product development. Battle-tested practices from a multi-repository, safety-critical systems project.

---

## What's Inside

This repository contains two companion documents that work together to set up and run AI-assisted software development effectively:

### [Agentic AI Guide For Teams](./Agentic-AI-Guide-For-Teams.md)

**Audience:** Tech leads, prompt engineers, and engineering managers.

A practical guide for orchestrating AI agents in your development process. Covers:

- Project structure for agentic AI (instruction files, version pinning, spec repos)
- The design-to-implementation pipeline (HLD > LLD > Epic > Story)
- Writing effective agent instructions
- Defining agent roles & authority levels (development, review, validation, research)
- Quality gates & review pipelines
- Story execution orchestration
- Epic & story template design
- Specification ownership & cross-team coordination
- Completion verification framework
- Escalation framework
- Common pitfalls & lessons learned

### [Agentic AI Agent Instructions](./Agentic-AI-Agent-Instructions.md)

**Audience:** AI agents (loaded into agent context at session start).

The companion rule set that gets loaded into your AI agent's context window. Covers:

- Session & context management (surviving compression and new sessions)
- Version & API trust (never trust training data for versions)
- Fail-fast philosophy (forbidden patterns, correct patterns, `return None` rules)
- No placeholders policy
- Repository boundaries
- Destructive operation safeguards
- Security rules
- Specification compliance
- Code conventions
- Story execution role definition
- Completion verification scans
- Escalation triggers

---

## How to Use

1. **Start with the Team Guide** to understand the overall framework and set up your project structure.
2. **Adapt the Agent Instructions** to your project's specific rules, conventions, and tooling.
3. **Place the adapted agent instructions** in your repository root (e.g., as `CLAUDE.md` for Claude Code) so they're loaded into agent context automatically.
4. **Evolve both documents** as you discover new agent failure modes -- every violation is a rule waiting to be written.

---

## Core Philosophy

These guides are built on five principles learned the hard way:

1. **Design before implementation.** Agents execute plans well. They make architectural decisions poorly.
2. **One story at a time.** Agents lose coherence across large scopes.
3. **Verify everything.** "It compiles" is necessary but insufficient.
4. **Fail-fast, never fail-silent.** The most damaging bugs are silent fallbacks, not crashes.
5. **Escalate decisions, not problems.** Agents implement; humans decide.

---

## Contributing

These documents are living guides. If you've found patterns that work (or don't) in your own agentic development workflow, contributions are welcome.

---

## License

This project is open source. See individual files for details.
