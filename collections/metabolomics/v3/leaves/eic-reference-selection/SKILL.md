---
name: eic-reference-selection
description: Use when processing MRM/SRM extracted-ion chromatogram peak tables from
  multiple samples and a per-transition-group reference trace must be chosen before
  estimating retention time drifts or grouping EICs.
when_to_use_negative:
- Input is a single-sample dataset with no cross-sample drift to correct — there is
  no need for a reference EIC
- Input is already drift-corrected and peak-aligned; selecting a reference would add
  no information
- Analysis goal is raw vendor-to-Aird file conversion (use AirdPro instead) rather
  than post-extraction chromatogram alignment
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MRMPro
  role: Web-based MRM analysis platform implementing the reference selection and subsequent
    RT drift estimation based on chromatogram similarity
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: Aird-SDK
  role: Java/C#/Python SDK for reading chromatogram data (ChromatogramIndex for MRM
    acquisition mode) from Aird files prior to EIC analysis
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: GUI client for converting vendor MRM files to Aird format; supports MRM/SRM
    acquisition so EIC peak tables can be generated
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:80cadd7eca8d431557abda61b5a6e05772d9ba4da7870adb8f2d8b0d1a7ef1bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MRMPro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# EIC reference selection

## Summary

Selects the highest-quality reference extracted-ion chromatogram (EIC) per transition group for MRM data, scoring each sample by the product of its peak count and maximum peak height, so that downstream retention time drift estimation aligns weaker chromatograms against the strongest one.

## When to use

Use when processing MRM/SRM extracted-ion chromatogram peak tables from multiple samples and a per-transition-group reference trace must be chosen before estimating retention time drifts or grouping EICs.

## When NOT to use

- Input is a single-sample dataset with no cross-sample drift to correct — there is no need for a reference EIC
- Input is already drift-corrected and peak-aligned; selecting a reference would add no information
- Analysis goal is raw vendor-to-Aird file conversion (use AirdPro instead) rather than post-extraction chromatogram alignment

## Inputs

- Extracted-ion chromatogram (EIC) peak tables for all MRM samples (e.g. pandas DataFrame with columns: sample, transition group, peak count, maximum peak height, RT trace)

## Outputs

- Reference EIC assignment table mapping each transition group to its reference sample
- Per-sample EICs marked for alignment against the reference (input to RT drift estimation)

## How to apply

Load the EIC peak tables for all MRM samples into a pandas DataFrame keyed by sample and transition group. For each transition group, compute per sample the product of the number of detected peaks and the maximum peak height, then take the arg-max sample as the reference EIC for that group. The rationale is that a reference with many peaks and high intensity gives the most informative cosine-similarity target when the algorithm aligns each remaining EIC individually against it using grid search plus gradient descent. The resulting reference assignment feeds MRMPro's retention time drift estimation, whose drift values are subsequently used for peak alignment and automatic EIC grouping.

## Related tools

- **MRMPro** (Web-based MRM analysis platform implementing the reference selection and subsequent RT drift estimation based on chromatogram similarity) — github.com/CSi-Studio/MRMPro-Doc
- **Aird-SDK** (Java/C#/Python SDK for reading chromatogram data (ChromatogramIndex for MRM acquisition mode) from Aird files prior to EIC analysis) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (GUI client for converting vendor MRM files to Aird format; supports MRM/SRM acquisition so EIC peak tables can be generated) — https://github.com/CSi-Studio/AirdPro/releases/

## Examples

```
import pandas as pd; eics = pd.read_csv('eic_peaks.csv'); eics['ref_score'] = eics.peak_count * eics.max_height; refs = eics.loc[eics.groupby('transition_group')['ref_score'].idxmax()]
```

## Evaluation signals

- Exactly one reference sample is assigned per transition group (arg-max is unique per group)
- The selected reference maximizes peak count × maximum peak height within its transition group across all samples
- Each downstream RT drift estimate, initialized from the grid search at the reference-aligned optimum, attains a cosine similarity no lower than its grid-search initialization
- The rt_drift_estimates.csv output contains exactly one drift estimate per input sample per transition group

## Limitations

- MRMPro relies on classical algorithms rather than more advanced analytical algorithms; incorporation of advanced algorithms is only planned future work, so peak extraction accuracy is not yet improved
- The paper reports no numeric threshold for drift quality, so drift acceptance must be validated by relative cosine-similarity checks rather than a fixed cutoff
- Manual inspection of MRM data is still necessary in most scenarios, and its cost grows as the sample queue grows — reference-based drift estimation supports but does not replace manual calibration

## Evidence

- [methods] Reference EIC selection scoring rule: [evidence span withheld — anchor status: not_found]
- [abstract] RT drift estimation via chromatogram similarity: "we proposed a retention time drift estimation algorithm based on similarity of chromatograms"
- [abstract] Downstream use of estimated drifts: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [readme] MRM chromatogram data access via Aird SDK: [evidence span withheld — anchor status: not_found]
