---
name: mz-to-mass-adduct-conversion
description: Use when a mass spectral search tool needs to compare experimental precursor
  m/z or theoretical fragment m/z to molecule masses and the ion mode and maximum
  absolute charge must be chosen to define the [M+H]+/[M-H]- conversion.
when_to_use_negative:
- Spectra have already been converted to neutral masses upstream — reapplying an adduct
  would double-correct and shift all masses
- Input spectra contain charge states above the configured maximum absolute charge,
  so the assumed [M+H]+/[M-H]- or doubly charged adduct does not describe the observed
  ion
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: VInSMoC
  role: Applies the adduct conversion internally when scoring query mass spectra against
    candidate molecules, using the user-configured ion mode and maximum absolute charge
    to translate m/z into masses for precursor and fragment matching
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:7ccc813c520d7ff443f34fe8aadcdc2cc90005d7f6af55969fea0413e3baf67e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - VInSMoC
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# mz-to-mass-adduct-conversion

## Summary

Converts experimental precursor and product m/z values into neutral molecular masses by applying adduct and charge assumptions before any mass-based matching, as implemented in VInSMoC's exact and variable search modes.

## When to use

Use when a mass spectral search tool needs to compare experimental precursor m/z or theoretical fragment m/z to molecule masses and the ion mode and maximum absolute charge must be chosen to define the [M+H]+/[M-H]- conversion.

## When NOT to use

- Spectra have already been converted to neutral masses upstream — reapplying an adduct would double-correct and shift all masses
- Input spectra contain charge states above the configured maximum absolute charge, so the assumed [M+H]+/[M-H]- or doubly charged adduct does not describe the observed ion

## Inputs

- Query mass spectra with precursor m/z and peak lists (MGF, mzXML, or mzML)
- Ion mode setting (positive or negative) matching the input spectrum
- Maximum absolute charge parameter (1 or 2)

## Outputs

- Neutral molecular masses derived from precursor m/z for candidate-molecule matching
- Neutral fragment masses derived from theoretical fragment m/z for peak matching within the product ion tolerance

## How to apply

Read the ion mode from the input spectrum (positive or negative) and set the maximum absolute charge parameter: 1 restricts conversion to [M+H]+ (positive) or [M-H]- (negative) adducts, while 2 additionally allows [M+2H]++ or [M-2H]-- adducts, based on ion mode. Apply the chosen adduct to convert query precursor m/z to neutral mass so candidate molecules whose masses (or variants explaining the difference Δ) can be matched within the precursor ion tolerance. During fragmentation-graph expansion, use the same adduct rule to convert theoretical fragment m/z back to neutral masses before matching to experimental peaks within the product ion tolerance. The rationale is that m/z and mass are only comparable once the charge state and protonation/deprotonation assumption are fixed; misconfigured adducts shift all theoretical masses and invalidate precursor and fragment matching.

## Related tools

- **VInSMoC** (Applies the adduct conversion internally when scoring query mass spectra against candidate molecules, using the user-configured ion mode and maximum absolute charge to translate m/z into masses for precursor and fragment matching) — https://run.npanalysis.org

## Examples

```
Upload extract-3_Metal-mix_1mM.mzML on run.npanalysis.org, select VInSMoC, set ion mode=positive, maximum absolute charge=1 (i.e. [M+H]+), then submit the task
```

## Evaluation signals

- Positive-mode spectra with maximum absolute charge 1 must match candidate molecules via [M+H]+ only; any hit whose mass offset corresponds to [M+2H]++ indicates misconfiguration
- Converted precursor masses should fall within the precursor ion tolerance of matched candidate molecule masses
- Theoretical fragment masses derived from the same adduct rule should match experimental peaks within the product ion tolerance
- Varying the maximum absolute charge from 1 to 2 should change which candidate molecules are matched, and the change should correspond to doubly charged adduct explanations of Δ

## Limitations

- Adduct choice is an assumption: maximum absolute charge of 1 restricts to [M+H]+ or [M-H]- and may miss ions carrying other adducts or charge states
- If unsure, the README advises leaving precursor and product ion tolerances at defaults, but incorrect ion mode settings relative to the input spectrum will invalidate all mass conversions

## Evidence

- [methods] computing higher-depth fragments only when their parent fragment is explained by a query peak; m/z converted to neutral masses using [M+H]+ adducts (maximum absolute charge 1, positive ion mode): [evidence span withheld — anchor status: not_found]
