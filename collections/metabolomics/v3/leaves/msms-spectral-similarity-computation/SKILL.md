---
name: msms-spectral-similarity-computation
description: Use when you have a processed LC-MS/MS untargeted metabolomics feature
  table with associated MS/MS spectra and need to quantify structural similarity between
  every feature pair before clustering.
when_to_use_negative:
- Input is already a pairwise similarity/distance matrix or pre-clustered feature
  groups — re-computation is redundant.
- You only have MS1 quantification without MS/MS spectra; the skill requires per-feature
  MS/MS spectra as input.
- You want feature-set statistical testing directly on raw intensities without spectral
  structural grouping — use globaltest on feature groups instead of recomputing similarity.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: msFeaST
  role: Pipeline module performing the spectral similarity computation stage; exposes
    the user-selected similarity score and emits the distance matrix for k-medoids
    clustering.
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:56dc2363167bcd3f6942f75ec0fbbe6c5836724633e7ec929a6f423f89f8c7bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# msms-spectral-similarity-computation

## Summary

Compute all pairwise similarity scores between LC-MS/MS feature MS/MS spectra using a user-selected spectral similarity score within the msFeaST workflow, assembling the results into a symmetric distance matrix used for downstream k-medoids feature grouping and t-SNE embedding.

## When to use

Use when you have a processed LC-MS/MS untargeted metabolomics feature table with associated MS/MS spectra and need to quantify structural similarity between every feature pair before clustering.

## When NOT to use

- Input is already a pairwise similarity/distance matrix or pre-clustered feature groups — re-computation is redundant.
- You only have MS1 quantification without MS/MS spectra; the skill requires per-feature MS/MS spectra as input.
- You want feature-set statistical testing directly on raw intensities without spectral structural grouping — use globaltest on feature groups instead of recomputing similarity.

## Inputs

- Processed MS/MS spectra for each LC-MS/MS feature (peak-handled/precursor-normalized fragment peak lists)
- User-selected similarity score choice (e.g. ms2deepscore or another msFeaST-supported score)
- Feature identifiers matching the quantification/metadata tables used elsewhere in the msFeaST pipeline

## Outputs

- Square symmetric pairwise spectral similarity/distance matrix (one row and column per feature)
- Matrix serving as input to k-medoids clustering and t-SNE embedding in subsequent msFeaST steps

## How to apply

Load the processed MS/MS spectra belonging to the features of your quantification table (e.g. the Kathib et al. 2024 example dataset bundled with the msFeaST GitHub repository), ensuring peak handling/preprocessing has made fragment peaks comparable across features. Select a spectral similarity score via the msFeaST spectral similarity module (the pipeline exposes this as user input, highlighted in the notebook, e.g. ms2deepscore in the omsw_pleurotus_ms2deepscore example). Compute pairwise scores between all feature spectra and assemble them into a square, symmetric similarity/distance matrix with one row and column per feature. This matrix is the canonical input to the subsequent k-medoids clustering step, whose clusters serve as a proxy for grouping structurally similar features, and also underpins the t-SNE embedding and the dashboard JSON used by the msFeaST interactive dashboard. Validate that the emitted matrix is square, symmetric, has one row/column per input feature, and that off-diagonal entries lie in the score's valid bounded range.

## Related tools

- **msFeaST** (Pipeline module performing the spectral similarity computation stage; exposes the user-selected similarity score and emits the distance matrix for k-medoids clustering.) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment && jupyter-notebook  # open msfeast_pipeline_mushroom_type_comparison.ipynb and run the spectral similarity cell
```

## Evaluation signals

- Emitted matrix is square with dimensions equal to the number of input features.
- Matrix is symmetric: entry[i][j] equals entry[j][i] for all i, j.
- Diagonal entries reflect self-similarity (e.g. 1.0 for bounded scores).
- All pairwise entries lie within the chosen score's valid bounded range.
- Downstream k-medoids clustering on this matrix yields clusters whose members are structurally similar features consistent with t-SNE embedding in the dashboard.

## Limitations

- The msFeaST pre-processing and pipeline workflow has been tested on macOS and Linux; Windows support is still being worked on (interactive dashboard works regardless of OS in desktop browsers).
- The Python module is installed via conda (python=3.10) with R 4.3.3 pinned inside the same environment; rscript path caching can misinstall R dependencies if the terminal is not restarted after pip install.
- The similarity score is a user-selected input; choosing an inappropriate score for the available spectra (e.g. ms2deepscore without suitable reference embeddings) will degrade cluster quality.
- The GitHub README notes 'No changelog found' for this source, so no versioned behavior guarantees exist beyond the pinned versions above.

## Evidence

- [methods] msFeaST workflow performs spectral similarity computations using the user-selected similarity score, yielding a distance matrix that is subsequently used for k-medoid clustering and t-SNE embedding.: [evidence span withheld — anchor status: not_found]
- [abstract] spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] The current msFeaST pre-processing and pipeline workflow has been tested on macos and should work identically on linux operating systems. Windows support is currently being worked on.: [evidence span withheld — anchor status: external_doc]
