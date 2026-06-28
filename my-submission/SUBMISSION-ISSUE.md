# GitHub Issue — copy each field into the submission form

Form URL: https://github.com/microsoft/sql-ai-promptathon/issues/new?template=promptathon-submission.yml

---

## Title

```
Mission: Principal Data Scientist — Voice-of-Customer semantic retrieval with an honest precision audit
```

---

## Field 1 — Mission/open goal Description

```
I took the Principal Data Scientist mission: build a semantic theme-discovery system over Zava's multilingual customer feedback (89 documents spanning 44 reviews + 45 support chats, in English, Spanish, and French) and — the part that mattered most — audit how far the vector matches can actually be trusted.

The business question: Zava's leaders argue about what customers think based on cherry-picked reviews. Can precomputed embeddings + the FindSimilarDocsByDocId vector tool surface real cross-language themes, and can I quantify where that retrieval is wrong rather than just demoing that it "works"?

Goal: move from "the tool returned neighbors" to "here is the theme, here is its retrieval precision on a sample I hand-labeled, and here is exactly where and why it fails."
```

---

## Field 2 — Harness and model

```
GitHub Copilot CLI in agent mode, with SQL MCP server tools. Model selection: Auto (Copilot auto-routed; exact backing model not pinned).
```

---

## Field 3 — Turn-by-turn journey

```
1. Prompt: "As Zava's Principal Data Scientist, discover the available entities and schema using the SQL MCP tools, then tell me the most relevant tables/fields for studying multilingual customer themes. Keep it grounded in actual tool output."
   Agent action: called describe_entities + read_records.
   Result: Confirmed 8 entities + the FindSimilarDocsByDocId vector tool. Docs (with VECTOR(1536) embeddings + source/language tags) identified as the right starting point.

2. Prompt: "Before any hypothesis, show counts describing the corpus — volume, source mix, languages."
   Agent action: describe_entities + read_records aggregations.
   Result: 89 Docs (44 reviews / 45 support chats), 47 support tickets, 1,228 customers. Support-transcript languages: EN 28 / ES 13 / FR 4. English-dominant and unbalanced — noted as a limit on cross-language claims.

3. Prompt: "Pick one representative English seed and one Spanish seed, run the similarity tool for each, and describe emerging themes before evaluating quality."
   Agent action: read_records + FindSimilarDocsByDocId on DocId 2 (English, negative) and DocId 9 (Spanish, positive).
   Result: English seed → a negative quality/refund cluster (DocId 7, 21, 3). Spanish seed → a positive fit/comfort cluster (DocId 15, 25, 37). DocId 18 (an English review) appeared in BOTH neighborhoods.

4. Prompt (correction): "Don't overclaim. If a theme is plausible but not well supported, say so and name the missing evidence."
   Agent action: re-examined DocId 18.
   Result: Fetched its body — an English POSITIVE review. Diagnosed: a true cross-language match for the Spanish positive seed, but a false positive for the English negative seed (shared product/defect vocabulary, opposite sentiment). Cosine distances confirmed it: 0.2751 to the positive seed vs 0.3197 to the negative seed.

5. Prompt: "Label each DocId 2 neighbor RELEVANT/NOT RELEVANT for a negative-quality theme — relevant only if it shares both topic and negative sentiment."
   Result: 7=RELEVANT, 18=NOT RELEVANT, 21=RELEVANT, 3=RELEVANT → precision@4 = 0.75.

6. Prompt: "Label each DocId 9 neighbor for a positive fit/comfort theme."
   Result: 15, 25, 18, 37 all RELEVANT → precision@4 = 1.00.

Headline finding: the embedding separates TOPIC well but SENTIMENT poorly. The same document (DocId 18) is a correct neighbor for a positive seed and a false positive for a negative seed. Retrieval is least trustworthy exactly where the business cares most — complaints.

Full notebook with evidence, distances, labeled tables, precision@k code, architecture diagram, and model card:
https://github.com/route2sabya/sql-ai-promptathon/blob/data-scientist-submission/my-submission/voice-of-customer.ipynb
```

---

## Field 4 — Completion

- [x] Yes, the agent completed the mission or goal.

---

## Field 5 — Bonus work

```
- Quantitative false-positive proof: used cosine distances (0.2751 positive vs 0.3197 negative) to show DocId 18 objectively belongs to the positive cluster, not just by eyeballing text.
- Two-seed symmetric audit revealing an asymmetry (positive precision 1.00 vs negative 0.75) — a finding, not just a metric: negative-sentiment retrieval is the weak spot.
- A model-card style note (what the system is good for / not good for / known failure modes / corpus limits / next steps): https://github.com/route2sabya/sql-ai-promptathon/blob/data-scientist-submission/my-submission/model-card.md
- Documented the agent's overclaiming moment and the exact correction prompt that reined it in.
- Reusable prompt sequence for the mission: https://github.com/route2sabya/sql-ai-promptathon/blob/data-scientist-submission/my-submission/prompt-sequence.md
```
