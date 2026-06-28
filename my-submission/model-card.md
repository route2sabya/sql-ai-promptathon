# Model Card — Zava Voice-of-Customer Semantic Retrieval

## System summary

A semantic theme-discovery system over Zava's customer feedback corpus (89 documents spanning Reviews and SupportChats, multilingual: English, Spanish, French). It uses precomputed `VECTOR(1536)` embeddings and a cosine-distance similarity tool (`FindSimilarDocsByDocId`) to surface thematically related feedback across languages.

## What it IS good for

- **Cross-language topical retrieval.** Surfaces documents about the same product theme regardless of source language. Evidence: English review DocId 18 retrieved as a true neighbor of Spanish review DocId 9 (both "good fit/comfort + minor defect").
- **Theme discovery.** Groups feedback into recognizable clusters — product-quality disappointment, fit/comfort satisfaction, minor-defect tolerance — without manual keyword rules.
- **Bridging Reviews and SupportChats.** Operates over one shared document space, so a theme can be traced across both source types.

## What it is NOT good for

- **Sentiment-sensitive routing.** The model captures *topic* strongly but *sentiment polarity* weakly. The same document (DocId 18, a positive review) is a correct neighbor for a positive seed and a false positive for a negative seed (DocId 2, a refund-seeking complaint). Do not use raw similarity to triage by sentiment.
- **Automated decisions.** No ground-truth labels exist; precision was estimated on a hand-labeled sample only. A human must review matches before any refund/escalation action.
- **Statistical generalization.** At 89 documents, clusters are illustrative, not significant.

## Known failure modes

| Failure | Example | Root cause |
|---|---|---|
| Sentiment mismatch | DocId 18 (delighted) retrieved for DocId 2 (wants refund) | Shared product/quality/defect vocabulary outweighs opposite sentiment in embedding space |
| Asymmetric precision | Negative seed (DocId 2) = 0.75 vs positive seed (DocId 9) = 1.00 | Negative complaints share fit/quality/defect vocabulary with satisfied reviews, so positive docs leak into negative clusters |

## Measured quality

- precision@4 on English **negative** seed (DocId 2): **3/4 = 0.75** (miss: DocId 18, sentiment mismatch)
- precision@4 on Spanish **positive** seed (DocId 9): **4/4 = 1.00**
- Sample size hand-labeled: **8** neighbor pairs across 2 seeds
- **Asymmetry:** positive themes retrieve cleanly; negative themes pull in topically-similar but positive documents. Retrieval is least trustworthy exactly when the business cares most (complaints).

## Corpus limitations

- 89 documents total; language mix is unbalanced
- No ground-truth theme or sentiment labels
- Reviews and SupportChats share one space — a feature for theme breadth, a trap for source-specific questions

## Recommended next steps

- Add a sentiment dimension (separate model or metadata filter) so retrieval can distinguish polarity
- Expand the corpus and balance languages before trusting cluster sizes
- Build a small labeled gold set to measure precision@k more rigorously
- Keep all refund/escalation decisions human-in-the-loop
