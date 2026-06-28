# Data Scientist Mission — Copilot Prompt Sequence

**Mission:** Semantic theme discovery + retrieval quality audit over Zava customer feedback  
**Artifact to produce:** Jupyter notebook with precision@k audit, labeled pairs, model card  
**Deadline:** July 24, 2026

---

## Pre-flight checklist (do before Turn 1)

- [ ] Codespace is fully loaded (SQL Server container is running)
- [ ] Copilot Chat panel is open in VS Code
- [ ] Agent mode is selected (not "Ask" or "Edit" — must be **Agent**)
- [ ] Model: pick GPT-4o or Claude Sonnet if available — note the model name for submission
- [ ] Open a new `.ipynb` notebook file alongside the chat: `my-submission/voice-of-customer.ipynb`

---

## Turn 1 — Schema discovery (no repo files)

**Paste this exactly:**

```
You are a principal data scientist on Zava's team. Your goal today is to build a semantic theme-discovery and retrieval-quality study over Zava's multilingual customer feedback.

Before you do anything else: use the SQL MCP tools to discover what entities and tools you have access to. Do not read any repository documentation files — derive everything from live tool calls. List every entity you can query and every tool you can call.
```

**What to capture from the response:**
- Full `describe_entities` output (paste into notebook cell as a markdown comment)
- Confirm you see: Products, Customers, SalesOrders, SalesOrderLines, SupportTickets, SupportChats, Docs, FindSimilarDocsByDocId

**Likely agent behaviour:** It will call `describe_entities` and list 8 entities. Good. If it starts describing the schema from memory or repo files, correct it.

**Correction prompt if needed:**
```
Stop. Do not use the repository files. Call describe_entities via the SQL MCP tool and paste the actual tool response.
```

---

## Turn 2 — Understand the Docs corpus before forming any hypothesis

**Paste this exactly:**

```
Now focus on the Docs table. I want to understand the corpus before forming any hypotheses.

Use read_records and aggregate_records to answer:
1. How many documents are there in total?
2. What is the split between SourceType = 'Review' and SourceType = 'SupportChat'?
3. What languages appear in the data? (check TagsJson for a language field)
4. What is the range of DocId values?

Do not summarize or interpret yet — just return the raw counts and a sample of 3 Docs rows showing DocId, SourceType, Title, Body (first 200 chars), and TagsJson.
```

**What to capture:**
- Total doc count (should be ~83)
- Review vs SupportChat split
- Language distribution (EN / ES / FR)
- 3 sample rows with their DocIds

**Likely pivot moment:** If TagsJson language field isn't obvious from a basic read, the agent may need to be told to look inside the JSON. Note this in your "where the agent struggled" section.

**Correction prompt if needed:**
```
The language tag is inside TagsJson as a JSON string. Parse one sample TagsJson value and show me the keys available inside it.
```

---

## Turn 3 — Sample one document per language as seeds

**Paste this exactly:**

```
Using the language distribution you found, select exactly one document per language (English, Spanish, French) from SourceType = 'Review'. Choose documents with substantive Body text, not short or empty ones.

For each of the 3 seed documents, return:
- DocId
- Title
- Full Body text
- TagsJson (full value)
- RelatedCustomerId, RelatedOrderId, RelatedTicketId (even if null)

These will be our theme seeds. Do not run vector search yet.
```

