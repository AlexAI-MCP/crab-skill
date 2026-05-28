---
name: crab
description: "Build high-quality LocalCrab ontology packs from source material. Use when the user asks to make an ontology pack, build/index documents with LocalCrab, create an Evidence-backed pack from laws, URLs, files, code, papers, news, business, or government sources, or says Korean requests such as '팩 만들어줘', '온톨로지 빌딩해줘', '/crab', or 'LocalCrab로 인덱싱해줘'."
---

# Crab Skill

## Overview

Use this workflow to create LocalCrab ontology packs with complete source coverage, full-text Evidence ingestion, and quality gates. This Codex skill mirrors the Claude Code global command at `~/.claude/commands/crab.md`.

This is not a new MCP tool. It is an orchestration guide for using the available LocalCrab/OpenCrab MCP tools correctly.

## Activation Intake

If the user already provided topic, source, and purpose, proceed to domain detection. Otherwise ask exactly these three questions and do not ask for more up front:

1. 주제: 이 팩은 무엇에 관한 것인가?
2. 소스: 원본 자료가 어디 있는가? URL, 파일, 법령명, 텍스트 등.
3. 목적: 이 팩으로 어떤 질문에 답해야 하는가?

## Domain Detection

Choose the best template from the topic and source, then confirm once with the user: `감지된 도메인: [domain] - 맞나요?`

| Domain | Signals | Node types | Chunk unit |
| --- | --- | --- | --- |
| legal | 법, 법률, 조례, 규정, law | Law -> Chapter -> Article | One article |
| technical | 코드, API, SDK, 문서, GitHub | Module -> Function -> Class | Function or section |
| news | 뉴스, 기사, media | Article -> Person -> Event | Paragraph |
| academic | 논문, 연구, paper | Paper -> Author -> Finding | Section |
| business | 스타트업, 기업, 투자 | Company -> Founder -> Product | Section |
| government | 정부, 정책, 행정 | Policy -> Ministry -> Program | Section |

## Source Collection

Collect every source unit before building nodes. Do not summarize or sample the source.

For Korean laws, use the KoreanLaw MCP tools when available:

1. Search the law by name and identify the `mst`.
2. Fetch the full law text or table of contents.
3. Fetch each article individually when the API supports it.
4. Create one chunk per article with the complete original article text.

For URLs, fetch the page and split by semantic section or paragraph. For local files, read the file and split by its natural structure such as chapter, section, function, or article. Every source article, section, or paragraph must become a chunk.

## LocalCrab Build Workflow

Discover the LocalCrab/OpenCrab MCP tools if they are not already available. Always start with the ontology manifest and use only spaces, node types, and relations that the manifest supports.

1. Run the ontology manifest operation first.
2. Add hierarchy nodes before leaf nodes, such as `Law -> Chapter -> Article`.
3. Give every node a stable unique `node_id`.
4. Put the full original source text in a `content` property on the source-bearing node.
5. Add edges only after both endpoint nodes exist.
6. Ingest every chunk into Evidence with metadata linking it back to the node.

Node properties should include source identifiers such as law name, article number, title, section, URL, file path, and `chunk_id` whenever applicable.

Use relations such as `PART_OF`, `CONTAINS`, `HAS_KEYWORD`, `REFERENCES`, or `AMENDS` only when they exist in the manifest. Never create an edge to a missing node.

## Quality Gate

Before promotion, verify and fix all of the following:

- Every source article, section, or paragraph exists as a node or source chunk.
- Every source-bearing node has full original `content`.
- Every chunk was ingested into Evidence.
- No broken edge references a missing source or target node.
- No orphan node remains unintentionally disconnected.
- The pack can answer the stated user purpose from Evidence-backed content.

Do not promote until the quality gate passes. If a check fails, repair the graph or Evidence ingestion and run the relevant query/check again.

## Promotion

After the quality gate passes, register promotion candidates, validate them, and promote them using the available LocalCrab promotion operations. Use `confidence=0.95` and `source_id="crab_skill"` unless the task gives a stronger source-specific value.

## Completion Report

Report in Korean when the user is working in Korean. Include this structure:

```text
온톨로지 팩 빌딩 완료

도메인: [legal/technical/news/academic/business/government]
주제: [topic]

통계
- 노드: X개 (type별 개수)
- 엣지: Y개
- 인제스트된 청크: Z개 (Evidence indexed)
- 누락 단위: 0개

품질
- broken edges: 0
- 고아 노드: 0
- 모든 노드 Evidence 연결: 통과
```

## Absolute Rules

1. Preserve original text. Never summarize, invent, or use representative samples in place of source chunks.
2. Keep chunks atomic. For legal sources, one article equals one chunk.
3. Ingest every chunk into Evidence.
4. Add nodes before edges.
5. Use only source-backed facts. Do not fill gaps with AI-generated content.
6. Never skip the quality gate.
