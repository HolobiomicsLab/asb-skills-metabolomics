---
name: cross-validation-rank-statistics-evaluation
description: Use when you hold a table of generative-model outputs with sampling frequencies
  and want to test whether frequency alone prioritizes withheld cross-validation metabolites
  or newly released HMDB structures over generated decoys.
when_to_use_negative:
- When the query structures were part of the model's training data — ranks would be
  inflated; only withheld test-fold metabolites or post-training releases like HMDB
  v5.0 additions give a fair ROC
- When prioritization should use analytical evidence (mass-to-charge ratio, MS/MS
  spectra) rather than sampling frequency alone — this skill isolates the frequency-only
  signal
- When applying to evolutionarily distant metabolomes (plant or bacterial metabolism),
  since DeepMet was trained and evaluated on human metabolite structures
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DeepMet
  role: chemical language model producing the 34.4M generated structures and their
    sampling frequencies that form the ranked table
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: canonicalizes withheld test and HMDB v5.0 SMILES for SMILES/InChIKey matching
    against the ranked frequency table
- name: Snakemake
  role: runs the end-end clm Snakefile workflow that produces the evaluation logic
    being reproduced
  repo: https://github.com/skinniderlab/CLM
- name: Python 3.10
  role: environment for computing ROC statistics, hypergeometric test, and writing
    metrics/plot outputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:c09a4669780973abcee7c3a3b030eadb6de8fa116a6ebfbb1497d68260cae297
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - Python
  - Python 3.10
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-validation-rank-statistics-evaluation

## Summary

Evaluates whether a chemical language model's sampling frequency can rank withheld or newly discovered metabolites above its generated background, using rank-based ROC/AUC analysis and hypergeometric enrichment against HMDB.

## When to use

Use when you hold a table of generative-model outputs with sampling frequencies and want to test whether frequency alone prioritizes withheld cross-validation metabolites or newly released HMDB structures over generated decoys.

## When NOT to use

- When the query structures were part of the model's training data — ranks would be inflated; only withheld test-fold metabolites or post-training releases like HMDB v5.0 additions give a fair ROC
- When prioritization should use analytical evidence (mass-to-charge ratio, MS/MS spectra) rather than sampling frequency alone — this skill isolates the frequency-only signal
- When applying to evolutionarily distant metabolomes (plant or bacterial metabolism), since DeepMet was trained and evaluated on human metabolite structures

## Inputs

- DeepMet generated-structure table of 34.4M SMILES with per-structure sampling frequencies
- Withheld cross-validation test SMILES from the clm train/test fold split
- 313 metabolite SMILES newly added in HMDB v5.0

## Outputs

- evaluation_metrics.csv (CV AUC, generated fraction, prospective AUC, top-10,000 hit count, hypergeometric p-value)
- roc_curves.png with both cross-validation and prospective ROC curves

## How to apply

Load the released DeepMet resource of 34.4M generated SMILES with their sampling frequencies and sort structures by decreasing sampling frequency, mirroring the clm write_structural_prior_CV_overall evaluation logic. Canonicalize the withheld cross-validation test SMILES with RDKit, match each against the ranked table by SMILES/InChIKey comparison, record each hit's rank, and derive the cross-validation ROC (target AUC 0.98). For prospective validation, load the 313 SMILES newly added in HMDB v5.0, determine the generated fraction by matching against the database (target 252/313, 81%), and compute the prospective ROC on those 252 generated additions (target AUC 0.97). Count HMDB v5.0 additions falling within the top-10,000 most frequent structures (target 105/252) and compute enrichment over random expectation (~1,500-fold) with a hypergeometric test (target p = 4.3e-5). Write all metrics to evaluation_metrics.csv and plot both ROC curves to roc_curves.png.

## Related tools

- **DeepMet** (chemical language model producing the 34.4M generated structures and their sampling frequencies that form the ranked table) — https://github.com/skinniderlab/CLM
- **RDKit** (canonicalizes withheld test and HMDB v5.0 SMILES for SMILES/InChIKey matching against the ranked frequency table)
- **Snakemake** (runs the end-end clm Snakefile workflow that produces the evaluation logic being reproduced) — https://github.com/skinniderlab/CLM
- **Python 3.10** (environment for computing ROC statistics, hypergeometric test, and writing metrics/plot outputs)

## Examples

```
git clone https://github.com/skinniderlab/CLM.git && cd CLM && snakemake --cores 1  # end-end clm workflow incl. write_structural_prior_CV_overall evaluation logic
```

## Evaluation signals

- Cross-validation ROC AUC over withheld metabolites reaches ~0.98
- 252 of 313 HMDB v5.0 newly added metabolites (81%) are found among the generated structures
- Prospective ROC on the 252 generated HMDB v5.0 additions reaches ~0.97
- 105 of 252 HMDB v5.0 additions fall within the top-10,000 most frequent structures
- Hypergeometric enrichment over random expectation is ~1,500-fold with p = 4.3e-5

## Limitations

- DeepMet was trained and evaluated on the structures of human metabolites; evolutionarily distant applications (e.g., plant or bacterial metabolism) will likely require bespoke models
- The model has inherent limitations anticipating metabolites that originate from divergent, as-of-yet undiscovered biosynthetic routes
- Sampling frequency alone is a weak single-feature ranker outside metabolite-like chemical space; missing matches (61/313 HMDB v5.0 additions) yield no rank and are excluded from the prospective ROC

## Evidence
