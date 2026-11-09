---
name: jaccard-similarity-scoring-multi-collision-energy-fragments
description: Use when query MS/MS spectra are acquired at multiple collision energies
  and must be matched to multi-CE reference spectra (e.g. Exploris 480 queries vs.
  CQMD Exploris 120 references) to call metabolite identifications under Figure 2
  cutoff criteria.
when_to_use_negative:
- Query or reference spectra were acquired at only a single collision energy, so no
  longitudinal fragment profile (fragment intensity across multiple CEs) can be constructed
- The task is generic cross-sectional spectral similarity scoring against public libraries
  (MoNA, GNPS, METLIN) rather than longitudinal profiling-based metabolite identification
  against a multi-CE reference set
- Reference spectra lack the matched 13-CE / stepped NCE 20/40/60% acquisition scheme
  required to align fragment profiles with the query spectra
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Jaccard similarity algorithm
  role: Optimized similarity metric that weighs predominant and minute fragments of
    longitudinal fragment profiles independently to score query-to-reference MS/MS
    pairs
- name: mzbatch.py
  role: Python library of mass spectrometry data-processing methods (file operations,
    data validation, calculation, extraction) used to load and preprocess query spectra
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series implementing spectrum filtering, invalid fragment removal,
    fragment rationality assessment, isomer similarity calculations, and metabolite
    identification run sequentially in Jupyter Notebook
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: conda
  role: Creates the longprof Python 3.8.13 environment in which the scoring pipeline
    is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:37ef7db3d0919eb33eda15e02fa05ec621650d10cac70f789e4837a25fe4e4d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - conda
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6818181818181818
n_claims: 22
---

# jaccard-similarity-scoring-multi-collision-energy-fragments

## Summary

Scores query MS/MS spectra against a multi-collision-energy reference database (CQMD, ~180,000 spectra at 13 distinct CEs and stepped NCE 20/40/60%) using an optimized Jaccard similarity over longitudinal fragment profiles that weighs predominant and minute fragments independently.

## When to use

Use when query MS/MS spectra are acquired at multiple collision energies and must be matched to multi-CE reference spectra (e.g. Exploris 480 queries vs. CQMD Exploris 120 references) to call metabolite identifications under Figure 2 cutoff criteria.

## When NOT to use

- Query or reference spectra were acquired at only a single collision energy, so no longitudinal fragment profile (fragment intensity across multiple CEs) can be constructed
- The task is generic cross-sectional spectral similarity scoring against public libraries (MoNA, GNPS, METLIN) rather than longitudinal profiling-based metabolite identification against a multi-CE reference set
- Reference spectra lack the matched 13-CE / stepped NCE 20/40/60% acquisition scheme required to align fragment profiles with the query spectra

## Inputs

- Query MS/MS spectra acquired at 13 distinct collision energies (e.g. 150 CP-mode and 134 CN-mode Exploris 480 spectra from an independent laboratory)
- CQMD reference spectral database (~180,000 MS/MS spectra of >4500 metabolites at 13 CEs and stepped NCE 20/40/60%)
- Candidate metabolite lists (e.g. retrieved from KEGG and HMDB)

## Outputs

- Per-pair optimized Jaccard similarity scores between longitudinal fragment profiles of query and reference spectra
- Match/no-match calls under Figure 2 strict cutoff criteria and 0.8-fold relaxed cutoff criteria
- Matching-rate summaries (matched/total per mode, e.g. interinstrument_matching_results.csv)

## How to apply

Preprocess each query spectrum by removing peaks with relative intensity below 1%, filtering fragments corresponding to common neutral losses, and running fragment rationality assessment via the 001-007.ipynb pipeline and mzbatch.py library. Build longitudinal fragment profiles (relative ion intensities of each fragment across the 13 distinct collision energies) for query and reference spectra so predominant and minute fragments are scored independently rather than diluted as in cross-sectional dot-product or entropy scoring. Score each query-to-reference pair with the optimized Jaccard similarity and call matches using the Figure 2 cutoff criteria derived by ROC analysis of thousands of isomers in CQMD; optionally apply 0.8-fold relaxed cutoffs for near-100% sensitivity. Benchmark reproducibility, e.g. Exploris 480 queries against Exploris 120 CQMD references yielded 95.33% (CP, 143/150) and 91.79% (CN, 123/134) strict matching rates.

## Related tools

- **Jaccard similarity algorithm** (Optimized similarity metric that weighs predominant and minute fragments of longitudinal fragment profiles independently to score query-to-reference MS/MS pairs)
- **mzbatch.py** (Python library of mass spectrometry data-processing methods (file operations, data validation, calculation, extraction) used to load and preprocess query spectra) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series implementing spectrum filtering, invalid fragment removal, fragment rationality assessment, isomer similarity calculations, and metabolite identification run sequentially in Jupyter Notebook) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **conda** (Creates the longprof Python 3.8.13 environment in which the scoring pipeline is executed)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt
```

## Evaluation signals

- Strict-criteria matching rates reproduce 95.33% CP (143 of 150 Exploris 480 spectra matched) and 91.79% CN (123 of 134 matched) against CQMD references
- 0.8-fold relaxed cutoff criteria yield nearly 100% matching rates in both CP and CN modes
- Preprocessing invariants hold: no peaks with relative intensity below 1% and no common neutral-loss fragments remain in spectra entering the scoring step
- Longitudinal fragment profiles span all 13 distinct collision energies (and stepped NCE 20/40/60%) for every query and reference spectrum scored
- Cut-off calls are consistent with the ROC-derived Figure 2 criteria established from thousands of CQMD isomers, not ad-hoc thresholds

## Limitations

- The CQMD in-house database has no stated public accession, so an independent laboratory may not be able to obtain the multi-CE reference spectra
- The identification cutoffs are calibrated by ROC analysis of isomers within CQMD; porting them to other reference libraries or instrument families is not validated by the paper
- Matched longitudinal profiles require spectra acquired on instruments with comparable stepped-NCE acquisition (validation was Exploris 480 queries vs. Exploris 120 references); single-CE legacy libraries cannot be scored this way

## Evidence

- [methods] Peaks with relative intensity below 1% were removed from the MS/MS spectra.: [evidence span withheld — anchor status: not_found]
