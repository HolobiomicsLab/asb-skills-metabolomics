---
name: mass-spectrometry-annotation-file-parsing
description: Use when you have SIRIUS/ZODIAC annotation output files plus ground-truth
  formula lists (e.g. from MassIVE MSV000080502, MSV000081364) and need to reproduce
  or extend the molecular formula accuracy benchmark.
when_to_use_negative:
- Raw mzML/mzXML spectra that have not yet been annotated by SIRIUS/ZODIAC — run the
  annotation pipeline first (including the modified OpenMS 2.4.0 workflow), not this
  parsing step
- Structure-level identification tasks (CSI:FingerID database search) — this skill
  only compares top-ranked molecular formulas, not structural hits
- Datasets lacking a manually validated ground truth — accuracy and error-rate columns
  cannot be computed without it
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS
  role: Source of top-ranked molecular formula annotations (enumerates formula candidates
    from fragmentation trees) whose output files are parsed
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Source of reranked top-50 SIRIUS formula candidates via Gibbs sampling; output
    files parsed and compared against SIRIUS baselines
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS 2.4.0 (modified)
  role: Modified version plus patch file used upstream to process mzML/mzXML inputs
    and discard low-quality MS/MS spectra before annotation
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:fee524607a77b07cbcd96bb1c873be8c7de6abf46540a31087cc230f01e9d6cf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - OpenMS 2.4.0 (modified)
  - SIRIUS
  - ZODIAC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mass-spectrometry-annotation-file-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse deposited SIRIUS and ZODIAC molecular formula annotation result files and match top-ranked formulas against manually validated ground truth to compute per-dataset annotation accuracy and error rates across five biological LC-MS/MS datasets.

## When to use

Use when you have SIRIUS/ZODIAC annotation output files plus ground-truth formula lists (e.g. from MassIVE MSV000080502, MSV000081364) and need to reproduce or extend the molecular formula accuracy benchmark.

## When NOT to use

- Raw mzML/mzXML spectra that have not yet been annotated by SIRIUS/ZODIAC — run the annotation pipeline first (including the modified OpenMS 2.4.0 workflow), not this parsing step
- Structure-level identification tasks (CSI:FingerID database search) — this skill only compares top-ranked molecular formulas, not structural hits
- Datasets lacking a manually validated ground truth — accuracy and error-rate columns cannot be computed without it

## Inputs

- SIRIUS molecular formula annotation result files (deposited at bio.informatik.uni-jena.de/data/)
- ZODIAC reranked annotation result files (Gibbs sampling output)
- Ground-truth formula files per dataset from spectral library search and manual validation
- Dataset provenance metadata (dendroides MSV000080502, NIST1950 MSV000081364, tomato, diatoms, mice stool)

## Outputs

- CSV benchmark table: 5 datasets x 2 methods with annotation accuracy and incorrect-annotation rate columns
- Dendroides error-reduction factor (expected ~16.17-fold / reported 16.2-fold)
- Per-compound top-ranked formula strings from SIRIUS and ZODIAC

## How to apply

Download the deposited SIRIUS and ZODIAC results and ground-truth files from https://bio.informatik.uni-jena.de/data/. Parse the annotation files to extract, per compound per dataset, the top-ranked formula from SIRIUS (which enumerates formula candidates via fragmentation tree computation) and from ZODIAC (which reranks the top 50 SIRIUS candidates using Bayesian statistics via Gibbs sampling with a burn-in of 1,000 and sampling of 2,000 epochs). Match each top-ranked formula to the ground truth established by spectral library search and manual validation for the five datasets (dendroides, NIST1950, tomato, diatoms, mice stool). Compute per-dataset accuracy as the fraction of correct top-ranked formulas and the incorrect-annotation rate for both methods. Derive the dendroides error-reduction factor as the ratio of SIRIUS to ZODIAC incorrect rates, and write the 5 datasets x 2 methods table (accuracy and error-rate columns) to CSV.

## Related tools

- **SIRIUS** (Source of top-ranked molecular formula annotations (enumerates formula candidates from fragmentation trees) whose output files are parsed) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Source of reranked top-50 SIRIUS formula candidates via Gibbs sampling; output files parsed and compared against SIRIUS baselines) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS 2.4.0 (modified)** (Modified version plus patch file used upstream to process mzML/mzXML inputs and discard low-quality MS/MS spectra before annotation) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Evaluation signals

- Reproduce dendroides accuracies: SIRIUS 50.76% correct (197 evaluable ground truth compounds) vs ZODIAC-without-anchors 96.95%
- Reproduce per-dataset incorrect-annotation rates: NIST1950 8.51% vs 5.32%; tomato 4.44% vs 2.22%; diatoms 12.90% roughly halved; mice stool 30.23% vs 20.93%
- Computed dendroides error-reduction factor (ratio of incorrect rates) matches the reported ~16.17-fold (paper reports 16.2-fold)
- Every evaluable ground-truth compound is matched against exactly one top-ranked formula per method; no unmatched or duplicated compounds
- Output CSV contains exactly 5 datasets x 2 methods rows with both accuracy and error-rate columns populated

## Limitations

- Only compounds with an established ground truth (spectral library search plus manual validation) are evaluable; e.g. dendroides has 197 evaluable compounds
- Reproducibility requires the modified OpenMS 2.4.0 plus a separate patch file, not stock OpenMS
- ZODIAC results without anchors (library hits) are the reported benchmark; anchors have only a small effect on molecular formula annotations
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data, so high-quality adduct annotations must come from preprocessing

## Evidence

- [readme] SIRIUS is the java-based framework whose annotation output this skill parses.: [evidence span withheld — anchor status: external_doc]
