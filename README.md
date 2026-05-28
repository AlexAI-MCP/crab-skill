# Crab Skill

Crab is a workflow skill for building high-quality LocalCrab/OpenCrab ontology packs from source material.

It is designed for the [OpenCrab SaaS](https://opencrab.sh/) and [AlexAI-MCP/OpenCrab](https://github.com/AlexAI-MCP/OpenCrab) ecosystem: install the LocalCrab MCP server, invoke this skill from Codex or Claude Code, and use it as the operator guide for evidence-grounded ontology building.

LocalCrab is the local ontology factory: it handles crawl planning, source parsing, Evidence indexing, graph construction, Neo4j validation, and pack export. OpenCrab SaaS is the hosted layer that ingests those packs and makes them useful through graph search, Graph RAG workflows, marketplace/community distribution, and connected MCP access. Used together, LocalCrab MCP + Crab Skill + OpenCrab SaaS turns raw documents, URLs, laws, code, and research bundles into reusable ontology knowledge products.

It guides an agent through:

- collecting source material without omission
- chunking source text at the right unit
- building MetaOntology nodes and edges
- ingesting every chunk into Evidence
- running quality gates before promotion
- reporting pack statistics and validation status

## OpenCrab + LocalCrab Flow

```text
source material or crawl target
        |
        v
LocalCrab MCP + Crab Skill
        |
        v
complete source chunks + Evidence ingest
        |
        v
MetaOntology nodes, edges, claims, outcomes
        |
        v
Neo4j validation + OpenCrab Pack v1 ZIP
        |
        v
OpenCrab SaaS ingest, graph search, Graph RAG, and pack reuse
```

Use this skill when you want an agent to build ontology packs with source coverage, traceable evidence, stable graph relationships, and a quality gate before promotion. The resulting pack can stay local for Neo4j/MCP workflows or move into OpenCrab SaaS to maximize reuse across hosted workspaces and connected AI tools.

## LocalCrab MCP

Install and run LocalCrab from the OpenCrab repository:

```bash
git clone https://github.com/AlexAI-MCP/OpenCrab.git
cd OpenCrab
pip install -e ".[dev]"
opencrab serve
```

Then register it with an MCP-capable client, for example Claude Code:

```bash
claude mcp add opencrab -- opencrab serve
```

Once LocalCrab MCP is available, this skill tells the agent how to use the MCP tools in the right order: inspect the ontology manifest, add source-backed nodes, add valid edges, ingest every chunk into Evidence, validate graph quality, then promote.

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