**What to capture:**
- 3 seed DocIds (write these down — you'll use them in Turns 4–6)
- Note which language is richest in text (likely English)

---

## Turn 4 — First vector search: English seed → surface cross-language neighbours

**Paste this (replace `[ENGLISH_DOC_ID]` with the English DocId from Turn 3):**

```
Run the vector similarity tool find_similar_docs_by_doc_id with @DocId = [ENGLISH_DOC_ID] and @TopN = 10.

For each returned document, also call read_records on Docs to get: DocId, SourceType, Title, Body (first 300 chars), TagsJson.

Present results as a table with: Rank | DocId | SourceType | CosineDistance | Language (from TagsJson) | Short body excerpt.

Do not interpret themes yet. Just show me the raw neighbours.
```

**What to capture:**
- Full results table (paste into notebook)
- Note whether the neighbours cross SourceType (Reviews mixing with SupportChats) — this is the "feature or trap" the mission mentions
- Note cross-language neighbours — this is the core value of semantic search over keyword search

---

## Turn 5 — Second vector search: Spanish seed → check if themes cross language

**Paste this (replace `[SPANISH_DOC_ID]` with the Spanish DocId from Turn 3):**

```
Now run find_similar_docs_by_doc_id with @DocId = [SPANISH_DOC_ID] and @TopN = 10.

Same output format as before: Rank | DocId | SourceType | CosineDistance | Language | Short body excerpt.

Then compare the neighbour sets from Turn 4 and this turn. Do any DocIds appear in both neighbour lists? If so, those documents may represent a cross-language theme cluster. Name what you think the shared theme might be, and flag it as a hypothesis not a finding.
```

**What to capture:**
- Second results table
- Any overlapping DocIds between Turn 4 and Turn 5 results
- The agent's tentative theme hypothesis

**Important:** If the agent states the theme as a confident fact, correct it:
```
That is a hypothesis based on vector proximity. Do not state it as a confirmed theme until we hand-label the neighbour pairs in the next step.
```
*(Save this exchange — it counts as "where the agent overclaimed" for your submission.)*

---

## Turn 6 — Hand-label neighbour pairs for precision@k audit

**Paste this exactly (fill in DocIds from your Turn 4 + Turn 5 tables):**

```
We are going to do a retrieval quality audit. I will hand-label the top-5 neighbours from Turn 4.

For each of the 5 neighbours, fetch the full Body text from Docs using read_records. Present them to me one at a time so I can label each as:
- RELEVANT: same theme or complaint as the seed document
- PARTIAL: related topic but different complaint or context  
- IRRELEVANT: shares vocabulary but not meaning (false positive)

Start with Rank 1. Show me:
1. The seed document body (first 400 chars)
2. The neighbour body (first 400 chars)
3. The cosine distance
4. Ask me for my label before moving to Rank 2.
```

**What to do for each pair:**
- Read both bodies
- Assign RELEVANT / PARTIAL / IRRELEVANT
- Note WHY for the ones labelled IRRELEVANT — these are your false positives

**After labelling all 5, continue with:**

```
Now calculate precision@5 for this seed document: what fraction of the top-5 neighbours did I label as RELEVANT?

Then show me precision@3 as well. Present as a small table.
```

**What to capture:**
- Your 5 labels with reasoning
- precision@3 and precision@5 values
- At least 1 false positive with an explanation of WHY it's a false positive

---

## Turn 7 — Characterise the false positives

**Paste this exactly (if you found false positives in Turn 6):**

```
Look at the documents I labelled IRRELEVANT in Turn 6. 

For each false positive:
1. What words or phrases do it and the seed document share?
2. What is different about the actual meaning or context?
3. Is this false positive a Review or SupportChat?
4. Would a keyword search (e.g. LIKE '%word%') have made the same mistake?

This is a retrieval failure analysis. Be specific about the text evidence — quote the exact phrases that caused the confusion.
```

**What to capture:**
- Specific shared phrases that triggered false matches
- Whether false positives cluster around a source type (all Reviews, or all SupportChats)
- This becomes your "limits of an 83-document corpus" section in the model card

---

## Turn 8 — Synthesise themes and write the model card

**Paste this exactly:**

```
Based on everything we've found across Turns 1–7, help me write two things:

**Part A — Theme summary (3–4 themes max)**
For each theme:
- Theme name (1 short phrase)
- The DocIds that ground it (from our vector search results)
- Whether it crosses language (EN/ES/FR)
- Whether it crosses SourceType (Review vs SupportChat)
- Confidence: HIGH (multiple converging documents), MEDIUM (2–3 documents), LOW (single document only)

**Part B — Model card**
A short note covering:
- What this system IS good for (1–3 bullet points)
- What this system IS NOT good for (1–3 bullet points)
- Known failure modes (from our false positive analysis)
- Corpus limitations (83 documents, multilingual but unbalanced, no ground-truth labels)
- Recommended next steps if Zava wanted to improve this

Keep both parts grounded in SQL evidence we already collected. Do not invent themes we did not find.
```

**What to capture:**
- Themes table
- Full model card text (paste into notebook as a markdown cell)

---

## Turn 9 — Generate the architecture diagram and notebook scaffold

**Paste this exactly:**

```
Generate two things:

**1. Mermaid architecture diagram** showing the full workflow we ran:
- Input: Multilingual customer Docs (Reviews + SupportChats)
- Process: Vector seed selection → find_similar_docs_by_doc_id → neighbour retrieval → hand-labelling → precision@k
- Output: Theme clusters + retrieval audit + model card
- Include the SQL tables we touched and the MCP tools we called

**2. Notebook scaffold** — write the section headers and key code cells for voice-of-customer.ipynb:
- Section 1: Corpus exploration (language split, source split)
- Section 2: Seed document selection
- Section 3: Vector similarity retrieval (calls for each seed)
- Section 4: Hand-labelling table
- Section 5: Precision@k calculation
- Section 6: False positive analysis
- Section 7: Theme summary
- Section 8: Model card

For Section 5, write the actual Python code to compute precision@k from a list of labels.
```

**What to capture:**
- Mermaid diagram code (paste into submission)
- Notebook scaffold with Python code for Section 5

---

## Turn 10 (bonus) — Stress-test the system with a hard query

**Paste this exactly:**

```
Final stress test. I want to find one case where the vector search fails badly — a seed document whose top neighbour is clearly wrong.

Scan through DocIds we haven't used yet. Pick one that seems likely to be an "outlier" — unusual language, very short body, or a niche product complaint. Run find_similar_docs_by_doc_id on it with @TopN = 5.

Show me the top neighbour and the seed side by side. If this is a genuine false positive, explain what it reveals about the limits of cosine similarity on a small corpus. If it is actually a good match, say so.

Add this case to the false positive analysis in the notebook.
```

---

## After all turns — fill the notebook

Open `voice-of-customer.ipynb` and paste in:

| Section | Source |
|---|---|
| Corpus counts table | Turn 2 output |
| Language + SourceType split | Turn 2 output |
| 3 seed documents | Turn 3 output |
| Vector results tables | Turn 4 + Turn 5 output |
| Hand-labelled pairs table | Turn 6 work |
| precision@3 and precision@5 | Turn 6 calculation |
| False positive analysis | Turn 7 output |
| Theme summary | Turn 8 Part A |
| Model card | Turn 8 Part B |
| Mermaid diagram | Turn 9 |

---

## Submission checklist

Before filing the GitHub Issue at `github.com/microsoft/sql-ai-promptathon/issues/new`:

- [ ] Goal statement: *Build a semantic theme-discovery and retrieval-quality audit over Zava's 83 multilingual customer documents*
- [ ] Agent + model name noted (from Codespace Copilot Chat header)
- [ ] SQL MCP tool call evidence pasted (at minimum: describe_entities, read_records on Docs, aggregate_records on Docs, find_similar_docs_by_doc_id x2)
- [ ] Turn-by-turn journey documented (include the overclaiming correction from Turn 5)
- [ ] Notebook committed to fork and linked
- [ ] Mermaid architecture diagram included
- [ ] Model card written
- [ ] "What to improve next" section filled in
