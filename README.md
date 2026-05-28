# Crab Skill

Crab is a workflow skill for building high-quality LocalCrab/OpenCrab ontology packs from source material.

It guides an agent through:

- collecting source material without omission
- chunking source text at the right unit
- building MetaOntology nodes and edges
- ingesting every chunk into Evidence
- running quality gates before promotion
- reporting pack statistics and validation status

## Codex Install

Clone this repository into Codex's global skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/AlexAI-MCP/crab-skill.git ~/.codex/skills/crab
```

After opening a new Codex session, the skill can be invoked explicitly as `$crab` or implicitly with requests such as:

- `팩 만들어줘`
- `온톨로지 빌딩해줘`
- `LocalCrab로 인덱싱해줘`
- `Create an ontology pack from this source`

## Claude Code Slash Command

This repo also includes the Claude Code global slash command files:

```text
claude/commands/crab.md
claude/commands/codex-handoff.md
```

Install them globally:

```bash
mkdir -p ~/.claude/commands
cp claude/commands/crab.md ~/.claude/commands/crab.md
cp claude/commands/codex-handoff.md ~/.claude/commands/codex-handoff.md
```

Then use `/crab` from any Claude Code project.

## Contents

```text
SKILL.md
agents/openai.yaml
claude/commands/crab.md
claude/commands/codex-handoff.md
```

## Notes

This repository contains orchestration instructions, not a new MCP server. It assumes the runtime already has access to LocalCrab/OpenCrab MCP tools such as ontology manifest, node/edge writes, Evidence ingest, and promotion operations.

No license has been declared yet.
