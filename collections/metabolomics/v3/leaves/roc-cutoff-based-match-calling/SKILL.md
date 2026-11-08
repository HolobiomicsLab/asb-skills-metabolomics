---
name: roc-cutoff-based-match-calling
description: Use when query MS/MS spectra (e.g. Exploris 480, 150 CP and 134 CN modes)
  must be scored against a multi-collision-energy reference library and a binary match/no-match
  decision is needed using validated Figure 2 cutoff thresholds.
when_to_use_negative:
- Query spectra were acquired on the same instrument as the reference library under
  identical collision energies — cross-sectional (single-CE) similarity methods such
  as dot product or optimal entropy may suffice
- No ROC-derived cutoff criteria exist for the reference library in use; thresholds
  would have to be re-derived before calling matches
- The goal is candidate ranking rather than binary match calling — similarity scores
  alone, without cutoff application, are the appropriate output
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Jaccard similarity algorithm
  role: Optimized longitudinal-profile scoring that weighs predominant and minute
    fragments independently to produce query-reference similarity scores before cutoff
    application
- name: mzbatch.py
  role: Library of mass spectrometry data processing methods (filtering, neutral-loss
    removal, fragment rationality assessment) applied to query spectra prior to scoring
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series executing spectrum extraction, filtering, isomer similarity
    calculations, and identification; run every cell in sequence
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Python 3.8.13
  role: Runtime environment for the pipeline (conda env 'longprof' with requirements.txt)
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:cdf3d78dd38b55ebf436a13893c4f9b34b348edceef501d00a246fbc46ca92f6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - conda
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# roc-cutoff-based-match-calling

## Summary

Calls MS/MS query-to-reference metabolite matches by applying cutoff criteria derived from ROC analysis of thousands of isomers in the in-house CQMD database, optionally with 0.8-fold relaxed cutoffs for sensitivity-oriented re-scoring. Used here to quantify inter-instrument transferability (Exploris 480 queries vs Exploris 120 CQMD references).

## When to use

Use when query MS/MS spectra (e.g. Exploris 480, 150 CP and 134 CN modes) must be scored against a multi-collision-energy reference library and a binary match/no-match decision is needed using validated Figure 2 cutoff thresholds.

## When NOT to use

- Query spectra were acquired on the same instrument as the reference library under identical collision energies — cross-sectional (single-CE) similarity methods such as dot product or optimal entropy may suffice
- No ROC-derived cutoff criteria exist for the reference library in use; thresholds would have to be re-derived before calling matches
- The goal is candidate ranking rather than binary match calling — similarity scores alone, without cutoff application, are the appropriate output

## Inputs

- Query MS/MS spectra acquired on a different instrument (150 CP-mode and 134 CN-mode Exploris 480 spectra from an independent laboratory)
- CQMD reference spectral database (~180,000 MS/MS spectra of >4500 metabolites at 13 collision energies and stepped NCE 20/40/60%)
- Figure 2 ROC-derived cutoff criteria and their 0.8-fold relaxed variants

## Outputs

- Per-spectrum match calls under strict Figure 2 cutoff criteria (CP and CN counts)
- Per-spectrum match calls under 0.8-fold relaxed cutoff criteria
- Matching-rate summary table, e.g. interinstrument_matching_results.csv with matched/total counts and percentages

## How to apply

Preprocess each query spectrum in the 001-007.ipynb pipeline using mzbatch.py: remove peaks with relative intensity below 1%, filter fragments corresponding to common neutral losses, and run fragment rationality assessment. Compute longitudinal fragment profiles (relative ion intensities across 13 distinct CEs plus stepped NCE 20/40/60%) for query spectra and matching CQMD reference entries, then score pairs with the optimized Jaccard similarity algorithm that weighs predominant and minute fragments independently. Apply the Figure 2 cutoff criteria — established by ROC analysis of thousands of isomers in CQMD — to call a match, counting matched CP and CN spectra out of totals (150 CP, 134 CN). Repeat the calling step with 0.8-fold relaxed cutoffs when near-complete matching rates are needed. Tabulate matched/total counts and percentages for both strict and relaxed criteria and save the summary (e.g. interinstrument_matching_results.csv).

## Related tools

- **Jaccard similarity algorithm** (Optimized longitudinal-profile scoring that weighs predominant and minute fragments independently to produce query-reference similarity scores before cutoff application)
- **mzbatch.py** (Library of mass spectrometry data processing methods (filtering, neutral-loss removal, fragment rationality assessment) applied to query spectra prior to scoring) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series executing spectrum extraction, filtering, isomer similarity calculations, and identification; run every cell in sequence) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Python 3.8.13** (Runtime environment for the pipeline (conda env 'longprof' with requirements.txt)) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt; jupyter nbconvert --to notebook --execute 001-007.ipynb --inplace
```

## Evaluation signals

- Strict-criteria matching rates reproduce the reported values: 95.33% of 150 CP spectra (= 143 matched) and 91.79% of 134 CN spectra (= 123 matched)
- 0.8-fold relaxed cutoff criteria yield a nearly 100% matching rate for both CP and CN modes
- Every query spectrum was preprocessed with the <1% relative-intensity peak filter, common neutral-loss removal, and fragment rationality assessment before scoring
- Match calls are based on the same Figure 2 cutoff criteria derived from ROC analysis of thousands of isomers in CQMD, not ad hoc thresholds
- Summary CSV contains matched/total counts and percentages for both strict and relaxed criteria, split by CP and CN acquisition modes

## Limitations

- The CQMD reference database is described as in-house with no accession or public access stated, limiting out-of-the-box reuse
- Matching rates depend on reference entries acquired across 13 distinct CEs plus stepped NCE 20/40/60%; single-CE references cannot form longitudinal profiles
- Repository README states the data are 'intended for experimental reference only and should not be directly interpreted or used to guide human medicine' and are provided 'as is' without warranty

## Evidence

- [results] Treating Exploris 480 spectra as query spectra against CQMD references, 95.33% (CP) and 91.79% (CN) of metabolites were satisfactorily matched under the Figure 2 cutoff criteria: [evidence span withheld — anchor status: not_found]
- [methods] Peak filtering threshold: [evidence span withheld — anchor status: not_found]
- [methods] Neutral loss filtering: [evidence span withheld — anchor status: not_found]
