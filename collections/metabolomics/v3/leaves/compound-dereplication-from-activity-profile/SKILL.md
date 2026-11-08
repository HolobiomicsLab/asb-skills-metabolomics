---
name: compound-dereplication-from-activity-profile
description: Use when a prefractionated extract library has both processed MS feature
  data (mzML, MZmine 2 peak lists, or GNPS graphML/CSV) and normalized bioassay columns,
  and the goal is to identify which MS feature explains the community's activity profile
  and dereplicate the compound behind it.
when_to_use_negative:
- The input is raw, unprocessed mass spectrometry rather than MZmine 2/GNPS peak lists
  or mzML — running NP Analyst without preprocessing typically increases the number
  of candidate bioactive MS features with junk peaks
- The bioactive community has no clear individual bioactive MS feature — use the retention-time
  stripe strategy instead (identify sets of active MS features with the same retention
  time in the m/z vs retention time plot)
- Bioassay columns are not normalized to the same scale, since the data are only suitable
  for NP Analyst if each column uses the same scale
tools:
- name: NP Analyst
  role: Open platform that computes Activity Scores and Cluster Scores relating MS
    features to bioactivity profiles; used via its Docker-wrapped CLI (import, basketing,
    scoring)
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Supplies molecular-network graphML files and processed MS feature data that
    serve as the input peak/graph layer for NP Analyst
- name: MZmine 2
  role: Supplies output peak lists of unique MS features that NP Analyst accepts as
    an alternative MS input format
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:8a17e77bad236cd0cac9a18c29e1292df00417c3dc5131a005f4f1a51a5813d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  - npanalyst CLI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# compound-dereplication-from-activity-profile

## Summary

Dereplicate and prioritize known compounds by matching MS feature distributions across prefractions to normalized bioassay profiles using NP Analyst, which computes Activity Scores and Cluster Scores for each MS feature from MZmine 2 or GNPS outputs. In the source case this flagged the micromonolactam feature (m/z 452.2788, rt 3.12 min) rather than the co-purifying, inactive dracolactams.

## When to use

Use when a prefractionated extract library has both processed MS feature data (mzML, MZmine 2 peak lists, or GNPS graphML/CSV) and normalized bioassay columns, and the goal is to identify which MS feature explains the community's activity profile and dereplicate the compound behind it.

## When NOT to use

- The input is raw, unprocessed mass spectrometry rather than MZmine 2/GNPS peak lists or mzML — running NP Analyst without preprocessing typically increases the number of candidate bioactive MS features with junk peaks
- The bioactive community has no clear individual bioactive MS feature — use the retention-time stripe strategy instead (identify sets of active MS features with the same retention time in the m/z vs retention time plot)
- Bioassay columns are not normalized to the same scale, since the data are only suitable for NP Analyst if each column uses the same scale

## Inputs

- Processed MS feature data in mzML, MZmine 2 peak-list CSV, or GNPS graphML format
- Bioassay results for the same extract library as CSV columns normalized to the same scale
- Known compound bioactivity/distribution profiles for dereplication (e.g., micromonolactam vs dracolactams A/C)

## Outputs

- Activity Score and Cluster Score per MS feature
- Community and network views (scatter, network, community) of bioactive chemical space
- Prioritized MS feature attributes (m/z, retention time, prefraction connection count) and dereplication conclusion, e.g. results_community12_verification.json

## How to apply

Assemble individual bioassay results for the same extract library into biological profiles whose columns are normalized on the same scale, then import processed MS data (GNPS graphML or MZmine 2 peak lists) and the bioactivity CSVs with the npanalyst CLI. Run R-tree based basketing, which builds connected component graphs of m/z features with overlapping error ranges in all dimensions and combines them (averaged) into replicated/basketed features. NP Analyst then computes Activity Score and Cluster Score per MS feature; because each feature is scored independently, all features from one molecule (adducts, fragments, multiply-charged species) will show similar Activity and Cluster Scores but different m/z ratios, so interpret adduct/fragment retention-time stripes manually. Use the scatter plot, network, and community views interdependently to pick the priority feature — in the reproduced case, m/z 452.2788 at rt 3.12 min, which connects 16 of 23 prefractions in community 12. Dereplicate by matching the prioritized feature's prefraction distribution against known compound profiles: micromonolactam's distribution matched community 12's activity, while the co-isolated dracolactams A and C were largely inactive. Parameters can be tuned via a config file generated with `npanalyst get_config` (defaults to npanalyst.configuration.DEFAULT_CONFIG).

## Related tools

- **NP Analyst** (Open platform that computes Activity Scores and Cluster Scores relating MS features to bioactivity profiles; used via its Docker-wrapped CLI (import, basketing, scoring)) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Supplies molecular-network graphML files and processed MS feature data that serve as the input peak/graph layer for NP Analyst)
- **MZmine 2** (Supplies output peak lists of unique MS features that NP Analyst accepts as an alternative MS input format)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The reproduced priority feature must match the expected attributes: m/z 452.2788, retention time 3.12 min, connecting 16 of 23 prefractions
- The dereplication conclusion must name the compound whose distribution matches the community's activity profile (micromonolactam), not the co-purifying but largely inactive dracolactams A/C
- Replicate comparison should be applied before scoring (default requirement: a signal present in n − 1 samples for n replicates) to remove noise peaks
- Config file must be reproducible: `npanalyst get_config` produces ./config.json; parameters are documented via npanalyst --help
- Repository tests pass: pytest tests/unit and pytest tests/integration

## Limitations

- NP Analyst scores each MS feature independently, so manual interpretation of adduct/fragment/multiply-charged feature stripes is required
- Replicate comparison is only possible 'where possible' — replicate data are not always available
- Low minimum signal-intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features; choose cutoffs with the assay's limit of detection in mind
- Communities without clear individual bioactive MS features require the retention-time stripe strategy rather than single-feature prioritization

## Evidence

- [abstract] We demonstrate the utility of this new tool for both the dereplication of known compounds and the discovery of novel bioactive natural products: "We demonstrate the utility of this new tool for both the dereplication of known compounds and the discovery of novel bioactive natural products"
- [discussion] NP Analyst offers a target-agnostic platform for predicting biological activities of MS features directly from complex mixtures: "NP Analyst offers a target-agnostic platform for predicting biological activities of MS features directly from complex mixtures"
- [discussion] all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
- [discussion] these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale: "these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale"
