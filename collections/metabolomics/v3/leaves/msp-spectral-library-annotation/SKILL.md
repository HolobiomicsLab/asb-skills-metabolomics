---
name: msp-spectral-library-annotation
description: Use when a .msp spectral library (e.g. MoNA-style entries) must carry
  predicted retention times so downstream tools like MS-DIAL or MS-FINDER can filter
  or rank structure candidates by RT similarity.
when_to_use_negative:
- The task is predicting retention times from structures (use Retip's RT.spell/getRT.smile
  prediction workflow, not the .msp annotation/export step).
- The downstream tool is older than the supported versions (MS-DIAL < 3.90 or MS-FINDER
  < 3.24), which cannot consume RT windows or Gaussian RT similarity from the exported
  library.
- Predictions fall outside the chemical landscape of the training set (e.g. complex
  high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines were
  largely excluded from training), making the embedded RTs unreliable.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Retip
  role: R package providing prep.mona(), addRT.mona(), and RT.export() to embed predicted
    RTs into .msp library files and format output for target programs
  repo: https://github.com/olobion/Retip
- name: MS-DIAL
  role: Downstream consumer (version 3.90+) that uses RT windows from the annotated
    .msp in compound identification confidence scoring
- name: MS-FINDER
  role: Downstream consumer (version 3.30) that ranks or filters structure candidates
    by retention time similarity using Gaussian functions and RT tolerances
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:0368bb3a21f2d0177ccf248b087842b21553e1e4e446a7ea712a2d5f5b42a216
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# msp-spectral-library-annotation

## Summary

Write Retip-predicted retention times back into NIST-compatible .msp mass search format spectral library entries (via prep.mona/addRT.mona) and export them with RT.export() so MS-DIAL (3.90+) and MS-FINDER (3.24+/3.30) can apply RT windows and Gaussian RT similarity scoring during compound annotation.

## When to use

Use when a .msp spectral library (e.g. MoNA-style entries) must carry predicted retention times so downstream tools like MS-DIAL or MS-FINDER can filter or rank structure candidates by RT similarity.

## When NOT to use

- The task is predicting retention times from structures (use Retip's RT.spell/getRT.smile prediction workflow, not the .msp annotation/export step).
- The downstream tool is older than the supported versions (MS-DIAL < 3.90 or MS-FINDER < 3.24), which cannot consume RT windows or Gaussian RT similarity from the exported library.
- Predictions fall outside the chemical landscape of the training set (e.g. complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines were largely excluded from training), making the embedded RTs unreliable.

## Inputs

- Retip-predicted retention times for MoNA or library entries (from RT.spell/getRT.smile)
- Source .msp mass search format spectral library file
- Target program name (MS-DIAL, MS-FINDER, Agilent MassHunter, Waters, ThermoFisher) and ion polarity for RT.export()

## Outputs

- NIST-compatible .msp spectral library file with predicted RT embedded per compound entry
- RT annotations formatted for MS-DIAL RT windows and MS-FINDER Gaussian RT similarity scoring

## How to apply

Load the Retip-predicted RTs (produced by RT.spell/getRT.smile) together with the source .msp spectral file via prep.mona(). Use addRT.mona() to write each predicted retention time back into the matching entry of the .msp file, keyed by the compound/library identifier. Then call RT.export() with its data, program, and pol (polarity) arguments to format the output for the target program — MS-DIAL, MS-FINDER, Agilent MassHunter, Waters, or ThermoFisher packages. Emit NIST-compatible .msp files: MS-DIAL 3.90+ converts the embedded RTs into RT windows for identification confidence scoring, and MS-FINDER 3.30 ranks/filters isomer candidates using Gaussian RT similarity with RT tolerances. Validate the export by parsing the .msp in NIST-compatible MS/MS search software and confirming each compound carries its predicted RT, consistent with the ±1 min windows used to exclude false positives (a threshold that excluded ~25% of true positives in the study).

## Related tools

- **Retip** (R package providing prep.mona(), addRT.mona(), and RT.export() to embed predicted RTs into .msp library files and format output for target programs) — https://github.com/olobion/Retip
- **MS-DIAL** (Downstream consumer (version 3.90+) that uses RT windows from the annotated .msp in compound identification confidence scoring)
- **MS-FINDER** (Downstream consumer (version 3.30) that ranks or filters structure candidates by retention time similarity using Gaussian functions and RT tolerances)

## Examples

```
devtools::install_github('olobion/Retip'); rt <- prep.mona(speclib, predictedRTs); addRT.mona(rt); RT.export(rt, program='MS-FINDER', pol='positive')
```

## Evaluation signals

- The exported .msp parses cleanly in NIST-compatible MS/MS search software without format errors.
- Every compound entry in the output .msp carries a predicted RT consistent with its source library match.
- The RT.export() program and pol (polarity) arguments match the intended downstream tool and ionization mode.
- Embedded RTs support the expected filtering behavior: ~±1 min windows that reduced MS-FINDER isomer candidates by 68% on mouse blood plasma.
- RT annotations round-trip: re-reading the exported file yields the same predicted RTs written by addRT.mona().

## Limitations

- 1 min prediction error windows preclude discriminating closely eluting compounds in typical 12 min UPLC methods.
- The ±1 min windows excluded about 25% of true positives, so RT filtering trades sensitivity for specificity.
- Predictions used outside the training set's chemical landscape may be more erroneous; complex high-molecular-weight lipids were largely excluded from training.

## Evidence
