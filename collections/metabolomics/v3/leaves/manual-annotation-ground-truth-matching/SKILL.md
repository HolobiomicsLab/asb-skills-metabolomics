---
name: manual-annotation-ground-truth-matching
description: Use when benchmarking feature alignment or detection methods on public
  LC–MS metabolomics datasets (e.g., MTBLS562, QE_HF, TripleTOF_6600) where a manual
  annotation of library analytes is available to serve as ground truth.
when_to_use_negative:
- No manually annotated library analytes exist for the dataset — the evaluation would
  have no ground truth to match against
- The goal is de novo feature alignment or quantification rather than scoring an alignment
  method's accuracy against a reference
- Input is a single-run feature table with no multi-run alignment to evaluate
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Aligner whose multipartite-graph/multidimensional-assignment output features
    are matched against the manual annotation to compute feature and analyte accuracy
  repo: https://github.com/CSi-Studio/G-Aligner
- name: MZmine2
  role: Comparison feature extraction/alignment software whose results (e.g., MTBLS562_results_mzmine2)
    are scored against the same manual annotation baseline
- name: OpenMS
  role: Comparison feature extraction/alignment software scored against the manual
    annotation in the benchmark workflow
- name: XCMS
  role: Comparison feature extraction/alignment software (R workflow, evaluate_metapro_xcms.Rmd)
    scored against the manual annotation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:543e55b9be8a2177820b35343c1fa31eea339a6bd14b5d3e13ef59788f344fa2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - OpenMS
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# manual-annotation-ground-truth-matching

## Summary

Evaluates LC–MS feature alignment accuracy by matching algorithm-aligned consensus features against a manually annotated ground-truth dataset (m/z, RT, area of library analytes), counting accurately aligned features and analytes. Provides the ground-truth-based validation used to benchmark G-Aligner against MZmine2, OpenMS, and XCMS.

## When to use

Use when benchmarking feature alignment or detection methods on public LC–MS metabolomics datasets (e.g., MTBLS562, QE_HF, TripleTOF_6600) where a manual annotation of library analytes is available to serve as ground truth.

## When NOT to use

- No manually annotated library analytes exist for the dataset — the evaluation would have no ground truth to match against
- The goal is de novo feature alignment or quantification rather than scoring an alignment method's accuracy against a reference
- Input is a single-run feature table with no multi-run alignment to evaluate

## Inputs

- Manually annotated feature dataset (m/z, RT, area of library analytes) from Zenodo 10.5281/zenodo.8313034
- Extracted feature lists in csv format containing m/z, RT and area columns (e.g., from data/MTBLS562)
- Feature alignment results saved in the experiment folder (G-Aligner, MZmine2, OpenMS, or XCMS outputs)

## Outputs

- alignment_accuracy_mtbls562.csv with feature-level accuracy, analyte-level accuracy, F1 score, and computational time
- Counts of accurately aligned features and accurately aligned analytes per alignment method

## How to apply

Download the manually annotated feature dataset containing m/z, RT, and area information of library analytes from Zenodo record 10.5281/zenodo.8313034, plus the extracted feature lists (e.g., from data/MTBLS562 in the G-Aligner repository). Configure the evaluation script (e.g., data/evaluate_metapro_galigner.py with the MTBLS562 parameter set from Table S3) to point at the annotated feature lists. Run the aligner (e.g., G-Aligner builds a multipartite graph over features of all runs and solves the unbalanced multidimensional assignment problem), then match the aligned features in the experiment folder against the manual annotation to count accurately aligned features and accurately aligned analytes. Compute feature-level and analyte-level accuracy percentages, F1 score, and computational time, saving the metrics to a CSV such as alignment_accuracy_mtbls562.csv for comparison against the paper's Table 3.

## Related tools

- **G-Aligner** (Aligner whose multipartite-graph/multidimensional-assignment output features are matched against the manual annotation to compute feature and analyte accuracy) — https://github.com/CSi-Studio/G-Aligner
- **MZmine2** (Comparison feature extraction/alignment software whose results (e.g., MTBLS562_results_mzmine2) are scored against the same manual annotation baseline)
- **OpenMS** (Comparison feature extraction/alignment software scored against the manual annotation in the benchmark workflow)
- **XCMS** (Comparison feature extraction/alignment software (R workflow, evaluate_metapro_xcms.Rmd) scored against the manual annotation)

## Examples

```
python data/evaluate_metapro_galigner.py  # after pointing the script at the MTBLS562 annotated feature lists from Zenodo 10.5281/zenodo.8313034
```

## Evaluation signals

- Computed feature accuracy, analyte accuracy, and F1 for the G-Aligner row on MTBLS562 should reproduce Table 3 values: F1 = 1.000, feature accuracy = 0.999, analyte accuracy = 0.981
- G-Aligner should outperform OpenMS, MZmine2, and XCMS on the same annotation, with up to 9.8% and 26.6% increases in accurately aligned features and analytes
- Counts of accurately aligned features and analytes must be derived by matching aligned consensus features against the manual annotation's m/z, RT, and area values
- Output CSV (alignment_accuracy_mtbls562.csv) contains one row per method with feature accuracy, analyte accuracy, and runtime columns
- Optimized parameters used for each dataset must match the supplementary parameter sets (Table S1–S3) for reproducibility

## Limitations

- The main limitation of G-Aligner is the lack of computing speed; the multidimensional assignment problem is NP-Hard and requires a lot of computing time
- Optimized parameters for each dataset are reported only in supplementary tables (S1–S3), not in the main text
- GPU acceleration is not yet implemented, and a gap remains between the provided acceleration methods and painless application

## Evidence

- [methods] Manual annotation of feature lists: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [abstract] Accuracy findings on manual annotations: "G-Aligner achieved the highest alignment accuracy on manual annotated and popular software extracted features, proving the effectiveness and robustness of the algorithm"
- [readme] Output location of alignment results: "Feature alignment results are saved in ```experiment``` folder."
- [discussion] Dataset-specific optimized parameters: "Table S3, Optimized parameters used in the MTBLS562 dataset evaluation"
- [readme] Supported input format: "Feature extraction rsults in csv format, containing m/z, RT and area columns."
