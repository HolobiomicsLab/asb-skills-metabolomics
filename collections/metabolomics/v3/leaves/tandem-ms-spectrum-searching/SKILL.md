---
name: tandem-ms-spectrum-searching
description: Use when a list of unknown metabolites with MS/MS spectra exists (e.g.,
  KGMN annotations of NIST urine positives) and the goal is to test whether those
  unknowns recur across public metabolomics repositories.
when_to_use_negative:
- The input is a peak/feature table without MS/MS spectra — MASST requires precursor
  m/z plus fragment peaks, so MS1-only unknowns cannot be searched.
- The unknowns are already validated with chemical standards or present in standard
  spectral libraries — repository mining adds value mainly for metabolites absent
  from common MS/MS libraries.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: MASST spectral search against all public MassIVE data sets to find where each
    unknown MS/MS spectrum recurs
- name: MetDNA2Vis
  role: Provides the KGMN annotation results and unknown MS/MS spectra for the Demo_MetDNA2_NIST_urine_pos
    dataset that feed the MASST searches
  repo: github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:e8bcda50faa6f5593da051a1e5626ee7a19649230b79e08a797e52f730d8e1cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MetDNA2Vis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tandem-ms-spectrum-searching

## Summary

Searches putatively annotated unknown MS/MS spectra (precursor m/z plus fragment peaks) against all public data in GNPS/MassIVE using the MASST spectral search tool, then aggregates where each unknown recurs. This turns single-run KGMN annotations into recurrence evidence that can support validation of unknown metabolites.

## When to use

Use when a list of unknown metabolites with MS/MS spectra exists (e.g., KGMN annotations of NIST urine positives) and the goal is to test whether those unknowns recur across public metabolomics repositories.

## When NOT to use

- The input is a peak/feature table without MS/MS spectra — MASST requires precursor m/z plus fragment peaks, so MS1-only unknowns cannot be searched.
- The unknowns are already validated with chemical standards or present in standard spectral libraries — repository mining adds value mainly for metabolites absent from common MS/MS libraries.

## Inputs

- Annotated unknown metabolite list from KGMN/MetDNA2Vis annotation results
- Per-unknown MS/MS spectra (precursor m/z and fragment peaks) from the demo dataset Demo_MetDNA2_NIST_urine_pos
- GNPS MASST-compatible spectrum format prepared from each unknown's MS/MS data

## Outputs

- MASST match results per unknown against public MassIVE/GNPS data sets
- Recurrence statistics table (masst_recurrence_stats.csv) with unknown identifier, data set count, data file count, species, and sample type

## How to apply

Load the annotated unknown list and MS/MS spectra from the KGMN annotation results (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/), and prepare each unknown spectrum (precursor m/z and fragment peaks) in the format required by GNPS MASST. Submit each spectrum so MASST searches it against all public data sets deposited in MassIVE. Aggregate MASST matches per unknown, counting distinct data sets, data files, species, and sample types in which each unknown appears. Tabulate recurrence statistics and save them (e.g., masst_recurrence_stats.csv) so recurrent unknowns can be prioritized for confirmation with synthetic chemical standards, as done in the paper for five recurrent unknown metabolites.

## Related tools

- **GNPS** (MASST spectral search against all public MassIVE data sets to find where each unknown MS/MS spectrum recurs)
- **MetDNA2Vis** (Provides the KGMN annotation results and unknown MS/MS spectra for the Demo_MetDNA2_NIST_urine_pos dataset that feed the MASST searches) — github.com/ZhuMetLab/MetDNA2Vis

## Evaluation signals

- masst_recurrence_stats.csv exists and contains fields for unknown identifier, data set count, data file count, species, and sample type.
- Recurrence counts match the reported benchmark: 187 unknowns recurrent in 351 data sets and 13,418 data files, spanning 10 species and 12 sample types.
- Every submitted unknown has a precursor m/z and at least a fragment peak list in MASST-compatible format before searching.
- Per-unknown aggregates are counts of distinct data sets/files/species/sample types (not raw match rows), so duplicates within one data set do not inflate counts.

## Limitations

- Recurrence alone does not confirm structure: only five recurrent unknown metabolites were validated with synthetic chemical standards via repository mining.
- The MASST search surface reflects what is deposited in MassIVE/GNPS; organism-specific databases and non-model organisms are not covered, so caution is needed for non-model organisms and environmental/exposomics samples.
- KGMN/KGMN-derived unknowns cannot effectively discriminate structurally similar isomers, so recurrent matches may not resolve isomeric identities.

## Evidence

- [methods] workflow: submission to MASST: [evidence span withheld — anchor status: not_found]
- [results] reported recurrence finding: [evidence span withheld — anchor status: not_found]
- [discussion] validation of recurrent unknowns: "We used such an approach to discover five recurrent unknown metabolites, and further confirmed their annotations with synthetic chemical standards"
- [intro] GNPS as the MS/MS network resource: "The most prominent network approach is molecular networking (MN) in GNPS, which is an MS/MS similarity network"
