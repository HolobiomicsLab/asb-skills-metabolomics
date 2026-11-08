---
name: nist-compatible-msp-file-generation
description: Use when predicted retention times must be exported into NIST-format
  .msp spectral files so downstream tools like MS-DIAL or MS-FINDER can apply RT-window
  filtering or Gaussian RT similarity scoring to candidate structures.
when_to_use_negative:
- Predictions fall outside the chemical landscape of the Retip training set — the
  paper states results may be more erroneous in that case
- The compound class is a complex, high molecular weight lipid such as triacylglycerols
  or phosphatidylcholines, which were largely excluded from Retip's training data
- The chromatographic method differs slightly from the training method — the study
  did not test whether RT predictions can be adjusted for small chromatographic changes
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: Retip
  role: Provides prep.mona(), addRT.mona(), and RT.export() to write predicted RTs
    into NIST-compatible .msp files formatted for MS-DIAL and MS-FINDER
  repo: https://github.com/olobion/Retip
- name: MS-DIAL
  role: 'Downstream consumer: uses the exported RT annotations as RT windows in compound
    identification confidence scoring (version 3.90+)'
- name: MS-FINDER
  role: 'Downstream consumer: ranks or filters structure candidates by retention time
    similarity using Gaussian functions and RT tolerances (versions 3.24+/3.30)'
- name: R
  role: Runtime environment for the Retip R package functions (addRT.mona, RT.export,
    prep.mona)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:281d7be189c3a005ba92db1401684adf61ddba6c79c707c6d7577b7c0df6e9e6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - MS-DIAL
  - MS-FINDER
  - R
  - Retip
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# NIST-compatible MSP file generation

## Summary

Writes Retip-predicted retention times back into NIST-compatible .msp mass spectral search files via addRT.mona() and RT.export(), so external mass spectrometry software (MS-DIAL 3.90+, MS-FINDER 3.24+/3.30, Agilent MassHunter, Waters, ThermoFisher) can use RT windows and Gaussian RT similarity scoring for compound annotation.

## When to use

Use when predicted retention times must be exported into NIST-format .msp spectral files so downstream tools like MS-DIAL or MS-FINDER can apply RT-window filtering or Gaussian RT similarity scoring to candidate structures.

## When NOT to use

- Predictions fall outside the chemical landscape of the Retip training set — the paper states results may be more erroneous in that case
- The compound class is a complex, high molecular weight lipid such as triacylglycerols or phosphatidylcholines, which were largely excluded from Retip's training data
- The chromatographic method differs slightly from the training method — the study did not test whether RT predictions can be adjusted for small chromatographic changes

## Inputs

- Retip prediction results (predicted RTs for MoNA or library entries from RT.spell/getRT.smile)
- Source .msp mass spectral search format file (via prep.mona())
- Target downstream program identifier (MS-DIAL, MS-FINDER, Agilent MassHunter, Waters, ThermoFisher)
- Polarity mode (pol argument)

## Outputs

- NIST-compatible .msp file with predicted retention time annotations written per compound entry

## How to apply

Load the Retip-predicted retention times (e.g., from RT.spell or getRT.smile for MoNA or library entries) together with the source .msp spectral file using prep.mona(). Implement addRT.mona() to write each matched entry's predicted retention time back into the corresponding .msp entry. Implement RT.export() with its data, program, and pol (polarity) arguments to format the output for the target program (MS-DIAL, MS-FINDER, Agilent MassHunter, Waters, or ThermoFisher). Emit NIST-compatible .msp files so MS-DIAL (3.90+) can use RT windows in compound identification confidence scoring and MS-FINDER (3.24+/3.30) can rank or filter structure candidates by Gaussian-function RT similarity with RT tolerances. Ground the RT window width in the model error: the paper's HILIC/RPLC Keras models had mean absolute errors of about 1 min, used as thresholds for excluding false positives.

## Related tools

- **Retip** (Provides prep.mona(), addRT.mona(), and RT.export() to write predicted RTs into NIST-compatible .msp files formatted for MS-DIAL and MS-FINDER) — https://github.com/olobion/Retip
- **MS-DIAL** (Downstream consumer: uses the exported RT annotations as RT windows in compound identification confidence scoring (version 3.90+))
- **MS-FINDER** (Downstream consumer: ranks or filters structure candidates by retention time similarity using Gaussian functions and RT tolerances (versions 3.24+/3.30))
- **R** (Runtime environment for the Retip R package functions (addRT.mona, RT.export, prep.mona))

## Examples

```
RT.export(data = rt_predictions, program = "MS-FINDER", pol = "positive")
```

## Evaluation signals

- The exported .msp is parseable by NIST-compatible MS/MS search software
- Every compound entry in the exported .msp carries its predicted RT value matching the Retip prediction table
- RT windows used downstream are consistent with the ~1 min prediction MAE thresholds reported for false-positive exclusion
- In MS-FINDER isomer search, the RT filter reduces candidate structures (the paper reports a 68% reduction in candidate structures on mouse blood plasma)
- Export polarity and program formatting match the target software's expectations for the pol/program arguments used

## Limitations

- 1 min prediction errors preclude discriminating closely eluting compounds in typical 12 min UPLC methods
- ±1 min RT windows excluded about 25% of true positives in the paper's models
- No current models yield validation MAEs better than ±1 min in retention prediction, so narrow RT search windows are not adequate; predictions are instead useful for disclosing likely in-source fragmentation MS/MS spectra and ranking positional isomers
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity; many chemicals in test and validation were not well represented by the training models

## Evidence
