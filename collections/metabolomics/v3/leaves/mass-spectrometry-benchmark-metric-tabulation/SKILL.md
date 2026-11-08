---
name: mass-spectrometry-benchmark-metric-tabulation
description: Decision support for quantifying how scaffold retrieval error limits
  spectra-conditioned molecular generation, when per-condition metric tables for both
  an oracle scaffold retriever (lookup table) and a predictive contrastive retriever
  are available alongside SPA values.
when_to_use_negative:
- Only one retriever condition (oracle or predictive alone) is available — no oracle-minus-predictive
  gap can be computed without paired per-condition tables
- The goal is to evaluate a method like MSNovelist whose results are limited to accuracy
  metrics — Tanimoto similarity and MCES edit distance are unavailable ('-') for such
  methods and cannot be gap-tabulated
- The question is about generation quality with the oracle retriever only (e.g., attention-integration
  effectiveness), not about the effect of scaffold retrieval error
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: MADGEN
  role: Source of the two-stage scaffold-retrieval + spectra-conditioned generation
    pipeline whose oracle and predictive retriever conditions are compared
  repo: https://github.com/HassounLab/MADGEN
- name: Python
  role: Compute per-dataset oracle-minus-predictive metric gaps, join with SPA values,
    and write the gap/SPA CSV table
- name: RDKit
  role: Compute maximum Tanimoto similarity between true and predicted molecules using
    Morgan fingerprints (radius=2, 2048 bits) as one of the tabulated metrics
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:07230fd18576d791e92dfc3746fa7512b4e82f4cfdf732d7471f8ad72b95b294
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
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mass-spectrometry-benchmark-metric-tabulation

## Summary

Tabulates oracle-vs-predictive performance gaps for MADGEN's two-stage scaffold-based de novo generation across NIST23, CANOPUS, and MassSpecGym, joining each metric gap with the retriever's scaffold prediction accuracy (SPA) to quantify how scaffold retrieval error constrains generation.

## When to use

Decision support for quantifying how scaffold retrieval error limits spectra-conditioned molecular generation, when per-condition metric tables for both an oracle scaffold retriever (lookup table) and a predictive contrastive retriever are available alongside SPA values.

## When NOT to use

- Only one retriever condition (oracle or predictive alone) is available — no oracle-minus-predictive gap can be computed without paired per-condition tables
- The goal is to evaluate a method like MSNovelist whose results are limited to accuracy metrics — Tanimoto similarity and MCES edit distance are unavailable ('-') for such methods and cannot be gap-tabulated
- The question is about generation quality with the oracle retriever only (e.g., attention-integration effectiveness), not about the effect of scaffold retrieval error

## Inputs

- Reproduced per-condition metric tables for MADGEN oracle-retriever runs (NIST23, CANOPUS, MassSpecGym)
- Reproduced per-condition metric tables for MADGEN predictive-retriever runs (NIST23, CANOPUS, MassSpecGym)
- SPA (scaffold prediction accuracy) values for the predictive scaffold retriever on each dataset
- CSV evaluation files with columns true (true SMILES), pred (predicted SMILES), scaffold (scaffold SMILES)

## Outputs

- CSV table with columns dataset, metric, oracle value, predictive value, gap, and SPA
- Qualitative interpretation of the gap-vs-SPA relationship (whether scaffold prediction accuracy constrains de novo generation accuracy)

## How to apply

Load the reproduced per-condition metric tables for the oracle-retriever and predictive-retriever MADGEN runs on NIST23, CANOPUS, and MassSpecGym. For each dataset, compute the oracle-minus-predictive gap for each metric: Top-1 and Top-10 accuracy (InChI-key matched), maximum Tanimoto similarity from Morgan fingerprints (radius 2, 2048 bits), and MCES edit distance. Join each per-dataset gap with the reported SPA of the predictive scaffold retriever (40.3%/17.7%/13.2% on NIST23/CANOPUS/MassSpecGym). Write a CSV with columns dataset, metric, oracle value, predictive value, gap, and SPA, then interpret whether lower SPA corresponds to smaller or larger oracle-vs-predictive degradation — the expected pattern is that predictive top-1 accuracy (4.6%/2.10%/1.31%) falls far below oracle (49.0%/18.7%/10.5%), with the largest gap (~44.4 points) on NIST23 where SPA is highest.

## Related tools

- **MADGEN** (Source of the two-stage scaffold-retrieval + spectra-conditioned generation pipeline whose oracle and predictive retriever conditions are compared) — https://github.com/HassounLab/MADGEN
- **Python** (Compute per-dataset oracle-minus-predictive metric gaps, join with SPA values, and write the gap/SPA CSV table)
- **RDKit** (Compute maximum Tanimoto similarity between true and predicted molecules using Morgan fingerprints (radius=2, 2048 bits) as one of the tabulated metrics) — https://github.com/HassounLab/MADGEN

## Examples

```
python evaluation_generation.py --file_path /path/to/your/csvfile.csv
```

## Evaluation signals

- Tabulated oracle and predictive values match the reproduced per-condition metric tables exactly for every dataset and metric
- Gap column equals oracle minus predictive for each dataset-metric row (e.g., NIST23 Top-1: 49.0 - 4.6 = ~44.4 percentage points)
- SPA values in the CSV match the reported retriever accuracies: 40.3% (NIST23), 17.7% (CANOPUS), 13.2% (MassSpecGym)
- MSNovelist rows (if included) contain only accuracy metrics; similarity and MCES cells are marked '-' rather than filled with invented values
- The qualitative gap-vs-SPA ordering is consistent with the paper's two-retriever evaluation (expert review of the interpretation)

## Limitations

- MSNovelist baseline results are limited to accuracy metrics; other measures (similarity, MCES) are not available, so gap tables covering all metrics cannot include it
- The relationship between SPA and oracle-vs-predictive gaps is assessed qualitatively (largest gap on NIST23 where SPA is highest), not as a fitted quantitative law
- Oracle retrieval requires a lookup table keyed by MS/MS spectrum and chemical formula, so the oracle condition presupposes knowledge of the correct scaffold and formula

## Evidence
