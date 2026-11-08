---
name: matching-rate-statistics-tabulation
description: Use when query MS/MS spectra acquired on a different Orbitrap instrument
  (e.g. Exploris 480) must be matched against a multi-collision-energy reference database
  and reported as matched/total percentages per ionization mode.
when_to_use_negative:
- Input spectra were acquired on the same instrument as the reference library and
  the goal is candidate ranking/identification rather than portability statistics
  — use longitudinal-profile identification workflow (001-007.ipynb metabolite identification
  cells) instead.
- The query spectra are single-collision-energy MS/MS with no multi-CE longitudinal
  fragment profiles to compute.
- The task is building or curating the reference database itself (CQMU database construction)
  rather than scoring query spectra against it.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Jaccard similarity algorithm
  role: Scores each query-to-reference spectral pair, weighing predominant and minute
    fragments independently, before cutoff criteria are applied
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Library of MS data-processing methods (peak filtering, neutral-loss removal,
    file operations) used to preprocess query spectra
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Sequential Jupyter pipeline cells spanning spectrum extraction through isomer
    similarity calculations and identification
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: conda
  role: Creates the python=3.8.13 environment (longprof) in which the matching-rate
    tabulation runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:7c6e35a1f32ee5bc85aa953ecfc7971043834f653b72a4ca520084bf37fb6779
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# matching-rate-statistics-tabulation

## Summary

Computes inter-instrument metabolite matching rates by scoring Exploris 480 query MS/MS spectra against CQMD reference spectra (~180,000 MS/MS spectra at 13 collision energies and stepped NCE 20/40/60%) with the optimized Jaccard similarity algorithm, then tabulating strict- and relaxed-cutoff match counts per mode.

## When to use

Use when query MS/MS spectra acquired on a different Orbitrap instrument (e.g. Exploris 480) must be matched against a multi-collision-energy reference database and reported as matched/total percentages per ionization mode.

## When NOT to use

- Input spectra were acquired on the same instrument as the reference library and the goal is candidate ranking/identification rather than portability statistics — use longitudinal-profile identification workflow (001-007.ipynb metabolite identification cells) instead.
- The query spectra are single-collision-energy MS/MS with no multi-CE longitudinal fragment profiles to compute.
- The task is building or curating the reference database itself (CQMU database construction) rather than scoring query spectra against it.

## Inputs

- 150 CP-mode and 134 CN-mode Exploris 480 query MS/MS spectra from an independent laboratory
- CQMD in-house reference spectral database (~180,000 MS/MS spectra of >4500 metabolites at 13 collision energies and stepped NCE 20/40/60%)
- Strict (Figure 2 ROC-derived) and 0.8-fold relaxed cutoff criteria

## Outputs

- Per-mode matched counts and percentages under strict criteria (e.g. 143/150 = 95.33% CP; 123/134 = 91.79% CN)
- Per-mode matched counts and percentages under 0.8-fold relaxed cutoffs (approaching 100%)
- interinstrument_matching_results.csv summary table

## How to apply

Load the 150 CP-mode and 134 CN-mode query spectra and the CQMD reference database via mzbatch.py within the 001-007.ipynb pipeline (conda env longprof, python=3.8.13). Preprocess each query spectrum by removing peaks with relative intensity below 1%, filtering fragments corresponding to common neutral losses, and running fragment rationality assessment. Compute longitudinal fragment profiles (relative ion intensity of each fragment across multiple distinct CEs) for query and reference spectra and score each pair with the optimized Jaccard similarity algorithm weighing predominant and minute fragments independently. Call matches using the Figure 2 cutoff criteria derived from ROC analysis of thousands of isomers in CQMD, then repeat with 0.8-fold relaxed cutoffs. Tabulate matched/total for each mode (150 CP, 134 CN) under both criteria and save as interinstrument_matching_results.csv.

## Related tools

- **Jaccard similarity algorithm** (Scores each query-to-reference spectral pair, weighing predominant and minute fragments independently, before cutoff criteria are applied) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Library of MS data-processing methods (peak filtering, neutral-loss removal, file operations) used to preprocess query spectra) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Sequential Jupyter pipeline cells spanning spectrum extraction through isomer similarity calculations and identification) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **conda** (Creates the python=3.8.13 environment (longprof) in which the matching-rate tabulation runs)

## Examples

```
git clone https://github.com/yourusername/LongitudinalProf-MSMS-Method.git; conda create -n longprof python=3.8.13; pip install -r requirements.txt
```

## Evaluation signals

- Strict-criteria rates reproduce Figure 2 values exactly: 95.33% of 150 CP spectra matched (=143) and 91.79% of 134 CN spectra matched (=123).
- 0.8-fold relaxed cutoffs drive the matching rate to nearly 100% in both CP and CN modes.
- Matched/total denominators must equal the expected totals of 150 CP and 134 CN spectra.
- Preprocessing invariants hold: peaks below 1% relative intensity are absent from scored spectra and common-neutral-loss fragments are removed.
- A single summary artifact (interinstrument_matching_results.csv) exists containing counts and percentages for both strict and relaxed criteria across both modes.

## Limitations

- CQMD reference spectra come from an Exploris 120 while query spectra are Exploris 480; cutoffs were ROC-derived on CQMD isomers, and CQMD itself is in-house with no stated public accession.
- No changelog exists for the repository, and supplementary result artifacts (Figures 5e–h, S6, Excel S9–S13) have unconfirmed availability.
- README notes the data are 'intended for experimental reference only and should not be directly interpreted or used to guide human medicine', provided 'as is' without warranty.

## Evidence

- [methods] discussion-methods quote: [evidence span withheld — anchor status: not_found]
