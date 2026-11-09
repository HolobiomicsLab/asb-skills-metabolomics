---
name: chromatogram-cosine-similarity-optimization
description: Use when MRM/LC-MS samples show retention time shifts that break peak
  correspondence across EICs and you need per-sample drift estimates to align peaks
  or group EICs across runs.
when_to_use_negative:
- Input is already an aligned feature/peak table where RT correspondence across samples
  is established
- Data is DDA/DIA rather than targeted MRM — MRMPro's drift estimation is designed
  for MRM EICs; use a proteomics-wide alignment method instead
- The goal is improving peak extraction accuracy itself — the paper states peak extraction
  accuracy is not yet improved and defers that to future work
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: Web-based MRM platform in which the chromatogram-similarity RT drift estimation
    algorithm runs; drifts feed peak alignment and automatic EIC grouping
  repo: https://github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: Converts vendor raw files (with MRM acquisition support) to Aird format consumed
    downstream by MRMPro workflows
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:ac97c07937b2827a8746f0ed90f7a8d51c9aeeb792795ecbe1cdc60d8fb232ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MRMPro-Doc
  tools:
  - AirdPro
  - MRMPro
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# chromatogram-cosine-similarity-optimization

## Summary

Estimates retention time (RT) drifts between MRM samples by maximizing cosine similarity between extracted-ion chromatograms (EICs), using a combined coarse grid search and gradient descent refinement against a reference EIC. This powers peak alignment and automatic EIC grouping in MRMPro.

## When to use

Use when MRM/LC-MS samples show retention time shifts that break peak correspondence across EICs and you need per-sample drift estimates to align peaks or group EICs across runs.

## When NOT to use

- Input is already an aligned feature/peak table where RT correspondence across samples is established
- Data is DDA/DIA rather than targeted MRM — MRMPro's drift estimation is designed for MRM EICs; use a proteomics-wide alignment method instead
- The goal is improving peak extraction accuracy itself — the paper states peak extraction accuracy is not yet improved and defers that to future work

## Inputs

- Extracted-ion chromatogram (EIC) peak tables for all MRM samples (loaded into a pandas DataFrame)
- Transition group identifiers grouping EICs across samples

## Outputs

- rt_drift_estimates.csv with one row per sample per transition group containing columns: sample, transition group, estimated RT drift

## How to apply

Load the EIC peak tables for all MRM samples into a pandas DataFrame, then select a reference EIC per transition group by computing, per sample, the product of the peak count and the maximum peak height and taking the arg-max sample as reference. For each non-reference sample, run a coarse grid search over candidate RT shifts to maximize cosine similarity between the shifted chromatogram and the reference chromatogram. Refine each grid-search optimum with gradient descent on the RT shift parameter, iterating while cosine similarity continues to increase. Collect the converged drift value for every sample into a table with columns sample, transition group, and estimated RT drift; these drifts are then used for peak alignment and automatic EIC grouping as in MRMPro.

## Related tools

- **MRMPro** (Web-based MRM platform in which the chromatogram-similarity RT drift estimation algorithm runs; drifts feed peak alignment and automatic EIC grouping) — https://github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (Converts vendor raw files (with MRM acquisition support) to Aird format consumed downstream by MRMPro workflows) — https://github.com/CSi-Studio/AirdPro/releases/

## Examples

```
df = pd.read_csv('eic_peaks.csv'); ref = (df.groupby('sample').peak_count * df.groupby('sample').max_height).idxmax(); drift = grid_search_then_gd(df, ref)
```

## Evaluation signals

- Output table contains exactly one drift estimate per input sample per transition group
- Each reported drift attains a cosine similarity no lower than its grid-search initialization (drift quality acceptance check)
- Converged drift values, when applied, produce successful peak alignment and automatic EIC grouping across samples
- Gradient descent refinement never decreases cosine similarity relative to the coarse grid-search optimum

## Limitations

- The paper reports no numeric threshold for drift quality; acceptance is only the monotonic-similarity check
- MRMPro relies on classical algorithms rather than more advanced analytical algorithms; incorporation of advanced algorithms is only planned future work
- Reference EIC choice depends on peak count × maximum peak height heuristic, which may be unstable for low-signal samples

## Evidence

- [abstract] we proposed a retention time drift estimation algorithm based on similarity of chromatograms: "we proposed a retention time drift estimation algorithm based on similarity of chromatograms"
- [abstract] The estimated retention time drifts were then used for peak alignment and automatic EIC grouping.: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [discussion] MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework: "MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework"
- [readme] Supporting MRM acquisition method: [evidence span withheld — anchor status: external_doc]
