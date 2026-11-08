---
name: structure-annotation-rank-benchmark-evaluation
description: Use when benchmarking a candidate-ranking score for MS/MS structure annotation
  against known-correct structures (e.g., CASMI 2016 contest data), to quantify separation
  via AUC, FDR-level hit counts, and mean rank of correct hits.
when_to_use_negative:
- Input is raw LC–MS/MS data (mzML/mzXML) requiring SIRIUS/CSI:FingerID candidate
  generation and scoring rather than evaluation of already-published hit scores
- There is no known correct structure for the queries (e.g., biological datasets),
  since the rank benchmark requires ground truth to compute ROC, FDR-level hit counts,
  and mean rank
- The goal is de novo annotation of unseen compounds rather than quantifying how well
  an existing score separates correct from incorrect annotations
tools:
- name: COSMIC
  role: Provides the confidence score whose separation performance is benchmarked
    against contest-method hit scores
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Structure candidate ranking method whose hit scores serve as a baseline in
    the evaluation
  repo: https://github.com/sirius-ms/sirius
- name: MetFrag
  role: Baseline in silico tool whose per-compound hit scores are downloaded from
    the CASMI 2016 results web page
- name: MAGMa+
  role: Baseline in silico tool whose per-compound hit scores are downloaded from
    the CASMI 2016 results web page
- name: CFM-ID
  role: Baseline in silico tool whose per-compound hit scores are downloaded from
    the CASMI 2016 results web page
- name: CSI:FingerID IOKR
  role: Baseline variant of CSI:FingerID whose hit scores are downloaded from the
    CASMI 2016 results web page
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:3da0a676c169444e96ca825d0689bbdee5f817e5402af8089c398f27ccf0079e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CFM-ID
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID IOKR
  - MAGMa+
  - MetFrag
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# structure-annotation-rank-benchmark-evaluation

## Summary

Evaluates whether a structure-annotation score (e.g., COSMIC confidence score vs in silico tool hit scores) actually separates correct from incorrect candidate annotations, using rank-based metrics (ROC AUC, hop plots, correct-hit counts at FDR levels) on benchmark data such as CASMI 2016.

## When to use

Use when benchmarking a candidate-ranking score for MS/MS structure annotation against known-correct structures (e.g., CASMI 2016 contest data), to quantify separation via AUC, FDR-level hit counts, and mean rank of correct hits.

## When NOT to use

- Input is raw LC–MS/MS data (mzML/mzXML) requiring SIRIUS/CSI:FingerID candidate generation and scoring rather than evaluation of already-published hit scores
- There is no known correct structure for the queries (e.g., biological datasets), since the rank benchmark requires ground truth to compute ROC, FDR-level hit counts, and mean rank
- The goal is de novo annotation of unseen compounds rather than quantifying how well an existing score separates correct from incorrect annotations

## Inputs

- Per-compound hit scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR from the CASMI 2016 results web page
- COSMIC confidence scores for 127 positive-ion Orbitrap standard compounds
- Known correct structures for the CASMI 2016 queries
- Candidate structure lists from the biomolecule structure database (123 queries) and ChemSpider (127 queries)

## Outputs

- casmi2016_roc_hop_results.csv with AUC, correct-hit counts at 10% and 0% FDR, and mean ranks per method
- ROC curves exported as PNG figures
- Hop plots exported as PNG figures

## How to apply

Download the published per-compound hit scores of the baseline methods (MetFrag, MAGMa+, CFM-ID, CSI:FingerID original, and CSI:FingerID IOKR) from the CASMI 2016 results web page, and load the COSMIC confidence scores computed for the 127 positive-ion Orbitrap standard compounds. For each method and compound, rank the candidate structures against the known correct structure and compute ROC curves and hop plots. Report the area under the ROC curve per score (expected contest-method baselines 0.40–0.55 versus COSMIC at ~0.82) and count correct hits at 10% and 0% FDR (57 hits with the biomolecule structure database over 123 queries; 16 hits with ChemSpider over 127 queries at 0% FDR). Also compute the average rank of correct hits per method to quantify the rank improvement from 56.5 to 33.9. Write all metrics (AUC, correct-hit counts at both FDR levels, mean ranks) to casmi2016_roc_hop_results.csv and export ROC and hop plots as PNG figures.

## Related tools

- **COSMIC** (Provides the confidence score whose separation performance is benchmarked against contest-method hit scores) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Structure candidate ranking method whose hit scores serve as a baseline in the evaluation) — https://github.com/sirius-ms/sirius
- **MetFrag** (Baseline in silico tool whose per-compound hit scores are downloaded from the CASMI 2016 results web page)
- **MAGMa+** (Baseline in silico tool whose per-compound hit scores are downloaded from the CASMI 2016 results web page)
- **CFM-ID** (Baseline in silico tool whose per-compound hit scores are downloaded from the CASMI 2016 results web page)
- **CSI:FingerID IOKR** (Baseline variant of CSI:FingerID whose hit scores are downloaded from the CASMI 2016 results web page)

## Examples

```
from sklearn.metrics import roc_auc_score; auc = roc_auc_score(y_true, scores)  # y_true: correct-structure indicator per ranked candidate, scores: hit/confidence scores
```

## Evaluation signals

- COSMIC AUC (~0.82) exceeds the contest-method baselines (expected 0.40–0.55)
- Correct-hit counts at 10% FDR equal 57 with the biomolecule structure database (123 queries)
- Correct-hit counts at 0% FDR equal 16 with ChemSpider (127 queries)
- Mean rank of correct hits improves from 56.5 to 33.9 with the confidence score
- Output CSV contains AUC, correct-hit counts at both FDR levels, and mean ranks for every evaluated method

## Limitations

- Confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- The confidence-score threshold of 0.64 roughly corresponding to FDR 10% is practical guidance only, with no guarantee in either direction
- Evaluations rely on reference measurements of standard compounds, as we do not know the correct answers for biological datasets
- Cannot guarantee a similar FDR for structure annotations on biological data given the inability to accurately estimate FDR from small-molecule fragmentation spectra

## Evidence

- [abstract] COSMIC annotates a substantial number of hits at low false discovery rates and outperforms spectral library search: "COSMIC annotates a substantial number of hits at low false discovery rates and outperforms spectral library search"
- [discussion] Scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR were downloaded from the CASMI 2016 results web page: "Scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR were downloaded from the CASMI 2016 results web page"
- [discussion] FDR estimation from confidence scores: "we can use the confidence score to estimate the FDR of the top k hits"
- [readme] SIRIUS framework description: [evidence span withheld — anchor status: external_doc]
