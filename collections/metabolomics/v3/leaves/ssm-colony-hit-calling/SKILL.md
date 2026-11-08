---
name: ssm-colony-hit-calling
description: Use when a colony-screening MALDI-TOF dataset (e.g. an S. cerevisiae
  SSM library) must be converted into per-colony fragment-ion ratios to identify variants
  producing shorter-chain fatty acyl products.
when_to_use_negative:
- Samples smaller than ~300 μm, where microMS rather than the macroMS workflow is
  recommended
- Analysis goals requiring full spectral processing rather than targeted ion-count
  extraction at a few known m/z values — macroMS provides only 'limited data analysis'
- Screening tasks without pre-specified target fragment ions or an unrankable ratio
  (no single-value discriminator per colony)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0621
tools:
- name: macroMS
  role: Web platform used to define colony targets from optical images, generate MALDI-TOF
    acquisition coordinates, and extract/rank ion-count ratios for hit calling; enables
    screening at 1–2 s per sample
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Predecessor optically guided MS software; reference point for samples smaller
    than 300 μm, which this skill does not cover
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:32dcf6b07cf55b1c500b83c03ee10782ffe27b09f0cccdf2f30b130130bbe293
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# ssm-colony-hit-calling

## Summary

Call 'hit' colonies from a MALDI-TOF screen of a Saccharomyces cerevisiae site saturation mutant (SSM) library by extracting fatty-acyl ion counts per colony spectrum with macroMS, computing per-colony C16:1/C18:1 ion-count ratios, ranking colonies, and flagging top hits (e.g. the G1250S mutant) with population and top-five statistics.

## When to use

Use when a colony-screening MALDI-TOF dataset (e.g. an S. cerevisiae SSM library) must be converted into per-colony fragment-ion ratios to identify variants producing shorter-chain fatty acyl products.

## When NOT to use

- Samples smaller than ~300 μm, where microMS rather than the macroMS workflow is recommended
- Analysis goals requiring full spectral processing rather than targeted ion-count extraction at a few known m/z values — macroMS provides only 'limited data analysis'
- Screening tasks without pre-specified target fragment ions or an unrankable ratio (no single-value discriminator per colony)

## Inputs

- MALDI-TOF mass spectra of individual colonies from an SSM library, keyed by colony position (e.g. 749 S. cerevisiae Gly1250 SSM colony spectra deposited from the macroMS screen)
- Target m/z values for fatty acyl fragment species (e.g. C16:1 and C18:1)
- Optional colony genotype/sequencing results for hit confirmation

## Outputs

- Per-colony C16:1/C18:1 ion-count ratio table (CSV)
- Population mean ± SD of ratios and mean ± SD of the top-ranked colonies (expected 3.8 ± 2.9 and 18.1 ± 1.1)
- Identified hit colonies with genotype calls (e.g. G1250S mutants)

## How to apply

Load the deposited MALDI-TOF spectra for all screened colonies (e.g. 749 S. cerevisiae FASII Gly1250 SSM colonies) into a data frame keyed by colony position. Use macroMS data analysis tools to extract ion counts at the m/z values of the target fragments (here C16:1 and C18:1 fatty acyl species) from each spectrum, then compute the per-colony ratio of C16:1 to C18:1 ion count. Rank colonies by this ratio, and compare the population statistics to expected values (population mean 3.8 ± 2.9; top-five mean 18.1 ± 1.1) to validate the computation. Confirm top-ranked colonies by genotyping/DNA sequencing (all five top colonies in the reference screen were G1250S mutants). Write the per-colony ratio table, the population/top-five mean±SD statistics, and the genotype identification to a CSV output file.

## Related tools

- **macroMS** (Web platform used to define colony targets from optical images, generate MALDI-TOF acquisition coordinates, and extract/rank ion-count ratios for hit calling; enables screening at 1–2 s per sample) — github.com/macroMS/macroMS_web
- **microMS** (Predecessor optically guided MS software; reference point for samples smaller than 300 μm, which this skill does not cover)

## Evaluation signals

- Computed population ratio mean matches 3.8 ± 2.9 on the deposited 749-colony Gly1250 dataset
- Top-five colony ratio mean matches 18.1 ± 1.1
- Top-ranked colony corresponds to the known G1250S mutant confirmed by DNA sequencing
- Per-colony ratio table keys align with colony positions and only the specified target m/z windows (C16:1, C18:1) contribute to the ratio
- Output CSV contains per-colony ratios plus population/top-five statistics and genotype identification

## Limitations

- No version information for macroMS is stated
- macroMS offers only limited data analysis/visualization; deeper statistical treatment must be done externally
- The upper limit of ~10,000 targets per run is presented without measurement details or uncertainty
- Comparison between macroMS and microMS is qualitative, with no quantitative benchmark

## Evidence

- [other] The five colonies with the highest C16:1 over 18:1 ratio had an average value of 18.1±1.1, and DNA sequencing demonstrated that all five isolated colonies were G1250S mutants.: [evidence span withheld — anchor status: not_found]
