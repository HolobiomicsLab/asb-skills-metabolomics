---
name: extracted-ion-chromatogram-peak-quality-assessment
description: Use when you have rescued features from a DaDIA (DDA-rescued-in-DIA)
  workflow and need to quantify what fraction carry genuine chromatographic peaks
  versus noise before reporting rescue metrics.
when_to_use_negative:
- When features have already passed an automated blank or intensity filter and no
  further shape validation is needed
- When the goal is de novo peak picking from raw DDA data rather than validating rescued
  DDA features within a DaDIA workflow
- When chromatographic data are non-LC-MS (e.g. direct infusion) and EICs are not
  defined
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DaDIA.R
  role: Produces the combined DaDIA feature table whose rescued DDA features are subsequently
    validated by EIC inspection
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Extracts metabolic features from mzXML and supports EIC extraction for manual
    peak shape review
- name: R
  role: Runtime environment (R 4.0+) for executing DaDIA.R and EIC visualisation scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:d59011df58dd2dd8c453e93d0f96fc4b18469c051e5eec2cf111f08ccf27aad1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Extracted-ion chromatogram peak quality assessment

## Summary

Manual inspection of extracted-ion chromatograms (EICs) of rescued features in DaDIA datasets to score peak shape quality, yielding true-positive and false-positive rates that quantify the reliability of the DDA feature rescue step.

## When to use

Use when you have rescued features from a DaDIA (DDA-rescued-in-DIA) workflow and need to quantify what fraction carry genuine chromatographic peaks versus noise before reporting rescue metrics.

## When NOT to use

- When features have already passed an automated blank or intensity filter and no further shape validation is needed
- When the goal is de novo peak picking from raw DDA data rather than validating rescued DDA features within a DaDIA workflow
- When chromatographic data are non-LC-MS (e.g. direct infusion) and EICs are not defined

## Inputs

- Rescued DDA feature list from DaDIA.R (m/z, retention time, intensity per sample)
- Raw mzXML files from urine RP(+) LC-MS runs (Bruker Impact II UHR-QqTOF coupled to Agilent 1290 LC, Waters ACQUITY UPLC BEH C18 column)
- XCMS 3.11.4+ / R 4.0+ / metaMS 1.25.1+ processing environment

## Outputs

- Per-feature peak shape classification (good / poor-noisy / undetermined)
- Aggregate metrics: average number of rescued DDA features (e.g. 344), true-positive rate (e.g. 0.71), false-positive rate (e.g. 0.17), and undetermined fraction (e.g. 0.12)

## How to apply

After DaDIA.R combines DDA and DIA feature lists, extract the EIC of each rescued DDA feature (m/z and retention time pair) from the raw urine mzXML files in the XCMS output. Visually classify each EIC into good chromatographic peak shape (true positive), poor or noisy shape (false positive), or undetermined when the chromatogram is ambiguous. Tally these counts across all samples and compute average rates — in the urine RP(+) study the published outcome was 71% true-positive, 17% false-positive, and 12% undetermined for an average of 344 rescued DDA features per sample. The rationale is that rescue alone does not guarantee a real peak; EIC shape is the arbiter of whether the merged feature reflects a detectable metabolite.

## Related tools

- **DaDIA.R** (Produces the combined DaDIA feature table whose rescued DDA features are subsequently validated by EIC inspection) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Extracts metabolic features from mzXML and supports EIC extraction for manual peak shape review)
- **R** (Runtime environment (R 4.0+) for executing DaDIA.R and EIC visualisation scripts)

## Examples

```
Rscript -e 'source("DaDIA.R"); plotEIC(rescued_features, files=list.files("mzxml_dir", "\\.mzXML$"))'
```

## Evaluation signals

- True-positive + false-positive + undetermined fractions sum to 1.0 (e.g. 0.71 + 0.17 + 0.12 = 1.00)
- Average rescued-feature count matches the published value of 344 for the urine RP(+) DaDIA dataset
- Every EIC classification is traceable to a concrete chromatogram image, not a spot check
- Rescued-feature count stays well below the ~1591 total DDA features, consistent with ~22% recovery
- Result is reported per-sample and averaged, matching the paper's per-sample averaging convention

## Limitations

- Manual EIC classification is subjective; ~12% of rescued features remained undetermined even with expert inspection
- Rates are validated only for urine RP(+) mode; other matrices, columns (e.g. ZIC-pHILIC), or ionization modes may shift the TPR/FPR balance
- Requires visual inspection — not fully automatable without a separate peak-shape scoring model

## Evidence

- [results] Manual EIC inspection rescued features TPR/FPR: [evidence span withheld — anchor status: not_found]
