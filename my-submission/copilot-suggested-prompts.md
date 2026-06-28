# Copilot-Suggested Prompt Sequence (Turn 1–4)

These prompts were suggested by Copilot during Codespace setup. Use alongside `prompt-sequence.md`.

---

## Turn 1 — First prompt

```text
I'm taking the role of Zava's Principal Data Scientist. I want to study customer themes across reviews and support chats using the SQL MCP tools and the vector similarity tool. Start by discovering the available entities and the schema, then tell me the most relevant tables and fields for this task.

Please:
1. Use the SQL MCP tools to inspect the available entities and schema.
2. Identify the most relevant tables and fields for analyzing multilingual customer themes.
3. Explain why each table or field is relevant.
4. Keep the answer grounded in the actual tool output rather than general assumptions.
5. If anything is unclear, say so explicitly and suggest the next best evidence-gathering step.
```

## Turn 1 — Shorter variant (also valid)

```text
I'm taking the role of Zava's Principal Data Scientist. I want to study customer themes across reviews and support chats using the SQL MCP tools and the vector similarity tool. Start by discovering the available entities and the schema, then tell me the most relevant tables and fields for this task.
```

---

## Turn 2 — Corpus characterisation before hypothesis

```text
Before forming any hypothesis, show me counts or summaries that describe the corpus. I care about document volume and the mix of sources, languages, and categories so I can judge whether the dataset is sufficient.
```

---

## Turn 3 — Seed selection + first vector search

```text
Pick one seed document in English and one in Spanish that look representative of the corpus. Use the similarity tool to retrieve nearby documents for each, and explain what themes appear to emerge before I ask you to evaluate quality.
```

---

## Turn 4 — Overclaiming correction (paste if agent over-asserts)

```text
Be careful not to overclaim. If a cluster or theme looks plausible but is not well supported, say that explicitly and note what evidence is still missing.
```

---

## Why this sequence works

- Forces schema discovery before any analysis
- Emphasises evidence before hypothesis
- Creates a natural path to multilingual retrieval
- Leaves room for the "agent overclaimed" moment the rubric rewards
- Turn 4 correction prompt is submission gold — paste it when the agent gets overconfident
