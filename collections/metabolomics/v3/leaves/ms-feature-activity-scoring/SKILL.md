---
name: ms-feature-activity-scoring
description: Use when you have processed MS feature data (MZmine 2 or GNPS outputs,
  mzML-derived peak lists) for an extract library plus bioassay results, and need
  to score which features match the biological profile for prioritization or dereplication.
when_to_use_negative:
- Input MS data are raw peak lists without preprocessing (replicate comparison, basketing,
  intensity cutoff) — raw mass lists add junk features and inflate candidate bioactive
  features
- Bioassay columns are not normalized to a common scale — the Activity Score sums
  squared values across assays and unscaled columns would dominate the score
- The goal is de novo MS/MS spectrum identification rather than linking features to
  bioactivity profiles — NP Analyst is a target-agnostic activity-mapping tool, not
  a spectral annotation engine
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NP Analyst
  role: 'Open online platform / CLI tool-chain that performs Compound Activity Mapping:
    import, basketing, feature scoring (Activity Score, Cluster Score), and output
    generation'
  repo: https://github.com/liningtonlab/npanalyst
- name: MZmine 2
  role: Produces the processed MS feature peak lists (unique MS features with m/z
    and retention time) used as scoring input
- name: GNPS
  role: Provides molecular network graphML files accepted by npanalyst import as a
    feature-table source
- name: Tableau
  role: Third-party visualization tool for manipulating NP Analyst output data
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:287d56897d6f93280b1e0fb2d7e625d6bf9759374626b52e9192020647fe7e4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  - Tableau
  - npanalyst CLI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# ms-feature-activity-scoring

## Summary

Computes NP Analyst Activity Scores and Cluster Scores for each mass spectrometry feature by integrating a basketed MS feature table (features x samples with m/z and retention time) with normalized bioassay data (samples x bioassay columns), to prioritize which metabolites in an extract library best explain the observed bioactivity.

## When to use

Use when you have processed MS feature data (MZmine 2 or GNPS outputs, mzML-derived peak lists) for an extract library plus bioassay results, and need to score which features match the biological profile for prioritization or dereplication.

## When NOT to use

- Input MS data are raw peak lists without preprocessing (replicate comparison, basketing, intensity cutoff) — raw mass lists add junk features and inflate candidate bioactive features
- Bioassay columns are not normalized to a common scale — the Activity Score sums squared values across assays and unscaled columns would dominate the score
- The goal is de novo MS/MS spectrum identification rather than linking features to bioactivity profiles — NP Analyst is a target-agnostic activity-mapping tool, not a spectral annotation engine

## Inputs

- Aligned/basketed MS feature table (CSV of features x samples with retention time and m/z columns; e.g. MZmine 2 output peak lists or GNPS graphML network files)
- Normalized bioassay data (CSV of samples x bioassay columns, all columns on the same scale)
- config.json configuration file (generated via 'npanalyst get_config')
- Optional: replicate-group assignment for replicate comparison filtering

## Outputs

- Feature table with appended Activity Score and Cluster Score columns (scored_features.csv)
- Ranked prioritization of MS features by Activity/Cluster Score for dereplication and isolation targeting

## How to apply

First prepare inputs: an aligned MS feature table (CSV of features x samples with retention time and m/z columns, e.g. from MZmine 2 or a GNPS graphML file imported via the npanalyst CLI) and a bioassay CSV normalized so every column is on the same scale. Compute each feature's Activity Score by summing, over all bioassay columns, the mean of the squares of the normalized bioactivity values for the samples/extracts containing that feature. For each pair of co-occurring features, correlate their binary biological fingerprints (the set of assays in which each is active) using Pearson similarity, then compute the Cluster Score as the mean Pearson similarity of a feature to its co-active partners. Apply the recommended pre-filters: replicate comparison (present in n - 1 samples for n replicates) to remove noise peaks, and a signal intensity cutoff chosen with the assay's likely limit of detection in mind. Append Activity Score and Cluster Score columns to the feature table and inspect prioritized features in the scatter, network, and community views (e.g. retention time vs m/z stripes), remembering NP Analyst scores each MS feature independently so all adducts/fragments of one molecule carry similar scores.

## Related tools

- **NP Analyst** (Open online platform / CLI tool-chain that performs Compound Activity Mapping: import, basketing, feature scoring (Activity Score, Cluster Score), and output generation) — https://github.com/liningtonlab/npanalyst
- **MZmine 2** (Produces the processed MS feature peak lists (unique MS features with m/z and retention time) used as scoring input)
- **GNPS** (Provides molecular network graphML files accepted by npanalyst import as a feature-table source)
- **Tableau** (Third-party visualization tool for manipulating NP Analyst output data)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- Each MS feature row in the output carries both an Activity Score and a Cluster Score column; the Activity Score equals the sum over bioassay columns of the mean of squared normalized bioactivity values across samples containing the feature
- Cluster Score is the mean Pearson correlation of the feature's binary biological fingerprint to those of co-active partner features (features co-occurring in at least one sample)
- All MS features derived from the same molecule (adducts, fragments, multiply-charged forms) show similar Activity and Cluster Scores but different m/z values, appearing as retention-time stripes in the community view
- A verified prioritization can be checked against a known compound: e.g. community 12's single prioritized feature must be m/z 452.2788 at retention time 3.12 min connecting 16 of 23 prefractions and matching micromonolactam's distribution
- Output validates against the pipeline's pytest integration tests (pytest tests/integration) in the npanalyst repo

## Limitations

- Running NP Analyst without data preprocessing increases the number of candidate bioactive MS features because raw mass lists include junk features
- Low minimum signal intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features, especially when the assay is insensitive
- Replicate comparison is only possible where replicate data exist; NP Analyst scores each MS feature independently, so manual interpretation of adduct/fragment/multiply-charged feature stripes is required
- Some communities lack clear individual bioactive MS features, requiring the retention-time stripe strategy (sets of active features sharing the same retention time) instead of single-feature prioritization

## Evidence

- [other] Activity Score is determined by summing, over all bioassay columns, the mean of the squares of the bioactivity values for each extract containing a given mass spectrometric feature.: [evidence span withheld — anchor status: not_found]
- [intro] feature score determination based on bioactivity profiles: [evidence span withheld — anchor status: not_found]
- [discussion] these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale: "the data are normalized so that each column uses the same scale"
- [discussion] Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets and is strongly recommended where possible: "Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets"
- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged) and represent a replicated or basketed feature.: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)"
- [methods] docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml"
