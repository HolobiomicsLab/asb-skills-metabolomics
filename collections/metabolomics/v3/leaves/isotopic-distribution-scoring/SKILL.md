---
name: isotopic-distribution-scoring
description: Use when you need to quantify how well saturation repair reconstructed
  the isotopic profile of detector-saturated high-abundance ions in LC–DT IM-MS data,
  by comparing repaired versus theoretical isotope envelopes.
when_to_use_negative:
- Input spectra are already peak-picked/centroided rather than raw profile-level,
  since saturation repair and isotopic scoring operate on profile-level data
- Ions are not saturated — i.e. all isotopic peaks fall well below the 40%-of-abundance-limit
  threshold — leaving no truncated envelope to score
- No theoretical isotopic distribution can be computed (e.g. unknown elemental composition),
  since the Pearson-correlation score requires a ChemCalc reference
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Computes the Pearson correlation between repaired and theoretical isotopic
    distributions (the isotopic score)
- name: ChemCalc
  role: Generates the theoretical isotopic distributions used as the reference envelope
    for scoring
- name: PNNL PreProcessor
  role: Produces the saturation-repaired MS-files whose isotopic distributions are
    then scored
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:410951677d2c6c4a068707c7e47b6457dbe958257d3b981f218ff21f8fd1e358
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ChemCalc
  - PNNL PreProcessor
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# isotopic-distribution-scoring

## Summary

Scores the quality of saturation-repaired LC–IM-MS spectra by correlating the repaired isotopic distribution of saturated high-abundance ions against theoretical distributions computed with ChemCalc in R, verifying that repair tripled monoisotopic peak height and cut mass error from 10.6 to 1.9 pm.

## When to use

Use when you need to quantify how well saturation repair reconstructed the isotopic profile of detector-saturated high-abundance ions in LC–DT IM-MS data, by comparing repaired versus theoretical isotope envelopes.

## When NOT to use

- Input spectra are already peak-picked/centroided rather than raw profile-level, since saturation repair and isotopic scoring operate on profile-level data
- Ions are not saturated — i.e. all isotopic peaks fall well below the 40%-of-abundance-limit threshold — leaving no truncated envelope to score
- No theoretical isotopic distribution can be computed (e.g. unknown elemental composition), since the Pearson-correlation score requires a ChemCalc reference

## Inputs

- Preprocessed (saturation-repaired) LC–DT IM-MS Agilent .d MS-file from the PNNL PreProcessor, profile mode, '_SR' suffix
- Original (unrepaired) LC–DT IM-MS Agilent .d MS-file for baseline comparison
- Theoretical isotopic distributions computed with ChemCalc for the lipid ions of interest

## Outputs

- Pearson correlation coefficient (isotopic score) per repaired ion comparing repaired vs theoretical isotopic distribution
- Quantified isotopic-peak recovery metrics, e.g. monoisotopic peak height fold-change and mass-error shift in pm

## How to apply

After running the PNNL PreProcessor saturation repair module on Agilent .d LC-DT IM-MS triplicate files from a human plasma lipid extract (saturation repair threshold set to 40% of the abundance limit), extract the isotopic distribution of the repaired saturated ions and compute its theoretical counterpart with ChemCalc. In R, calculate the Pearson correlation between the repaired isotopic distribution and the theoretical distribution as the isotopic score. In the reference study, successful repair of a saturated ion tripled the monoisotopic peak height and reduced the mass error from 10.6 to 1.9 pm; scores are therefore judged against these reported recovery metrics and by expert review of the correlation magnitude.

## Related tools

- **R** (Computes the Pearson correlation between repaired and theoretical isotopic distributions (the isotopic score))
- **ChemCalc** (Generates the theoretical isotopic distributions used as the reference envelope for scoring)
- **PNNL PreProcessor** (Produces the saturation-repaired MS-files whose isotopic distributions are then scored) — https://omics.pnl.gov/software/pnnl-preprocessor

## Examples

```
cor(as.numeric(repaired_iso), as.numeric(theoretical_iso), method="pearson")  # isotopic score in R
```

## Evaluation signals

- Repaired monoisotopic peak height is approximately tripled relative to the saturated (pre-repair) peak height
- Mass error of the repaired ion drops from ~10.6 pm to ~1.9 pm
- Pearson correlation between the repaired and ChemCalc theoretical isotopic distribution is high (close to 1), assessed by expert review
- The '_SR' output file exists, retains the same instrument format, and structurally matches the deposited preprocessed file in MSV000087295

## Limitations

- The isotopic score is a Pearson correlation judged by expert review rather than an automated pass/fail criterion
- Requires the ion to appear unsaturated at later LC or IM time to recover a reference isotopic distribution; fully saturated isotopic envelopes cannot be scored
- Scoring depends on external theoretical distributions from ChemCalc; it does not work on purely empirical isotope ratios alone

## Evidence
