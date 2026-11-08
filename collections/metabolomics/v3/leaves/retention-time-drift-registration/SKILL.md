---
name: retention-time-drift-registration
description: Use when aligning features across multiple LC–MS runs whose RTs drift
  nonlinearly due to chromatogram condition changes (column temperature, aging, contamination),
  as the first stage before feature matching.
when_to_use_negative:
- You need per-feature RT correction, not just drift-trend registration — the article
  notes RT alignment 'can only describe drifting trends but cannot accurately correct
  retention time of each feature'; pair this with graph-based feature matching
- Input is already an aligned consensus feature table with no cross-run RT drift to
  register
- Computation time is critical — the paper identifies 'lack of computing speed' as
  the main limitation of the G-Aligner pipeline that includes this stage
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: G-Aligner
  role: Implements the RT coarse registration stage (RANSAC + OBI-Warp) followed by
    multipartite-graph feature matching; configured via data/evaluate_mzmine2_galigner.py
  repo: https://github.com/CSi-Studio/G-Aligner
- name: RANSAC
  role: Robust fitting of the RT drift model between runs during the coarse registration
    stage
- name: OBI-Warp
  role: Chromatogram warping used with RANSAC for stable coarse RT registration between
    runs
- name: MZmine2
  role: Source of self-extracted untargeted feature lists (m/z, RT, area csv) fed
    into the RT registration and alignment workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:c1d297e29ffed8c8325ff35495ae8678e94cd224bd235a1c2d80002356e4ca64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OBI-Warp
  - RANSAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# retention-time-drift-registration

## Summary

Coarse registration stage of G-Aligner that corrects nonlinear retention time (RT) drift between LC–MS runs using RANSAC and OBI-Warp before multipartite-graph feature matching, for untargeted metabolomics feature alignment on mzML-derived csv feature tables (m/z, RT, area).

## When to use

Use when aligning features across multiple LC–MS runs whose RTs drift nonlinearly due to chromatogram condition changes (column temperature, aging, contamination), as the first stage before feature matching.

## When NOT to use

- You need per-feature RT correction, not just drift-trend registration — the article notes RT alignment 'can only describe drifting trends but cannot accurately correct retention time of each feature'; pair this with graph-based feature matching
- Input is already an aligned consensus feature table with no cross-run RT drift to register
- Computation time is critical — the paper identifies 'lack of computing speed' as the main limitation of the G-Aligner pipeline that includes this stage

## Inputs

- Per-run feature lists in csv format with m/z, RT and area columns (e.g., MZmine2 self-extracted untargeted features)
- mzML raw files of benchmark datasets (TripleTOF 6600, QE_HF, MTBLS562) if feature extraction must be re-run
- Dataset-specific optimized G-Aligner parameters (supplementary Tables S1–S3)

## Outputs

- RT-registered features per run (coarse-aligned RTs)
- Feature alignment results saved in the experiment folder, consisting of consensus features across runs
- F_ACC and A_ACC evaluation metrics when compared against manually annotated feature lists

## How to apply

First extract features per run (e.g., with MZmine2, XCMS, OpenMS) into csv tables containing m/z, RT and area columns. Then, in the retention time alignment stage, use RANSAC together with OBI-Warp to perform stable coarse registration between runs — RANSAC robustly fits the RT drift trend and OBI-Warp warps the chromatograms to correct the drifting trends. Treat this as coarse registration only: RT alignment describes drifting trends but cannot accurately correct each feature's RT, so downstream feature matching must handle residual drift (G-Aligner does this via a multipartite graph and unbalanced multidimensional assignment solved by three combinatorial optimization methods). In the G-Aligner repo, configure the dataset-specific parameters in data/evaluate_mzmine2_galigner.py (optimized parameters per dataset are reported in supplementary Tables S1–S3 for TripleTOF 6600, QE_HF and MTBLS562) and run it; alignment results are written to the experiment folder.

## Related tools

- **G-Aligner** (Implements the RT coarse registration stage (RANSAC + OBI-Warp) followed by multipartite-graph feature matching; configured via data/evaluate_mzmine2_galigner.py) — https://github.com/CSi-Studio/G-Aligner
- **RANSAC** (Robust fitting of the RT drift model between runs during the coarse registration stage)
- **OBI-Warp** (Chromatogram warping used with RANSAC for stable coarse RT registration between runs)
- **MZmine2** (Source of self-extracted untargeted feature lists (m/z, RT, area csv) fed into the RT registration and alignment workflow)

## Examples

```
cd ROOT_PATH && python data/evaluate_mzmine2_galigner.py  # runs RANSAC/OBI-Warp coarse registration + graph matching; results in experiment/
```

## Evaluation signals

- Feature alignment accuracy (F_ACC) gain over MZmine2's own alignment should reach ~1.7% (TripleTOF 6600), ~9.0% (QE HF), ~0.5% (MTBLS562) on self-extracted features
- Analyte accuracy (A_ACC) gain over MZmine2 should reach ~5.8%, ~26.6% and ~7.7% respectively per Tables 4–6
- Results directory 'experiment' is populated with alignment outputs after running data/evaluate_mzmine2_galigner.py
- RT drift distributions of library analytes per run should show the coarse-registration trend removed before feature matching
- Alignment outputs should correspond to the three benchmark datasets (TripleTOF_6600, QE_HF, MTBLS562) laid out under the data/ tree as in the README

## Limitations

- The overall pipeline's main limitation is computing speed: the multidimensional assignment problem is NP-Hard and requires substantial computing time even with two acceleration methods
- GPU acceleration is not yet implemented
- Optimized parameters are only reported in supplementary Tables S1–S3 and must be tuned per dataset
- RT registration alone is insufficient — most existing methods 'rely heavily on retention time alignment and only use naive nearest neighbor search in feature matching', which G-Aligner's matching stage is designed to fix

## Evidence

- [intro] In the retention time alignment stage, G-Aligner uses the existing RANSAC and OBI-Warp algorithms to perform stable coarse registration between runs.: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [intro] However, features have nonlinear retention time (RT) drifts between runs due to unavoidable slight changes in chromatogram conditions: "However, features have nonlinear retention time (RT) drifts between runs due to unavoidable slight changes in chromatogram conditions"
- [intro] RT alignment can only describe drifting trends but cannot accurately correct retention time of each feature: "Most existing methods rely heavily on retention time alignment and only use naive nearest neighbor search in feature matching."
- [discussion] The RT drift distribution of library analytes on the TripleTOF 6600 dataset and the QE HF dataset: "The RT drift distribution of library analytes on the TripleTOF 6600 dataset and the QE HF dataset"
- [readme] Feature extraction results in csv format, containing m/z, RT and area columns.: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
