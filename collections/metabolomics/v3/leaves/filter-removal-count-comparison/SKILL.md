---
name: filter-removal-count-comparison
description: Use when you must quantify how many spectra a cleaning pipeline removes
  with a given filter set versus an ablated filter set (e.g. without repair functions),
  to justify the filters' added value.
when_to_use_negative:
- When the goal is to produce a cleaned library for downstream use rather than to
  quantify filter impact — use the full library-cleaning pipeline with repair filters
  enabled instead
- When spectra already carry complete, validated SMILES/InChI/InChIKey annotations,
  so the repair-vs-no-repair distinction cannot change removal counts
- When the reference report for the with-repair run does not exist, since the skill
  is defined by comparing removal counts across the two configurations
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: matchms
  role: Provides the SpectrumProcessor, the library-cleaning filters, and the processing-report
    generation used to count spectra removed with and without repair functions
  repo: https://github.com/matchms/matchms
- name: Python
  role: Language in which the ablated pipeline is scripted and executed over the GNPS
    library
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:f06aefd133932d9809d986806df768e9b1c304f58e9d513dcaff58bbd5f275a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# filter-removal-count-comparison

## Summary

Runs the matchms library-cleaning pipeline with and without the annotation repair filters on the GNPS public mass spectral library and compares the resulting per-filter spectra-removal counts, to quantify how much the repair functions reduce spectrum removals.

## When to use

Use when you must quantify how many spectra a cleaning pipeline removes with a given filter set versus an ablated filter set (e.g. without repair functions), to justify the filters' added value.

## When NOT to use

- When the goal is to produce a cleaned library for downstream use rather than to quantify filter impact — use the full library-cleaning pipeline with repair filters enabled instead
- When spectra already carry complete, validated SMILES/InChI/InChIKey annotations, so the repair-vs-no-repair distinction cannot change removal counts
- When the reference report for the with-repair run does not exist, since the skill is defined by comparing removal counts across the two configurations

## Inputs

- Raw GNPS public mass spectral library (500,569 spectra, downloaded 21-08-2023)
- With-repair reference processing report (31,758 spectra removed, from the full pipeline run)
- matchms 0.26.4 pipeline configuration (YAML or SpectrumProcessor definition) with the repair filters removed

## Outputs

- cond_gnps_no_repair_report.txt processing report (spectra removed, metadata changed, peaks changed per filter)
- Total no-repair spectra-removal count (83,843)
- Ablation comparison against the with-repair removal count (31,758)

## How to apply

Load the raw GNPS public mass spectral library (500,569 spectra, downloaded 21-08-2023) into matchms 0.26.4. Build a custom SpectrumProcessor containing the require/remove filters but deliberately excluding the five repair filters (Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation, Derive Annotation From Compound Name). Execute the pipeline over all spectra with per-spectrum change logging, then generate the processing report, which 'gives an overview of the number of spectra that were changed in the library by each filter.' Compare the total spectra-removed count (83,843 without repairs, due to missing valid annotations or parent mass/annotation mismatches) against the with-repair reference of 31,758 removals. The rationale is ablation: if the no-repair removal count is strictly higher, the repair functions demonstrably rescue spectra that would otherwise be discarded.

## Related tools

- **matchms** (Provides the SpectrumProcessor, the library-cleaning filters, and the processing-report generation used to count spectra removed with and without repair functions) — https://github.com/matchms/matchms
- **Python** (Language in which the ablated pipeline is scripted and executed over the GNPS library)

## Examples

```
from matchms import SpectrumProcessor; processor = SpectrumProcessor("basic")  # custom processor with require/remove filters only, repair filters excluded
```

## Evaluation signals

- The spectra-removed count in cond_gnps_no_repair_report.txt is strictly greater than 31,758 (the with-repair reference), reproducing the paper's claim
- The no-repair removal count equals 83,843 spectra, consistent with removals driven by lacking a valid annotation or parent mass/annotation mismatch
- The processing report contains per-filter counts of spectra removed, metadata changed, and peaks changed
- The pipeline configuration (YAML file) verifiably excludes all five repair filters: Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation, and Derive Annotation From Compound Name
- The input spectrum count equals 500,569, confirming the full raw GNPS library was processed

## Limitations

- Repair functions can only fix annotation errors detectable from metadata (SMILES, adduct, parent mass); wrong chemical annotations consistent with the measured mass go unnoticed
- Filters that check whether measured fragments match the given annotation are not implemented — they are only proposed as future expansion
- The comparison is specific to the GNPS library snapshot of 21-08-2023 and matchms 0.26.4; removal counts will differ for other libraries (e.g. MassBank, MoNA) or library versions

## Evidence

- [methods] a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter.: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [results] Before cleaning, the GNPS library contained 500,569 spectra.: "Before cleaning, the GNPS library contained 500,569 spectra."
- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] Library cleaning. Runs all default filters, but in addition repairs errors in the annotations and requires complete annotations: "Library cleaning. Runs all default filters, but in addition repairs errors in the annotations and requires complete annotations"
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: "Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS)."
