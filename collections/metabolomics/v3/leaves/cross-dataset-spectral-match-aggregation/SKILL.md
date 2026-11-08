---
name: cross-dataset-spectral-match-aggregation
description: Use when you have KGMN-annotated unknown metabolites (e.g., from Demo_MetDNA2_NIST_urine_pos)
  and need to test whether their MS/MS spectra recur across public GNPS/MassIVE data
  sets as independent evidence for annotations lacking chemical standards.
when_to_use_negative:
- The metabolites are knowns already present in standard MS/MS spectral libraries
  — repository mining is intended for unknowns absent from such libraries
- The goal is discriminating structurally similar isomers of unknowns — MASST spectral
  matches cannot resolve isomers (KGMN itself cannot distinguish them effectively,
  e.g., isocitrate glucuronidation product isomers)
- The KGMN annotation step has not been run, so no annotated unknown MS/MS spectra
  exist to submit
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the MASST repository-mining job searching each unknown MS/MS spectrum
    against all public data sets in MassIVE and returning per-unknown matches
- name: MetDNA2Vis
  role: Provides the annotated unknown metabolites and MS/MS spectra from the KGMN
    results (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/) that are submitted
    to MASST
  repo: github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:cdba5c95a7489d9ba6f93d9ccbcf0092662be37631a74651b1b2773463512479
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MetDNA2Vis
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Cross-dataset spectral match aggregation

## Summary

Mines public metabolomics repositories by submitting putatively annotated unknown MS/MS spectra (precursor m/z + fragment peaks) from KGMN results to GNPS MASST, then aggregates recurrence per unknown across public data sets in MassIVE to corroborate annotations of metabolites absent from common MS/MS libraries.

## When to use

Use when you have KGMN-annotated unknown metabolites (e.g., from Demo_MetDNA2_NIST_urine_pos) and need to test whether their MS/MS spectra recur across public GNPS/MassIVE data sets as independent evidence for annotations lacking chemical standards.

## When NOT to use

- The metabolites are knowns already present in standard MS/MS spectral libraries — repository mining is intended for unknowns absent from such libraries
- The goal is discriminating structurally similar isomers of unknowns — MASST spectral matches cannot resolve isomers (KGMN itself cannot distinguish them effectively, e.g., isocitrate glucuronidation product isomers)
- The KGMN annotation step has not been run, so no annotated unknown MS/MS spectra exist to submit

## Inputs

- Annotated unknown metabolite list from KGMN annotation results (MetDNA2Vis package on Demo_MetDNA2_NIST_urine_pos)
- MS/MS spectra of the unknowns (precursor m/z and fragment peaks) in GNPS MASST submission format

## Outputs

- masst_recurrence_stats.csv with per-unknown fields: unknown identifier, data set count, data file count, species, sample type
- Prioritized list of recurrent unknowns for validation (e.g., 5 metabolites subsequently confirmed with synthetic chemical standards)

## How to apply

Load the annotated unknown list and MS/MS spectra from the KGMN annotation results (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/). Format each unknown spectrum (precursor m/z and fragment peaks) as required by GNPS MASST and submit each spectrum so it is searched against all public data sets in MassIVE. For each unknown, aggregate the matches by counting distinct data sets, data files, species, and sample types in which it recurs. Tabulate the recurrence statistics and save them (benchmark in this work: 187 unknowns recurrent in 351 data sets and 13,418 data files, spanning 10 species and 12 sample types). Recurrent unknowns can then be prioritized for confirmation, as done here for 5 metabolites validated with synthetic chemical standards.

## Related tools

- **GNPS** (Hosts the MASST repository-mining job searching each unknown MS/MS spectrum against all public data sets in MassIVE and returning per-unknown matches)
- **MetDNA2Vis** (Provides the annotated unknown metabolites and MS/MS spectra from the KGMN results (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/) that are submitted to MASST) — github.com/ZhuMetLab/MetDNA2Vis

## Evaluation signals

- masst_recurrence_stats.csv exists and contains fields for unknown identifier, data set count, data file count, species, and sample type
- Recurrence counts match the reported benchmark: 187 recurrent unknowns; 351 data sets; 13,418 data files; 10 species; 12 sample types
- Every unknown in the submitted list has an aggregated MASST result row (matched or zero-recurrence), with no dropped spectra
- Per-unknown data file counts are greater than or equal to the corresponding data set counts (each data set contains at least one matched file)

## Limitations

- Only five recurrent unknown metabolites were validated with synthetic standards via repository-mining, so recurrence alone corroborates rather than proves an annotation
- Validation of unknown annotations remains a grand challenge because unknowns are absent from spectral databases and lack chemical standards
- Caution is needed when applying the broader KGMN workflow to non-model organisms and environmental/exposomics samples; organism-specific databases are not yet incorporated

## Evidence

- [abstract] repository mining of recurrent unknowns benchmark: "we validate 5 metabolites that are absent in common MS/MS libraries through repository mining and synthesis of chemical standards."
- [discussion] recurrent unknowns confirmed with standards: "We used such an approach to discover five recurrent unknown metabolites, and further confirmed their annotations with synthetic chemical standards"
- [readme] MetDNA2Vis purpose: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [discussion] unknown validation challenge: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
