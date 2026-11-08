---
name: unknown-metabolite-recurrence-analysis
description: Use when a list of putatively annotated unknown metabolites with precursor
  m/z and MS/MS spectra (e.g., from KGMN on NIST urine data) needs external corroboration
  via public-data recurrence rather than spectral-library or standard-based validation.
when_to_use_negative:
- The metabolites are knowns already matched against standard spectral libraries —
  recurrence mining targets unknowns absent from MS/MS libraries
- The goal is discriminating structurally similar isomers (e.g., glucuronidation product
  isomers of isocitrate), which the article states cannot be effectively distinguished
  by KGMN/MS2 similarity approaches
- No MS/MS spectra are available for the unknowns (MS1-only features cannot be searched
  via MASST)
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: MASST search of each unknown MS/MS spectrum against all public datasets deposited
    in MassIVE for repository mining
- name: MetDNA2Vis
  role: Visualize, reproduce and investigate putatively annotated known and unknown
    metabolites from KGMN, and handle/export MS2 spectral data of unknowns prior to
    MASST submission
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Spectral data conversion utilities (installed alongside MetDNA2Vis) used to
    prepare MS/MS spectra of unknowns
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:28d1bccef8260de57da55959be42ae039d0c6b72d663c437d85f74bbe5901817
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - GNPS
  - MetDNA2Vis
  - SpectraTools
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# unknown-metabolite-recurrence-analysis

## Summary

Repository-mining of putatively annotated unknown MS/MS spectra by submitting them to GNPS MASST against all public GNPS/MassIVE datasets, then aggregating recurrence statistics (datasets, files, species, sample types) to corroborate KGMN unknown annotations absent from standard spectral libraries.

## When to use

Use when a list of putatively annotated unknown metabolites with precursor m/z and MS/MS spectra (e.g., from KGMN on NIST urine data) needs external corroboration via public-data recurrence rather than spectral-library or standard-based validation.

## When NOT to use

- The metabolites are knowns already matched against standard spectral libraries — recurrence mining targets unknowns absent from MS/MS libraries
- The goal is discriminating structurally similar isomers (e.g., glucuronidation product isomers of isocitrate), which the article states cannot be effectively distinguished by KGMN/MS2 similarity approaches
- No MS/MS spectra are available for the unknowns (MS1-only features cannot be searched via MASST)

## Inputs

- KGMN/MetDNA2 annotation table of putatively annotated unknown metabolites (e.g., from Demo_MetDNA2_NIST_urine_pos)
- MS/MS spectra (precursor m/z and fragment peak lists) for each unknown peak (e.g., M182T541, M262T526)
- Access to GNPS MASST searching against GNPS/MassIVE public datasets

## Outputs

- masst_recurrence_stats.csv with per-unknown fields: unknown identifier, dataset count, data file count, species, sample type
- Aggregated recurrence summary (benchmark: 187 unknowns; 351 datasets; 13,418 data files; 10 species; 12 sample types)
- Shortlist of recurrent unknowns prioritized for follow-up validation (e.g., synthesis of chemical standards)

## How to apply

Load the annotated unknown metabolite list and their MS/MS spectra from the KGMN annotation results (e.g., the Demo_MetDNA2_NIST_urine_pos /06_visualization/ output, which MetDNA2Vis can visualize and reproduce). Prepare each unknown spectrum (precursor m/z plus fragment peaks) in the format required by GNPS MASST, using SpectraTools/MetDNA2Vis utilities such as convertSpectraData for spectral handling. Submit each unknown spectrum to MASST so it is searched against all public datasets deposited in MassIVE, mirroring the article's repository-mining validation strategy for metabolites absent in common MS/MS libraries. Aggregate MASST matches per unknown, counting distinct datasets, data files, species, and sample types in which each unknown recurs, and tabulate them (benchmark: 187 of the NIST urine unknowns recurrent in 351 datasets, 13,418 files, 10 species, 12 sample types).

## Related tools

- **GNPS** (MASST search of each unknown MS/MS spectrum against all public datasets deposited in MassIVE for repository mining)
- **MetDNA2Vis** (Visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN, and handle/export MS2 spectral data of unknowns prior to MASST submission) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Spectral data conversion utilities (installed alongside MetDNA2Vis) used to prepare MS/MS spectra of unknowns) — https://github.com/ZhuMetLab/MetDNA2Vis

## Evaluation signals

- The summary table masst_recurrence_stats.csv exists and contains fields for unknown identifier, dataset count, data file count, species, and sample type
- Recurrence counts reproduce the reported benchmark: 187 recurrent unknowns, 351 datasets, 13,418 data files
- Recurrence coverage spans 10 species and 12 sample types as reported
- Every submitted unknown had precursor m/z and fragment peaks formatted for MASST (no spectra skipped or silently dropped)
- Only unknowns absent from common MS/MS libraries are selected, mirroring the article's validation strategy for 5 metabolites corroborated by repository mining plus chemical-standard synthesis

## Limitations

- Repository mining alone is putative; in the article only five recurrent unknown metabolites were further confirmed with synthetic chemical standards
- Recurrence indicates spectral similarity in public data but cannot effectively discriminate structurally similar isomers of unknown metabolites
- Validation of unknown annotations remains a grand challenge because unknowns are not included in spectral databases and have no chemical standards

## Evidence

- [results] Repository-mining via MASST showed that 187 of the NIST urine unknowns were recurrent, appearing in 351 data sets comprising 13,418 data files: [evidence span withheld — anchor status: not_found]
- [abstract] we validate 5 metabolites that are absent in common MS/MS libraries through repository mining and synthesis of chemical standards: "we validate 5 metabolites that are absent in common MS/MS libraries through repository mining and synthesis of chemical standards"
- [discussion] We used such an approach to discover five recurrent unknown metabolites, and further confirmed their annotations with synthetic chemical standards: "We used such an approach to discover five recurrent unknown metabolites, and further confirmed their annotations with synthetic chemical standards"
- [readme] The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
