---
name: tanimoto-similarity-performance-comparison
description: Use when you have generated/predicted molecule SMILES paired with true
  SMILES and need a structure-aware metric (Tanimoto via Morgan fingerprints) to quantify
  performance gaps between conditions, datasets, or retrievers.
when_to_use_negative:
- The metric being compared is unavailable for a baseline — MSNovelist results are
  limited to accuracy metrics, as similarity and MCES measures are denoted by '-'
- Input is raw MS/MS spectra rather than generated molecule outputs — this skill operates
  on post-generation SMILES comparison, not spectral preprocessing
- You need scaffold retrieval ranking itself (contrastive scoring of candidate scaffolds)
  rather than post-generation structure-similarity evaluation
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: RDKit
  role: Constructs Morgan fingerprints (radius=2, 2048 bits) used to compute Tanimoto
    similarity between true and predicted molecules
  repo: https://github.com/HassounLab/MADGEN
- name: MADGEN
  role: Generates the de novo molecular structures under oracle and predictive retriever
    conditions whose outputs are evaluated
  repo: https://github.com/HassounLab/MADGEN
- name: Python
  role: Runs the evaluation script and the gap computation/joining of per-dataset
    metrics with SPA values
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:849b263affe8d864a4a04d70a25ad3c447a58e8bb803a5e17833d69c2446badd
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - MADGEN
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# tanimoto-similarity-performance-comparison

## Summary

Computes Tanimoto similarity between true and generated molecules using Morgan fingerprints (radius=2, 2048 bits) and combines it with Top-K accuracy to compare performance across experimental conditions (e.g., oracle vs. predictive scaffold retriever) on NIST23, CANOPUS, and MassSpecGym.

## When to use

Use when you have generated/predicted molecule SMILES paired with true SMILES and need a structure-aware metric (Tanimoto via Morgan fingerprints) to quantify performance gaps between conditions, datasets, or retrievers.

## When NOT to use

- The metric being compared is unavailable for a baseline — MSNovelist results are limited to accuracy metrics, as similarity and MCES measures are denoted by '-'
- Input is raw MS/MS spectra rather than generated molecule outputs — this skill operates on post-generation SMILES comparison, not spectral preprocessing
- You need scaffold retrieval ranking itself (contrastive scoring of candidate scaffolds) rather than post-generation structure-similarity evaluation

## Inputs

- CSV file of generation results with columns 'true' (true SMILES), 'pred' (predicted SMILES, repeated for multiple samples), and 'scaffold' (scaffold SMILES)
- Per-condition metric tables for oracle-retriever and predictive-retriever runs on NIST23, CANOPUS, and MassSpecGym
- Scaffold prediction accuracy (SPA) values for the predictive scaffold retriever per dataset

## Outputs

- Per-condition evaluation metrics: Top-K accuracy (InChI-key matching) and Maximum Tanimoto similarity (Morgan fingerprints, radius=2, 2048 bits)
- CSV table with columns dataset, metric, oracle value, predictive value, gap, and SPA
- Interpretation of whether lower SPA corresponds to larger oracle-vs-predictive gaps

## How to apply

Generate molecules under each condition (e.g., oracle-retriever and predictive-retriever MADGEN runs) and collect them in a CSV with columns 'true', 'pred', and 'scaffold'. Run the MADGEN evaluation script, which computes Top-K accuracy by InChI-key matching and Maximum Tanimoto similarity using Morgan fingerprints with radius=2 and 2048 bits. For the condition-comparison use case, compute per-dataset metric values for each condition, then subtract to obtain the oracle-minus-predictive gap and join each gap with the retriever's scaffold prediction accuracy (SPA; e.g., 40.3%/17.7%/13.2% on NIST23/CANOPUS/MassSpecGym). Write the joined table as CSV with columns dataset, metric, oracle value, predictive value, gap, and SPA. Validate by checking that tabulated values reproduce the per-condition metric tables and that the gap-vs-SPA ordering matches the paper's finding that larger gaps occur where SPA is highest (NIST23).

## Related tools

- **RDKit** (Constructs Morgan fingerprints (radius=2, 2048 bits) used to compute Tanimoto similarity between true and predicted molecules) — https://github.com/HassounLab/MADGEN
- **MADGEN** (Generates the de novo molecular structures under oracle and predictive retriever conditions whose outputs are evaluated) — https://github.com/HassounLab/MADGEN
- **Python** (Runs the evaluation script and the gap computation/joining of per-dataset metrics with SPA values) — https://github.com/HassounLab/MADGEN

## Examples

```
python evaluation_generation.py --file_path /path/to/your/csvfile.csv
```

## Evaluation signals

- Tabulated oracle and predictive values must match the reproduced per-condition metric tables
- Oracle-vs-predictive gaps should be approximately 44.4/16.6/9.2 percentage points on NIST23/CANOPUS/MassSpecGym for Top-1 accuracy
- Gap-vs-SPA relationship must be qualitatively consistent with the paper's two-retriever evaluation (expert review)
- CSV output must contain exactly the columns dataset, metric, oracle value, predictive value, gap, and SPA
- Predictive top-1 accuracy values should fall near 4.6%/2.10%/1.31% (NIST23/CANOPUS/MassSpecGym) versus oracle 49.0%/18.7%/10.5%

## Limitations

- MSNovelist baseline cannot be included in Tanimoto-similarity comparisons since its results are limited to accuracy metrics, with other measures not available (denoted '-')
- NIST23 is commercially available and requires separate licensing; it is not distributed via the Zenodo data repository (records 15036069/17115068), which covers only msgym.pkl and canopus.pkl
- Tanimoto similarity is reported as maximum similarity against top-K predictions, so it is sensitive to the chosen K and the number of samples generated

## Evidence
