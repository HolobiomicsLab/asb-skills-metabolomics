---
name: ransac-obi-warp-retention-time-coarse-registration
description: Use when aligning features across multiple LC-MS runs whose retention
  times drift nonlinearly (e.g. due to column temperature, aging, contamination) and
  a coarse RT correction is needed before feature matching.
when_to_use_negative:
- You need final consensus feature groups — RT coarse registration alone only corrects
  drifting trends and cannot accurately correct retention time of each feature; use
  the full G-Aligner matching pipeline.
- Input is raw mzML/MS data rather than extracted feature lists — this stage operates
  on feature tables with m/z, RT and area columns produced by feature extraction software.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: G-Aligner
  role: Implements the RANSAC + OBI-Warp coarse retention time registration as the
    first stage of its alignment pipeline
  repo: https://github.com/CSi-Studio/G-Aligner
- name: RANSAC
  role: Robustly estimates the RT drifting trend between runs before OBI-Warp coarse
    registration
- name: OBI-Warp
  role: Performs the stable coarse retention time registration between runs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:97902251745b6152f84be148be52a5148505601125ebda18c5e1e176f6669fae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OBI-Warp
  - RANSAC
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ransac-obi-warp-retention-time-coarse-registration

## Summary

Performs stable coarse retention time registration between LC-MS runs using RANSAC and OBI-Warp, as the first stage of G-Aligner's feature alignment pipeline for untargeted metabolomics, before graph-based feature matching.

## When to use

Use when aligning features across multiple LC-MS runs whose retention times drift nonlinearly (e.g. due to column temperature, aging, contamination) and a coarse RT correction is needed before feature matching.

## When NOT to use

- You need final consensus feature groups — RT coarse registration alone only corrects drifting trends and cannot accurately correct retention time of each feature; use the full G-Aligner matching pipeline.
- Input is raw mzML/MS data rather than extracted feature lists — this stage operates on feature tables with m/z, RT and area columns produced by feature extraction software.

## Inputs

- CSV feature tables per LC-MS run (e.g. from MZmine2, OpenMS, XCMS or manual annotation), each containing m/z, RT and area columns
- Feature lists of the benchmark datasets MTBLS562, QE_HF and TripleTOF_6600 (mzML-derived, downloadable from Zenodo 10.5281/zenodo.8313034)

## Outputs

- Coarsely RT-registered feature lists across runs, ready for G-Aligner's graph-building stage (neighborhood search under user-specified m/z and RT tolerances)

## How to apply

After feature extraction (which yields CSV feature tables with m/z, RT and area columns for each run), run the retention time alignment stage of G-Aligner between runs. G-Aligner uses the existing RANSAC algorithm to robustly estimate the RT drifting trend and OBI-Warp to perform stable coarse registration between runs, correcting the nonlinear RT drifts that arise from unavoidable slight changes in chromatogram conditions. Because RT alignment can only describe drifting trends and cannot accurately correct the RT of each feature, this coarse registration is deliberately followed by the G-Aligner feature matching stage (multipartite graph over features with user-specified m/z and RT tolerance neighborhoods, solved as an unbalanced multidimensional assignment problem). Do not rely on the coarse registration alone for the final consensus feature assignment; it only narrows the RT search space for potential correspondences. Parameter values used per benchmark dataset (MTBLS562, QE_HF, TripleTOF_6600) are reported in the supplementary tables (e.g. Table S3 for MTBLS562), so consult those when reproducing evaluations.

## Related tools

- **G-Aligner** (Implements the RANSAC + OBI-Warp coarse retention time registration as the first stage of its alignment pipeline) — https://github.com/CSi-Studio/G-Aligner
- **RANSAC** (Robustly estimates the RT drifting trend between runs before OBI-Warp coarse registration)
- **OBI-Warp** (Performs the stable coarse retention time registration between runs)

## Examples

```
cd ROOT_PATH && pip install -r requirements.txt && python data/evaluate_metapro_galigner.py
```

## Evaluation signals

- After coarse registration, RT drifts between runs should follow a stable, corrected trend consistent with the library analyte RT drift distributions reported for the TripleTOF 6600 and QE HF datasets.
- The downstream neighborhood search under the user-specified m/z and RT tolerances should build a relation graph whose edges are plausible potential correspondences (no obviously misaligned edges spanning large residual RT differences).
- When combined with the full G-Aligner pipeline, the resulting consensus feature groups should not contain ambiguous matched groups, consistent with G-Aligner achieving the highest alignment accuracy.
- Reproduced runs should match the optimized per-dataset parameters reported in supplementary Tables S1–S3 (TripleTOF 6600, QE HF, MTBLS562).

## Limitations

- Retention time alignment can only describe drifting trends but cannot accurately correct the retention time of each feature, so coarse registration must be followed by feature matching.
- Most existing methods rely heavily on RT alignment and naive nearest neighbor search — this stage alone reproduces that weakness; it is not a substitute for graph-based matching.
- Optimized parameters are reported only in supplementary tables (S1–S3), not in the main text, so exact reproduction requires the supplementary material.

## Evidence

- [intro] Coarse registration precedes graph-based matching: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [intro] Nonlinear RT drift is the motivation: "features have nonlinear retention time (RT) drifts between runs due to unavoidable slight changes in chromatogram conditions [4], such as column temperature, column aging and contamination"
- [intro] Limitation of RT alignment alone: "Most existing methods rely heavily on retention time alignment and only use naive nearest neighbor search in feature matching."
- [readme] Downstream graph building after coarse registration: "G-Aligner treats features and potential correspondences as nodes and edges of a multipartite graph, converts the feature matching problem as a multidimensional assignment problem (MAP)"
- [discussion] Per-dataset optimized parameters in supplements: "Table S3, Optimized parameters used in the MTBLS562 dataset evaluation"
