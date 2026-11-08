---
name: roc-auc-metabolite-retrieval-assessment
description: Use when you have a set of generated structures with sampling frequencies
  and need to quantify how well that frequency signal retrieves withheld or newly-discovered
  metabolites (ROC AUC, enrichment, hypergeometric p-value).
when_to_use_negative:
- The evaluation set is drawn from the same time period / release as the model's training
  data, so 'novel' structures may already be in the training corpus (e.g. metabolites
  absent from or misannotated within the HMDB due to curation errors require explicit
  curation-error handling, not just frequency ranking)
- You need prioritization for metabolites from evolutionarily distant organisms (plant
  or bacterial metabolism) — the article notes the language model was trained and
  evaluated only on human metabolites and such applications will likely require a
  bespoke model
- You have no sampling-frequency or other model-intrinsic signal available and only
  possess an experimental mass-to-charge ratio to filter on
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: DeepMet
  role: Source of the generated structure resource (34.4M SMILES with sampling frequencies)
    being evaluated; also the model whose cross-validation withheld metabolites are
    matched
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Canonicalization of withheld test SMILES and molecule handling for matching
    against the ranked frequency table
- name: Snakemake
  role: Runs the end-end `clm` workflow including the evaluation step that writes
    ROC/enrichment metrics
  repo: https://github.com/skinniderlab/CLM
- name: Python
  role: Implementation language for the evaluation script (clm repository requires
    Python 3.10 or later)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:ebe04082a421e315a6f9409ee538b54a592b3579674d766531bd76604f0da35c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - Python
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# roc-auc-metabolite-retrieval-assessment

## Summary

Evaluates whether a generative chemical language model's sampling frequency prioritizes genuinely novel metabolites by ranking generated SMILES, matching withheld cross-validation and prospective (e.g. HMDB v5.0) additions against the ranked table, and reporting ROC AUCs plus enrichment statistics.

## When to use

Use when you have a set of generated structures with sampling frequencies and need to quantify how well that frequency signal retrieves withheld or newly-discovered metabolites (ROC AUC, enrichment, hypergeometric p-value).

## When NOT to use

- The evaluation set is drawn from the same time period / release as the model's training data, so 'novel' structures may already be in the training corpus (e.g. metabolites absent from or misannotated within the HMDB due to curation errors require explicit curation-error handling, not just frequency ranking)
- You need prioritization for metabolites from evolutionarily distant organisms (plant or bacterial metabolism) — the article notes the language model was trained and evaluated only on human metabolites and such applications will likely require a bespoke model
- You have no sampling-frequency or other model-intrinsic signal available and only possess an experimental mass-to-charge ratio to filter on

## Inputs

- Resource of 34.4 million generated SMILES with corresponding sampling frequencies (DeepMet output)
- Withheld cross-validation test SMILES (canonicalizable via RDKit)
- 313 SMILES of metabolites newly added in HMDB v5.0
- Ranked frequency table of generated structures

## Outputs

- evaluation_metrics.csv (CV AUC, prospective AUC, generated fraction, top-10k count, fold enrichment, hypergeometric p-value)
- roc_curves.png (cross-validation and prospective ROC curves)
- Per-metabolite rank within the ranked frequency table

## How to apply

Load the released resource of 34.4 million generated SMILES with their sampling frequencies and sort structures by decreasing sampling frequency, mirroring the `clm write_structural_prior_CV_overall` evaluation logic. Canonicalize the withheld cross-validation test SMILES with RDKit, match each against the ranked frequency table by SMILES/InChIKey comparison, record each hit's rank, and derive the cross-validation ROC (target AUC 0.98). Separately, load the 313 SMILES of metabolites newly added in HMDB v5.0, determine the generated fraction by matching against the database (target 252/313, i.e. 81%), compute the prospective ROC on those 252 generated additions (target AUC 0.97), and count how many fall within the top-10,000 most frequent structures (target 105/252). Convert that count to fold enrichment over random expectation (~1,500-fold) and test significance with a hypergeometric test (target p = 4.3e-5). Write all metrics to evaluation_metrics.csv and plot both ROC curves to roc_curves.png.

## Related tools

- **DeepMet** (Source of the generated structure resource (34.4M SMILES with sampling frequencies) being evaluated; also the model whose cross-validation withheld metabolites are matched) — https://github.com/skinniderlab/CLM
- **RDKit** (Canonicalization of withheld test SMILES and molecule handling for matching against the ranked frequency table)
- **Snakemake** (Runs the end-end `clm` workflow including the evaluation step that writes ROC/enrichment metrics) — https://github.com/skinniderlab/CLM
- **Python** (Implementation language for the evaluation script (clm repository requires Python 3.10 or later))

## Examples

```
snakemake --use-conda write_structural_prior_CV_overall  # or: python -m clm.evaluate roc_curves --frequency-table generated_structures.csv --cv-smiles withheld_test.smiles --hmdb-v5-additions new_v5.smiles
```

## Evaluation signals

- Cross-validation ROC AUC of 0.98 (withheld metabolites vs other generated molecules)
- Prospective ROC AUC of 0.97 on the 252 HMDB v5.0 additions that were generated
- 252 of 313 HMDB v5.0 newly-added metabolites (81%) found within the generated database
- 105 of 252 generated HMDB v5.0 additions fall within the top-10,000 most frequently sampled structures
- Approximately 1,500-fold enrichment over random expectation with hypergeometric p = 4.3e-5

## Limitations

- The model was trained and evaluated on the structures of human metabolites; evolutionarily distant applications (plant or bacterial metabolism) will likely require a bespoke model.
- Inherent limitations in the model's ability to anticipate metabolites originating from divergent, as-of-yet undiscovered biosynthetic routes.
- Prioritization is well-calibrated even in the absence of other analytical data, but tight integration with analytical data (e.g. MS/MS, retention time) is not yet realized and predicting spectra/retention times from structures remains an open problem.

## Evidence

- [other] CV AUC 0.98: [evidence span withheld — anchor status: not_found]
- [other] Enrichment and hypergeometric test target values: [evidence span withheld — anchor status: not_found]
- [other] Workflow step loading and sorting by decreasing sampling frequency: [evidence span withheld — anchor status: not_found]
