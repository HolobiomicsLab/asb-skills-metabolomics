---
name: yeast-mutant-library-ratio-ranking
description: Use when you have MALDI-TOF spectra for many colonies (e.g., 749 colonies
  at a site-saturation library position) and need per-colony ratios of fatty acyl
  fragments like C16:1 vs C18:1 to rank and identify mutants.
when_to_use_negative:
- Samples are smaller than 300 μm — the paper states microMS is more effective for
  such targets and macroMS is designed for macroscopic (>300 μm) objects
- The target property is not an interpretable per-sample ratio of specific m/z species;
  macroMS provides only limited data analysis, not full statistical modeling or identification
  pipelines
- Only one or a few samples need analysis; the workflow is built for high-throughput
  screening of large libraries (up to ~10,000 targets)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0621
tools:
- name: macroMS
  role: Web platform used to find/filter colony targets from optical images, generate
    MS device coordinates, and perform the limited data analysis (ion-count extraction)
    that yields the per-colony ratio table
  repo: github.com/macroMS/macroMS_web
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:f42c588ef37c45336816531ecf4a16209e28cb5693ad9f5680e66bf493724ebe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# yeast-mutant-library-ratio-ranking

## Summary

Rank yeast SSM library colonies by an ion-count ratio extracted from MALDI-TOF spectra via macroMS, then validate top hits against known genotypes. Used to surface genotype-defining phenotypes (e.g., C16:1/C18:1 ratio flagging G1250S mutants in the S. cerevisiae FASII Gly1250 library).

## When to use

Use when you have MALDI-TOF spectra for many colonies (e.g., 749 colonies at a site-saturation library position) and need per-colony ratios of fatty acyl fragments like C16:1 vs C18:1 to rank and identify mutants.

## When NOT to use

- Samples are smaller than 300 μm — the paper states microMS is more effective for such targets and macroMS is designed for macroscopic (>300 μm) objects
- The target property is not an interpretable per-sample ratio of specific m/z species; macroMS provides only limited data analysis, not full statistical modeling or identification pipelines
- Only one or a few samples need analysis; the workflow is built for high-throughput screening of large libraries (up to ~10,000 targets)

## Inputs

- MALDI-TOF mass spectra for each colony in a site-saturation mutant library (e.g., 749 S. cerevisiae colonies at the Gly1250 site)
- colony position keys mapping spectra to plate coordinates (from macroMS device geometry files)
- m/z values defining the fatty acyl species to compare (e.g., C16:1 and C18:1)

## Outputs

- per-colony ratio table (colony ID/position, C16:1 and C18:1 ion counts, ratio)
- population-level summary statistics (mean and SD, e.g., 3.8 ± 2.9)
- top-k colony statistics (mean and SD, e.g., 18.1 ± 1.1 for the top five)
- genotype identification result (e.g., which colony harbors the G1250S mutation)
- CSV file consolidating the ratio table and summary statistics

## How to apply

Load the deposited MALDI-TOF spectra (one per colony, keyed by colony position from the macroMS-generated geometry) into a data frame. For each colony, use macroMS data analysis tools to extract ion counts at the target m/z values for the fatty acyl species of interest — here C16:1 and C18:1 — then compute the per-colony ratio of C16:1 to C18:1 ion count. Rank colonies by this ratio; compute the population mean and standard deviation as a baseline (expected ~3.8 ± 2.9 across 749 colonies) and the mean and SD of the top-ranked hits (expected ~18.1 ± 1.1 for the top five). Cross-check the identity of the top-ranked colonies against known genotypes by DNA sequencing records; in the Gly1250 screen, all five top colonies were G1250S mutants. Write the per-colony ratio table and summary statistics (population, top-k, genotype identification) to a CSV output file.

## Related tools

- **macroMS** (Web platform used to find/filter colony targets from optical images, generate MS device coordinates, and perform the limited data analysis (ion-count extraction) that yields the per-colony ratio table) — github.com/macroMS/macroMS_web

## Examples

```
# Per-colony ratio ranking over 749 Gly1250 spectra
ratios = df.assign(r=lambda d: d['C16:1_ion_count'] / d['C18:1_ion_count']).sort_values('r', ascending=False)
ratios.to_csv('gly1250_colony_ratios.csv', index=False)
```

## Evaluation signals

- Population mean/SD of the per-colony ratio matches the reported value of 3.8 ± 2.9
- Top-five colonies' mean ratio matches 18.1 ± 1.1
- Top-ranked colony corresponds to the known G1250S mutant genotype as confirmed by DNA sequencing
- Per-colony ratio table row count equals the number of assayed colonies (749 for the Gly1250 library)
- Output CSV contains both per-colony ratios and the population/top-k summary statistics

## Limitations

- macroMS provides only limited data analysis; deeper statistics or de novo identification must be done externally
- No version information for macroMS is stated in the article
- Screening rate and capacity are bounded (1–2 s/sample, upper limit ~10,000 targets) with no quantitative uncertainty reported
- The comparison to microMS is qualitative with no quantitative benchmark given

## Evidence

- [results] The five colonies with the highest C16:1 over 18:1 ratio had an average value of 18.1±1.1, and DNA sequencing demonstrated that all five isolated colonies were G1250S mutants.: [evidence span withheld — anchor status: not_found]
- [other] Compute the per-colony ratio of C16:1 ion count to C18:1 ion count.: [evidence span withheld — anchor status: not_found]
