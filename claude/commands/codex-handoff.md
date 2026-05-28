# Crab Skill — Codex Handoff Prompt

**전역 설치 경로:** `~/.claude/commands/crab.md` (Claude Code 어느 폴더에서든 `/crab` 사용 가능)

아래를 Codex system prompt 또는 AGENTS.md에 추가하세요.

---

## [SYSTEM PROMPT — Crab Skill]

You have access to the **Crab Skill** — a workflow skill for creating high-quality LocalCrab ontology packs using LocalCrab MCP tools.

This is NOT a new MCP tool. It is a set of instructions that tells you how to correctly orchestrate the existing LocalCrab MCP tools to produce packs that pass quality gates.

---

### When to activate

When the user asks to:
- "팩 만들어줘" / "온톨로지 빌딩해줘"
- Create an ontology pack from any source material
- Use LocalCrab to index laws, documents, code, etc.

---

### Step 1 — Ask exactly 3 questions (skip if already provided)

1. **주제 (Topic)**: 이 팩은 무엇에 관한 것인가?
2. **소스 (Source)**: 원본 자료가 어디 있는가? (URL, 파일, 법령명 등)
3. **목적 (Purpose)**: 어떤 질문에 답해야 하는가?

---

### Step 2 — Detect domain template

| Domain | Keywords | Node types | Chunk unit |
|--------|----------|-----------|------------|
| legal | 법, 법률, 조례, law | Law→Chapter→Article | Per article |
| technical | 코드, API, github | Module→Function→Class | Per function/section |
| news | 뉴스, article | Article→Person→Event | Per paragraph |
| academic | 논문, paper | Paper→Author→Finding | Per section |
| business | 스타트업, company | Company→Founder→Product | Per section |
| government | 정부, 정책, policy | Policy→Ministry→Program | Per section |

---

### Step 3 — Collect and parse source material

**Korean laws** — Use KoreanLaw MCP:
```
search_law(query="법령명") → get mst
get_law_text(mst="...") → full article list
get_law_text(mst="...", jo="제N조") → full text per article
```
→ Each article = one chunk. Include full original text, no summaries.

**URLs** — Fetch and split by section/paragraph. Each section = one chunk.

**Files** — Read and split by structure. Minimum unit = one chunk.

**Zero omission**: every article/section in the source must become a chunk.

---

### Step 4 — Build ontology with LocalCrab MCP

**Always start with:**
```
ontology_manifest() → check available spaces, node types, relations
```

**Add nodes** (hierarchy first: Law → Chapter → Article):
```
ontology_add_node(
  space="...",
  node_type="Article",
  node_id="article_저작권법_1",
  properties={
    "label": "제1조",
    "law": "저작권법",
    "article_no": "1",
    "title": "목적",
    "content": "<full original text>"  ← never summarize
  }
)
```

**Add edges** (only after both nodes exist):
```
ontology_add_edge(space, source_id, target_id, relation="PART_OF")
```

**Ingest all chunks as Evidence:**
```
ontology_ingest(
  content="<full article text>",
  metadata={"law": "...", "article_no": "...", "node_id": "..."}
)
```
Every chunk must be ingested. Ingest = Evidence indexed.

---

### Step 5 — Quality check (do not skip)

Verify before promoting:
- All articles/sections from source → nodes ✓
- All nodes have content field ✓
- All chunks ingested (Evidence) ✓
- No broken edges (source/target nodes exist) ✓
- No orphan nodes ✓

Fix issues, re-check. Do not promote until all checks pass.

---

### Step 6 — Promote

```
promotion_register_candidate(space, node_type, node_id, properties, confidence=0.95, source_id="crab_skill")
promotion_validate_candidate(...)
promotion_promote(...)
```

---

### Step 7 — Report

```
✅ 온톨로지 팩 빌딩 완료

도메인: legal
주제: 저작권법

📊 통계
- 노드: X개 (Law: N, Chapter: N, Article: N)
- 엣지: Y개
- 인제스트 청크: Z개 (Evidence indexed)
- 누락 조문: 0개

🔍 품질
- broken edges: 0 ✓
- 고아 노드: 0 ✓
- Evidence 연결: 전체 ✓
```

---

### Absolute rules

1. **Never summarize** — full original text in every chunk
2. **One article = one chunk** — never merge multiple articles
3. **Ingest every chunk** — `ontology_ingest` for all content
4. **Nodes before edges** — both endpoints must exist before adding edge
5. **No AI-generated content** — only what's in the source
6. **Never skip quality check** — Step 5 is mandatory

---

*Crab Skill — guides LocalCrab MCP usage for high-quality pack creation*
