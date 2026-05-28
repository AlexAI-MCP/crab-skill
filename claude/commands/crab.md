# /crab — Crab Skill

LocalCrab MCP를 사용해서 고품질 온톨로지 팩을 만드는 스킬이다.
이 스킬은 초보자도 누락 없이 올바른 순서로 팩을 만들 수 있도록 안내한다.

---

## Step 1 — 3가지만 물어봐라

$ARGUMENTS에 이미 내용이 있으면 바로 Step 2로 간다.
없으면 딱 3개만 물어봐라. 더 이상은 묻지 마라.

1. **주제**: 이 팩은 무엇에 관한 것인가?
2. **소스**: 원본 자료가 어디 있는가? (URL, 파일, 법령명, 텍스트 등)
3. **목적**: 이 팩으로 어떤 질문에 답해야 하는가?

---

## Step 2 — 도메인 템플릿 자동 판단

주제와 소스를 보고 아래 중 하나를 선택한다:

| 도메인 | 판단 기준 | 노드 타입 | 청킹 단위 |
|--------|----------|-----------|-----------|
| **legal** | 법, 법률, 조례, 규정, law | Law → Chapter → Article | 조문 1개 = 청크 1개 |
| **technical** | 코드, API, SDK, 문서, github | Module → Function → Class | 함수/섹션 단위 |
| **news** | 뉴스, 기사, 미디어 | Article → Person → Event | 문단 단위 |
| **academic** | 논문, 연구, paper | Paper → Author → Finding | 섹션 단위 |
| **business** | 스타트업, 기업, 투자 | Company → Founder → Product | 섹션 단위 |
| **government** | 정부, 정책, 행정 | Policy → Ministry → Program | 섹션 단위 |

사용자에게 "감지된 도메인: [X] — 맞나요?" 확인 후 진행한다.

---

## Step 3 — 소스 수집 및 파싱

소스 타입별 수집 방법:

**법령 (KoreanLaw MCP)**
```
1. mcp__KoreanLaw__search_law(query="법령명") → mst 확인
2. mcp__KoreanLaw__get_law_text(mst="...") → 전체 조문 목차 확인
3. 각 조문: mcp__KoreanLaw__get_law_text(mst="...", jo="제N조") → 조문 전문
   → 조문 1개당 청크 1개 생성. 절대 요약하지 마라. 원문 전체를 청크에 담아라.
```

**URL / 웹페이지**
```
1. WebFetch로 페이지 수집
2. 섹션/문단 단위로 분리
3. 각 섹션 = 청크 1개
```

**로컬 파일**
```
1. Read 도구로 파일 읽기
2. 구조 파악 (챕터/섹션/조문)
3. 최소 단위로 청킹
```

**누락 금지 원칙**: 소스에 있는 내용이 청크에서 빠지면 안 된다.
요약이나 대표 샘플만 뽑는 것은 금지다.

---

## Step 4 — LocalCrab MCP로 온톨로지 빌딩

### 4-1. 그래머 확인 (첫 번째로 반드시 실행)
```
mcp__LocalCrab__ontology_manifest()
```
→ 사용 가능한 space, node type, relation 목록 확인

### 4-2. 노드 추가
```
mcp__LocalCrab__ontology_add_node(
  space="...",
  node_type="Article",          ← manifest에서 확인한 타입 사용
  node_id="article_저작권법_1",
  properties={
    "label": "제1조",
    "law": "저작권법",
    "chapter": "총칙",
    "article_no": "1",
    "title": "목적",
    "content": "...",           ← 원문 전체
    "chunk_id": "chunk_저작권법_1"
  }
)
```

노드 추가 규칙:
- 계층 구조대로 만들어라: Law → Chapter → Article 순서
- node_id는 중복 없이 고유하게
- content 필드에 원문 전체를 넣어라 (요약 금지)

### 4-3. 엣지 추가
```
mcp__LocalCrab__ontology_add_edge(
  space="...",
  source_id="chapter_저작권법_총칙",
  target_id="law_저작권법",
  relation="PART_OF"
)
```

엣지 추가 규칙:
- 존재하는 노드 ID만 참조 (없는 노드 참조 = broken edge)
- PART_OF, CONTAINS, HAS_KEYWORD, REFERENCES, AMENDS 등 적절한 relation 사용

### 4-4. 인제스트 (Evidence 인덱싱)
청킹된 내용을 evidence로 인덱싱:
```
mcp__LocalCrab__ontology_ingest(
  content="【저작권법 제1조】 목적\n이 법은...",
  metadata={
    "law": "저작권법",
    "article_no": "1",
    "chapter": "총칙",
    "node_id": "article_저작권법_1"
  }
)
```
→ 모든 청크는 반드시 인제스트해야 한다. 인제스트 = Evidence로 인덱싱됨

---

## Step 5 — 품질 확인

빌딩 완료 후 아래를 체크한다:

```
mcp__LocalCrab__ontology_query(query="전체 노드 목록")
```

체크 항목:
- [ ] 소스의 모든 조문/섹션이 노드로 존재하는가?
- [ ] 모든 노드에 content가 있는가?
- [ ] 모든 청크가 인제스트(Evidence) 되었는가?
- [ ] broken edge가 없는가? (source/target 노드가 실제로 존재하는가?)
- [ ] 고아 노드(연결 없는 노드)가 없는가?

문제가 있으면 고치고 다시 확인한다. 통과 전까지 다음 단계로 넘어가지 마라.

---

## Step 6 — 프로모션

품질 확인 통과 후:
```
mcp__LocalCrab__promotion_register_candidate(
  space="...",
  node_type="...",
  node_id="...",
  properties={...},
  confidence=0.95,
  source_id="crab_skill"
)
```

```
mcp__LocalCrab__promotion_validate_candidate(...)
mcp__LocalCrab__promotion_promote(...)
```

---

## Step 7 — 결과 보고

완료되면 반드시 이 형식으로 보고한다:

```
✅ 온톨로지 팩 빌딩 완료

도메인: [legal/technical/...]
주제: [사용자 입력]

📊 통계
- 노드: X개 (Law: N, Chapter: N, Article: N, Keyword: N)
- 엣지: Y개
- 인제스트된 청크: Z개 (Evidence indexed)
- 누락 조문: 0개

🔍 품질
- broken edges: 0
- 고아 노드: 0
- 모든 노드 evidence 연결: ✓
```

---

## 핵심 원칙 (절대 위반 금지)

1. **원문 보존** — 요약하거나 대표 샘플만 뽑지 마라. 모든 조문/섹션이 들어가야 한다.
2. **청크 단위** — 조문 1개 = 청크 1개. 여러 조문을 하나로 묶지 마라.
3. **Evidence 인덱싱** — 모든 청크는 `ontology_ingest`로 인제스트해야 한다.
4. **노드 우선** — 엣지 추가 전에 반드시 양쪽 노드가 먼저 존재해야 한다.
5. **AI 추가 금지** — 소스에 없는 내용을 AI가 생성해서 채우지 마라.
6. **품질 게이트 스킵 금지** — Step 5 확인 없이 프로모션하지 마라.

$ARGUMENTS
