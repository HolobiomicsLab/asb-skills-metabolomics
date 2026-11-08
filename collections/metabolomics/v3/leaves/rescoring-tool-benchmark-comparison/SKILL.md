---
name: rescoring-tool-benchmark-comparison
description: Use when a review or paper reports multiple PSM rescoring benchmark results
  that must be reconciled into a structured comparison, especially when findings appear
  contradictory across tools or acquisition conditions (DDA vs timsTOF, RNN vs transformer
  predictors).
when_to_use_negative:
- When the goal is to actually execute PSM rescoring on spectra (feature generation
  with MS²PIP/DeepLC and ristretto SVM/LDA rescoring) rather than comparing published
  benchmark findings.
- When the source text reports only one tool's result with no cross-tool comparison
  to reconcile.
- When the article contains no methods/discussion sections reporting empirical results
  (e.g. a changelog-style document with no evaluation data).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: DeepRescore
  role: 'Benchmark subject: reported delta-RT plus spectral-angle features as dominant
    in DDA immunopeptidomics rescoring'
- name: inSPIRE
  role: 'Benchmark subject: combines binding-affinity and spectral features for immunopeptide
    rescoring'
- name: MHCquant
  role: 'Benchmark subject: reported ΔRT ineffective due to reliance on the outdated
    RTPredict retention time predictor'
- name: MSBooster
  role: 'Benchmark subject: showed marginal benefit of ion-mobility features'
- name: Prosit
  role: 'Benchmark subject: spectral predictor finetuned for timsTOF; RNN architecture
    outperformed by pretrained transformers'
- name: Percolator
  role: 'Benchmark subject: exhibited numProt bias in reported results; semi-supervised
    rescoring algorithm reimplemented by ristretto'
- name: MS2Rescore
  role: Representative modular rescoring platform whose predictor-feature design the
    benchmark comparisons evaluate
  repo: https://github.com/compomics/ms2rescore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:18722fc7c7f574884ed528a6c778b44389a00905d8bb9857a90c3a64ec4d58e0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/compomics/ms2rescore
  tools:
  - DeepRescore
  - MHCquant
  - MS2Rescore
  - MSBooster
  - Percolator
  - Prosit
  - inSPIRE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# rescoring-tool-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts and cross-compares empirical PSM-rescoring benchmark findings reported across tools (DeepRescore, inSPIRE, MHCquant, MSBooster, Prosit, Percolator) into a structured finding–tool–condition table, so contradictory results (e.g. ΔRT judged ineffective by MHCquant but a top feature in DeepRescore) can be traced to their experimental conditions.

## When to use

Use when a review or paper reports multiple PSM rescoring benchmark results that must be reconciled into a structured comparison, especially when findings appear contradictory across tools or acquisition conditions (DDA vs timsTOF, RNN vs transformer predictors).

## When NOT to use

- When the goal is to actually execute PSM rescoring on spectra (feature generation with MS²PIP/DeepLC and ristretto SVM/LDA rescoring) rather than comparing published benchmark findings.
- When the source text reports only one tool's result with no cross-tool comparison to reconcile.
- When the article contains no methods/discussion sections reporting empirical results (e.g. a changelog-style document with no evaluation data).

## Inputs

- Full text of a PSM rescoring review or benchmark article (e.g. DOI 10.1002/pmic.202300336)
- Named tools per reported result (DeepRescore, inSPIRE, MHCquant, MSBooster, Prosit, Percolator, MS2Rescore)
- Experimental-condition descriptions per finding (acquisition type, predictor architecture, feature set)

## Outputs

- findings_table.json: list of exactly 7 records, each with non-empty 'finding', 'tool', and 'condition' fields
- Reconciliation narrative attributing contradictory conclusions (e.g. ΔRT effectiveness) to predictor accuracy

## How to apply

Load the full text of the review (DOI 10.1002/pmic.202300336) and locate each reported empirical benchmark result: DeepRescore's delta-RT plus spectral-angle dominance in DDA immunopeptidomics, inSPIRE's binding-affinity/spectral-feature concert, MHCquant's delta-RT ineffectiveness, MSBooster's marginal ion-mobility benefit, numProt bias in Percolator, Prosit finetuned for timsTOF, and transformer-with-pretraining outperforming Prosit's RNN. For each of the 7 findings, parse the reported result, the named tool, and the experimental condition (acquisition type, predictor architecture, feature set) into a structured record, then assemble them into a single table with fields 'finding', 'tool', and 'condition' and serialize to findings_table.json. Treat conflicting conclusions as signal rather than error: attribute differences to prediction-model quality — e.g. MHCquant relied on the >15-year-old RTPredict retention time predictor, while ΔRT was consistently among the highest-weight features in DeepRescore — because informative rescoring features require highly accurate predicted peptidoform properties (fragment ion intensities, retention time, collisional cross section). Validate the output by checking the file parses as JSON, contains exactly 7 records, and every record has non-empty fields, with tool-condition mapping confirmed by expert review against the article text.

## Related tools

- **DeepRescore** (Benchmark subject: reported delta-RT plus spectral-angle features as dominant in DDA immunopeptidomics rescoring)
- **inSPIRE** (Benchmark subject: combines binding-affinity and spectral features for immunopeptide rescoring)
- **MHCquant** (Benchmark subject: reported ΔRT ineffective due to reliance on the outdated RTPredict retention time predictor)
- **MSBooster** (Benchmark subject: showed marginal benefit of ion-mobility features)
- **Prosit** (Benchmark subject: spectral predictor finetuned for timsTOF; RNN architecture outperformed by pretrained transformers)
- **Percolator** (Benchmark subject: exhibited numProt bias in reported results; semi-supervised rescoring algorithm reimplemented by ristretto)
- **MS2Rescore** (Representative modular rescoring platform whose predictor-feature design the benchmark comparisons evaluate) — https://github.com/compomics/ms2rescore

## Examples

```
python -c "import json; t=json.load(open('findings_table.json')); assert len(t)==7 and all(r.get('finding') and r.get('tool') and r.get('condition') for r in t)"
```

## Evaluation signals

- findings_table.json exists and parses as valid JSON
- The table contains exactly 7 records
- Every record has non-empty 'finding', 'tool', and 'condition' fields
- Each tool–condition mapping is checked by expert review against the article text
- Each finding names a tool from the article's reported benchmark set (DeepRescore, inSPIRE, MHCquant, MSBooster, Prosit, Percolator) and a concrete condition (e.g. DDA immunopeptidomics, timsTOF, RNN vs transformer predictor)

## Limitations

- MHCquant's negative ΔRT result stems from the outdated RTPredict predictor (published >15 years ago), so 'feature ineffective' conclusions may reflect predictor quality rather than the feature class itself.
- MSBooster's ion-mobility benefit was only marginal, so feature-class conclusions should not be generalized without noting the small effect size.
- Percolator results carry a numProt bias that conditions interpretation of its reported benchmark numbers.

## Evidence

- [abstract] PSM rescoring as machine-learning solution: [evidence span withheld — anchor status: not_found]
- [readme] ristretto reimplements Percolator algorithm: [evidence span withheld — anchor status: external_doc]
