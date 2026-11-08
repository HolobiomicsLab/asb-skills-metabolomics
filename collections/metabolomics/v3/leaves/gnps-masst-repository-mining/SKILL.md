---
name: gnps-masst-repository-mining
description: Use when KGMN (or similar) has produced putatively annotated unknown
  metabolites with precursor m/z and MS/MS spectra and you need to test whether they
  recur in public metabolomics repositories (GNPS/MassIVE) as an orthogonal validation
  signal.
when_to_use_negative:
- The metabolites are knowns already covered by standard spectral libraries, so library
  searching (rather than repository mining) suffices for annotation or validation
- The task is constructing or visualizing the KGMN subnetworks themselves (MetDNA2Vis
  reconstructNetwork1/2/3, retrieveSubNetwork1/2/3, mergeSubnetwork) rather than testing
  recurrence of already-annotated unknowns
- The input spectra lack precursor m/z or MS/MS fragment peaks, making MASST matching
  impossible
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: MASST searches each unknown MS/MS spectrum against all public data sets deposited
    in MassIVE, returning matches per unknown
- name: MetDNA2Vis
  role: Reproduces and exports the putatively annotated known and unknown metabolites
    from KGMN that serve as the query spectra for MASST
  repo: github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:633d2b6bea41fdf729f345438c63195e02a7865d7d67377a7aad13dd7281a426
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MetDNA2Vis
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# gnps-masst-repository-mining

## Summary

Repository-mining of putatively annotated unknown metabolites: each KGMN-annotated unknown MS/MS spectrum (e.g., from the NIST urine Demo_MetDNA2_NIST_urine_pos dataset) is submitted to GNPS MASST and searched against all public data sets in MassIVE to quantify recurrence across data sets, data files, species, and sample types. This leverages public data as a validation route for unknowns that lack chemical standards.

## When to use

Use when KGMN (or similar) has produced putatively annotated unknown metabolites with precursor m/z and MS/MS spectra and you need to test whether they recur in public metabolomics repositories (GNPS/MassIVE) as an orthogonal validation signal.

## When NOT to use

- The metabolites are knowns already covered by standard spectral libraries, so library searching (rather than repository mining) suffices for annotation or validation
- The task is constructing or visualizing the KGMN subnetworks themselves (MetDNA2Vis reconstructNetwork1/2/3, retrieveSubNetwork1/2/3, mergeSubnetwork) rather than testing recurrence of already-annotated unknowns
- The input spectra lack precursor m/z or MS/MS fragment peaks, making MASST matching impossible

## Inputs

- KGMN-annotated unknown metabolite list with MS/MS spectra (precursor m/z and fragment peaks) from the Demo_MetDNA2_NIST_urine_pos dataset
- Unknown MS/MS spectra formatted in GNPS MASST input format

## Outputs

- masst_recurrence_stats.csv with per-unknown recurrence fields (unknown identifier, data set count, data file count, species, sample type)
- Aggregated recurrence statistics (benchmark: 187 recurrent unknowns in 351 data sets and 13,418 data files, spanning 10 species and 12 sample types)
- Shortlist of recurrent unknown metabolites prioritized for validation (e.g., the five confirmed with synthetic chemical standards)

## How to apply

Load the annotated unknown metabolite list and their MS/MS spectra from the KGMN annotation results of the Demo_MetDNA2_NIST_urine_pos dataset (the MetDNA2Vis package reproduces and investigates these putatively annotated unknowns). Format each unknown spectrum — precursor m/z plus fragment peaks — as required by GNPS MASST, then submit each spectrum so it is searched against all public data sets deposited in MassIVE. Aggregate the MASST matches per unknown, counting distinct data sets, data files, species, and sample types in which each unknown recurs. In the article's benchmark, 187 of the NIST urine unknowns were recurrent, appearing in 351 data sets comprising 13,418 data files across 10 species and 12 sample types. Tabulate the recurrence statistics and save them as masst_recurrence_stats.csv, which also flags candidate recurrent unknowns (the paper followed up on five such metabolites with synthetic standards).

## Related tools

- **GNPS** (MASST searches each unknown MS/MS spectrum against all public data sets deposited in MassIVE, returning matches per unknown)
- **MetDNA2Vis** (Reproduces and exports the putatively annotated known and unknown metabolites from KGMN that serve as the query spectra for MASST) — github.com/ZhuMetLab/MetDNA2Vis

## Evaluation signals

- masst_recurrence_stats.csv exists and contains fields for unknown identifier, data set count, data file count, species, and sample type
- Aggregated recurrence counts match the reported benchmark: 187 recurrent unknowns across 351 data sets and 13,418 data files, spanning 10 species and 12 sample types
- Every submitted unknown spectrum carries both a precursor m/z and MS/MS fragment peaks in MASST-compatible format
- The recurrent-unknown shortlist corresponds to metabolites absent from common MS/MS libraries, consistent with the paper's five repository-mining-validated metabolites

## Limitations

- Recurrence alone does not confirm identity: only five recurrent unknown metabolites were validated with synthetic chemical standards via repository-mining
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites, so MASST matches may inherit this ambiguity
- Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge
- Caution is needed for non-model organisms and environmental/exposomics samples because organism-specific databases are not yet incorporated

## Evidence

- [abstract] we validate 5 metabolites that are absent in common MS/MS libraries through repository mining and synthesis of chemical standards.: "we validate 5 metabolites that are absent in common MS/MS libraries through repository mining and synthesis of chemical standards."
- [discussion] We used such an approach to discover five recurrent unknown metabolites, and further confirmed their annotations with synthetic chemical standards: "We used such an approach to discover five recurrent unknown metabolites, and further confirmed their annotations with synthetic chemical standards"
- [discussion] Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
- [readme] The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN.: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
