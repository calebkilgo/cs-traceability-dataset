# cs-traceability-dataset

Requirement-to-function traceability dataset for NASA's Core Flight System Checksum (CS) application, prepared for dual-encoder retrieval training with BM25-mined hard negatives.

## Contents

```
data/
├── corpus.csv              65 functions (fn_id, fn_text)
├── queries.csv             114 requirements (req_id, req_text)
├── positives_{train,val,test}.csv   gold (req_id, fn_id) pairs per split
├── triplets_{train,val,test}.csv    (req_id, pos_fn_id, neg_fn_id) for training
└── stats.json              dataset statistics
```

| Split | Requirements | Positives | Triplets | CS Groups |
|---|---|---|---|---|
| Train | 94 | 259 | 518 | 1, 2, 3, 5, 6, 7, 9 |
| Val   | 13 | 17  | 34  | 4 |
| Test  | 7  | 8   | 16  | 8 |

## File formats

- **corpus.csv**: `fn_id, fn_text` — the function name and its production code block. Test code is excluded; production functions are the only valid trace targets.
- **queries.csv**: `req_id, req_text` — the requirement ID and its full description text.
- **positives_*.csv**: `req_id, fn_id` — ground-truth trace links. A pair appears here if the function implements that requirement.
- **triplets_*.csv**: `req_id, pos_fn_id, neg_fn_id` — anchor, positive, and hard negative for contrastive training. Each positive pair produces 2 triplets (one per mined hard negative).

## Hard negative mining

For each requirement, we mine 2 hard negatives using BM25 over the function corpus:

1. Tokenize each function's production code block.
2. Build a BM25Okapi index over all 65 functions.
3. For each requirement, score every function with BM25 against the requirement text.
4. Take the top-ranked functions that are not in that requirement's positive set.

BM25 selects functions that look lexically relevant to the requirement but are not real trace targets, forcing the model to learn semantic distinctions beyond simple word overlap. This is the recipe from Dense Passage Retrieval [1]. We use 2 hard negatives per positive instead of DPR's 1, since our corpus is only 65 functions and each additional mined negative adds proportionally more signal.

## Splits

We use `GroupShuffleSplit` over the CS group prefix (CS1–CS9) so no requirement family appears in more than one split. This prevents leakage: requirements within a group share vocabulary and target similar function clusters, and a random split would let the model see the test distribution at training time. This grouped strategy is standard for small traceability datasets [2].

## References

[1] Karpukhin, V. et al. *Dense Passage Retrieval for Open-Domain Question Answering.* EMNLP 2020. https://arxiv.org/abs/2004.04906

[2] Cleland-Huang, J., Rahimi, M., Mirakhorli, M. *Automating Requirements Traceability: Two Decades of Learning from KDD.* 2018. https://arxiv.org/abs/1807.11454