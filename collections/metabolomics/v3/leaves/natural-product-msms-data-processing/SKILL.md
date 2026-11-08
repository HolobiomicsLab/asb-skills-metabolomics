---
name: natural-product-msms-data-processing
description: Use when given MS/MS spectra of two structurally related natural products
  (known + unknown analog) with 2D structures, and the goal is to localize the modification
  site on the known molecule or reproduce benchmark analog-pair analyses.
when_to_use_negative:
- Input is de novo structure elucidation with no structurally related known analog
  — ModiFinder requires an MS2-known/MS2-unknown pair with a known structure
- Spectra have unmatched precursor masses or violate benchmark eligibility (precursor
  > 2000 Da, precursor mass difference > 50%)
- The goal is compound identification by library matching alone rather than localization
  of the modification site on a known structure
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ModiFinder
  role: 'Core tool: loads compounds/spectra, aligns MS/MS peaks, runs MF-R with MAGMa
    fragmentation, scores per-atom modification-site likelihoods'
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Combinatorial fragmentation engine (breadth-first, depth 2) assigning candidate
    substructures to peaks within 40 ppm tolerance
- name: MS/MS library search
  role: Source of the analog spectral pairs (GNPS/TUEBINGEN-NATURAL-PRODUCT-COLLECTION
    accessions) retrieved via Compound/Spectrum loaders
- name: SIRIUS
  role: Provides predicted formulas to filter possible substructures for each MS/MS
    peak (Substructure Refinement By Formula)
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:877509d5fe4933aa9bcf45e9031a5da7ec5aa139640bde08939af62b52ac60af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# natural-product-msms-data-processing

## Summary

Load and preprocess natural-product MS/MS spectra (e.g., Kirromycin/Goldinodox, Naphthomycin A/B from GNPS/TUEBINGEN libraries), then run ModiFinder's MF-R workflow to localize structural modification sites between compound analog pairs, refining peak-to-fragment maps with domain knowledge.

## When to use

Use when given MS/MS spectra of two structurally related natural products (known + unknown analog) with 2D structures, and the goal is to localize the modification site on the known molecule or reproduce benchmark analog-pair analyses.

## When NOT to use

- Input is de novo structure elucidation with no structurally related known analog — ModiFinder requires an MS2-known/MS2-unknown pair with a known structure
- Spectra have unmatched precursor masses or violate benchmark eligibility (precursor > 2000 Da, precursor mass difference > 50%)
- The goal is compound identification by library matching alone rather than localization of the modification site on a known structure

## Inputs

- MS/MS spectrum (m/z–intensity peak list) of the known natural product with its 2D structure
- MS/MS spectrum of the structurally related unknown/analog compound with its 2D structure
- True modification site annotation on the known molecule (for evaluation-score computation)
- Peak-to-fragment map (substructures assigned per peak) for optional expert refinement

## Outputs

- Per-atom likelihood score distribution over the known molecule's structure
- Average-distance Evaluation Scores (pre- and post-refinement)
- High-scoring modification site counts / ambiguity-reduction counts
- refinement_scores.csv with pre-/post-refinement scores and counts

## How to apply

Retrieve both analog spectra with their 2D structures from the MS/MS library (e.g., TUEBINGEN/GNPS accessions) using ModiFinder's Compound/Spectrum loaders. Preprocess each spectrum per ModiFinder defaults: remove peaks below 1% of the base peak and normalize intensities to Euclidean norm 1. Run MF-R (MAGMa combinatorial fragmentation at depth 2, 40 ppm m/z tolerance, bipartite graph greedy alignment) to obtain a likelihood distribution over atoms of the known molecule, then compute Average-distance Evaluation Scores with the BasicEvaluationEngine against the true modification sites. Apply expert refinement by editing peak-to-fragment maps — e.g., restrict specific m/z peaks (like 112.04/178.05 for Kirromycin, 133.07/147.08 for Naphthomycin B) from tens of candidate substructures down to the amide-bond fragmentation products — regenerate probabilities, and apply post-processing (values below 0.5 set to 0, scores raised to the power of 4). Verify against expected score improvements (0.86→0.93 Kirromycin, 0.56→0.72 Naphthomycin B) and reduced ambiguity in high-scoring sites.

## Related tools

- **ModiFinder** (Core tool: loads compounds/spectra, aligns MS/MS peaks, runs MF-R with MAGMa fragmentation, scores per-atom modification-site likelihoods) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Combinatorial fragmentation engine (breadth-first, depth 2) assigning candidate substructures to peaks within 40 ppm tolerance)
- **MS/MS library search** (Source of the analog spectral pairs (GNPS/TUEBINGEN-NATURAL-PRODUCT-COLLECTION accessions) retrieved via Compound/Spectrum loaders)
- **SIRIUS** (Provides predicted formulas to filter possible substructures for each MS/MS peak (Substructure Refinement By Formula)) — https://github.com/boecker-lab/sirius

## Examples

```
from modidf import Compound, Spectrum; kf, kf2 = Compound.load('kirromycin.mol'), Spectrum.load('goldinodox.mgf'); scores = mf_r(known=(kf, kf_msnms), unknown=(kf2, kf2_msnms), depth=2, tolerance=40)
```

## Evaluation signals

- Baseline Evaluation Scores match published values: 0.86 for Kirromycin/Goldinodox and 0.56 for Naphthomycin A/B before refinement
- Post-refinement scores reach 0.93 and 0.72 respectively after expert peak-to-fragment map edits
- Naphthomycin B high-scoring modification sites drop from 24 to 2 after restricting 133.07/147.08 m/z peaks from 27/40 substructures to 2 each
- Preprocessed spectra contain no peaks below 1% of base peak and intensity vector has Euclidean norm 1
- Final likelihood scores are post-processed: values below 0.5 set to 0 and raised to the power of 4

## Limitations

- Accuracy depends on the analog pair qualifying for the benchmark (precursor mass < 2000 Da; precursor mass difference < 50%)
- Ambiguous peak-to-fragment assignments (tens of candidate substructures per peak) can wash out the signal unless refined with domain knowledge of gas-phase fragmentation
- Helpers-based refinement relies on reference compounds with precursor mass within 0.5 of the unknown — not always available
- Software is under active development and licensed for academic/nonprofit use only

## Evidence

- [readme] ModiFinder is a tool for site localization of structural modifications using MS/MS data.: [evidence span withheld — anchor status: not_found]
