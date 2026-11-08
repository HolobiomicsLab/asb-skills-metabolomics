---
name: mzmine-feature-list-processing
description: Use when you have MZmine2 self-extracted untargeted feature lists (CSV
  with m/z, RT, area columns) from multiple LC–MS runs and need more accurate multi-run
  feature alignment than MZmine2's built-in alignment provides.
when_to_use_negative:
- Feature lists are already aligned into consensus features by another tool and only
  need evaluation rather than re-alignment.
- The analysis goal is feature detection or quantification accuracy rather than multi-run
  feature alignment — the manually annotated dataset supports those tasks too, but
  this skill only covers alignment.
- 'Low computing speed is unacceptable: the multidimensional assignment problem is
  NP-Hard and G-Aligner''s main limitation is lack of computing speed.'
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: G-Aligner
  role: 'Graph-based feature alignment engine: consumes the MZmine2 feature lists,
    performs RANSAC/OBI-Warp RT coarse registration and multipartite-graph feature
    matching'
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Source of the untargeted self-extracted feature lists (m/z, RT, area) that
    serve as alignment input, and baseline method for F_ACC/A_ACC comparison
- name: RANSAC
  role: Retention time coarse registration between runs in the G-Aligner alignment
    stage
- name: OBI-Warp
  role: Retention time coarse registration (warping) between runs in the G-Aligner
    alignment stage
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:044e030468faf34efe370efca9f97ea4ffbe556b9391529c9d2467fef2f71abc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OBI-Warp
  - RANSAC
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mzmine-feature-list-processing

## Summary

Consume MZmine2 untargeted feature extraction results (CSV feature lists with m/z, RT, area columns) as input to G-Aligner's graph-based alignment, replacing MZmine2's native RANSAC + nearest-neighbor alignment to improve feature and analyte accuracy on LC–MS metabolomics benchmark datasets.

## When to use

Use when you have MZmine2 self-extracted untargeted feature lists (CSV with m/z, RT, area columns) from multiple LC–MS runs and need more accurate multi-run feature alignment than MZmine2's built-in alignment provides.

## When NOT to use

- Feature lists are already aligned into consensus features by another tool and only need evaluation rather than re-alignment.
- The analysis goal is feature detection or quantification accuracy rather than multi-run feature alignment — the manually annotated dataset supports those tasks too, but this skill only covers alignment.
- Low computing speed is unacceptable: the multidimensional assignment problem is NP-Hard and G-Aligner's main limitation is lack of computing speed.

## Inputs

- MZmine2 untargeted extracted feature lists in CSV format with m/z, RT and area columns
- mzML files of the metabolomics datasets (TripleTOF 6600, QE_HF, MTBLS562 from Zenodo doi 10.5281/zenodo.8313034)
- Manually annotated feature lists (m/z, RT, area of library analytes) for evaluation

## Outputs

- G-Aligner alignment results per dataset, saved in the experiment folder
- F_ACC and A_ACC metrics per dataset computed against manually annotated feature lists

## How to apply

Download the mzML files, manual annotations, and MZmine2 untargeted extracted feature results for the TripleTOF 6600, QE_HF and MTBLS562 datasets from Zenodo (doi 10.5281/zenodo.8313034) and arrange them under the G-Aligner project tree (data/TripleTOF_6600, QE_HF, MTBLS562 folders). Install the G-Aligner third-party dependencies (setup.py in third_party/) and pip requirements to prepare the Python environment. Configure data/evaluate_mzmine2_galigner.py so G-Aligner consumes the MZmine2 self-extracted feature lists (m/z, RT, area columns) for each dataset and performs RT coarse registration with RANSAC/OBI-Warp, then multipartite-graph feature matching solved as an unbalanced multidimensional assignment problem via three combinatorial optimization solvers. Execute the script per dataset, writing alignment results into the experiment folder, then evaluate F_ACC (feature accuracy) and A_ACC (analyte accuracy) against the manually annotated feature lists.

## Related tools

- **G-Aligner** (Graph-based feature alignment engine: consumes the MZmine2 feature lists, performs RANSAC/OBI-Warp RT coarse registration and multipartite-graph feature matching) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Source of the untargeted self-extracted feature lists (m/z, RT, area) that serve as alignment input, and baseline method for F_ACC/A_ACC comparison)
- **RANSAC** (Retention time coarse registration between runs in the G-Aligner alignment stage)
- **OBI-Warp** (Retention time coarse registration (warping) between runs in the G-Aligner alignment stage)

## Examples

```
cd ROOT_PATH && python data/evaluate_mzmine2_galigner.py
```

## Evaluation signals

- Computed F_ACC/A_ACC gains over native MZmine2 alignment match Tables 4-6: F_ACC +1.7% (TripleTOF 6600), +9.0% (QE HF), +0.5% (MTBLS562); A_ACC +5.8%, +26.6%, +7.7%
- Alignment results are written to the experiment folder for each of the three datasets after running data/evaluate_mzmine2_galigner.py
- Input CSV feature lists contain the three required columns: m/z, RT and area
- Evaluation against manually annotated feature lists reproduces higher analyte accuracy on all three benchmark datasets, as reported for MZmine2 self-extracted features

## Limitations

- The main limitation of G-Aligner is the lack of computing speed: the multidimensional assignment problem is NP-Hard and inevitably requires a lot of computing time
- Optimized parameters for each dataset are reported only in supplementary Tables S1-S3, not in the main text
- GPU acceleration is not yet implemented; the two provided acceleration methods narrow but do not close the gap to painless application

## Evidence

- [results] Integrating G-Aligner into the MZmine2 workflow improved alignment on untargeted self-extracted features on all three datasets: [evidence span withheld — anchor status: not_found]
- [intro] G-Aligner performs RT alignment then graph matching: "In the retention time alignment stage, G-Aligner uses the existing RANSAC [10] and OBI-Warp [15] algorithms to perform stable coarse registration between runs."
- [readme] Supported input format is CSV feature lists: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
- [discussion] Optimized parameters in supplementary tables only: "Table S1, Optimized parameters used in the TripleTOF 6600 dataset evaluation"
