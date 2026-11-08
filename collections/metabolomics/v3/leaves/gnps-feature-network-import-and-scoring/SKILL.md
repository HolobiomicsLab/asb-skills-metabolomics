---
name: gnps-feature-network-import-and-scoring
description: Use when starting from a GNPS feature network (.graphml) plus bioassay
  CSVs (or MZmine 2 peak lists / mzML centroids) and you need Activity Scores, Cluster
  Scores, and bioactive communities to prioritize metabolites for isolation.
when_to_use_negative:
- Input is already a processed feature table with computed activity scores — import
  would duplicate prior processing
- Bioassay data columns are not normalized to the same scale, so the data are not
  yet suitable for assembly into biological profiles for NP Analyst
- No replicate data are available and unfiltered raw mass lists are all you have —
  running NP Analyst without preprocessing typically increases the number of candidate
  bioactive (junk) MS features
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: NP Analyst
  role: CLI tool-chain that imports GNPS .graphml networks and normalized bioassay
    CSVs, performs R-tree based basketing, and computes Activity Scores, Cluster Scores,
    and bioactive communities
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Source platform for the molecular networking .graphml files imported with
    the '-t GNPS' type flag
- name: MZmine 2
  role: Alternative upstream platform whose output peak lists are accepted as MS input
    for import and scoring
- name: Docker
  role: Container runtime used to build and execute the npanalyst-cli image with a
    bind-mounted data volume
  repo: https://www.docker.com/get-started
- name: Tableau
  role: Third-party visualization tool for manipulating NP Analyst scored output data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:c74b247b28f6cd40b1f54c8a4803d8203b53a7bd16958fa2c427f278ab82b439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Docker
  - GNPS
  - MZmine 2
  - NP Analyst
  - Tableau
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gnps-feature-network-import-and-scoring

## Summary

Import a GNPS .graphml feature network (or MZmine 2 peak lists / mzML data) together with normalized bioassay CSVs into the npanalyst CLI, then run replicate/basketing and Compound Activity Mapping to compute Activity Scores, Cluster Scores, and bioactive communities. This is the front half of the NP Analyst workflow, converting raw platform outputs into scored MS features.

## When to use

Use when starting from a GNPS feature network (.graphml) plus bioassay CSVs (or MZmine 2 peak lists / mzML centroids) and you need Activity Scores, Cluster Scores, and bioactive communities to prioritize metabolites for isolation.

## When NOT to use

- Input is already a processed feature table with computed activity scores — import would duplicate prior processing
- Bioassay data columns are not normalized to the same scale, so the data are not yet suitable for assembly into biological profiles for NP Analyst
- No replicate data are available and unfiltered raw mass lists are all you have — running NP Analyst without preprocessing typically increases the number of candidate bioactive (junk) MS features

## Inputs

- GNPS molecular networking feature network file (.graphml)
- MZmine 2 output peak lists (alternative MS input format)
- mzML centroided peak-picked mass spectrometry files (alternative MS input format)
- Normalized bioassay CSV files with columns on the same scale for the same extract library (e.g. prefraction antimicrobial screening data)

## Outputs

- Basketed/replicated feature list after R-tree based basketing
- Per-MS-feature Activity Scores and Cluster Scores
- Bioactive communities linking MS features to prefractions
- Output files for visualization (scatter plot, network view, community view), exportable to third-party visualization tools such as Tableau

## How to apply

Import the GNPS .graphml network file with the npanalyst CLI using the '-t GNPS' type flag, and assemble the bioassay results for the same extract library into biological profiles whose columns are normalized to the same scale. Optionally run replicate comparison first — for n technical replicates the default requirement is that a signal is present in n − 1 samples — since replicate comparison significantly reduces noise peaks in untargeted metabolomics data sets. Select a signal intensity cutoff for the MS data that considers the likely limit of detection of the biological assay, as overly low cutoffs dramatically increase network complexity without adding biologically informative MS features. Run the R-tree based basketing stage to build connected-component graphs of m/z features with overlapping error ranges in all dimensions and combine (average) replicated features; parameters are controlled via a config file generated with 'npanalyst get_config' (default npanalyst.configuration.DEFAULT_CONFIG). Then run basketing and Compound Activity Mapping to compute per-feature Activity Scores based on bioactivity profiles and Cluster Scores, and generate output files for visualization (scatter, network, and community views, exportable to tools like Tableau).

## Related tools

- **NP Analyst** (CLI tool-chain that imports GNPS .graphml networks and normalized bioassay CSVs, performs R-tree based basketing, and computes Activity Scores, Cluster Scores, and bioactive communities) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Source platform for the molecular networking .graphml files imported with the '-t GNPS' type flag)
- **MZmine 2** (Alternative upstream platform whose output peak lists are accepted as MS input for import and scoring)
- **Docker** (Container runtime used to build and execute the npanalyst-cli image with a bind-mounted data volume) — https://www.docker.com/get-started
- **Tableau** (Third-party visualization tool for manipulating NP Analyst scored output data)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- Import runs successfully with the '-t GNPS' type flag and verbose output on the .graphml file without schema errors
- Basketing combines m/z features with overlapping error ranges in all dimensions, so replicated features appear as single averaged entries rather than separate rows
- Downstream scoring yields Activity Scores and Cluster Scores for each MS feature, and bioactive communities link MS features to member prefractions (e.g. in the amychelin C validation, community 16 must contain 5 prefractions connected by 3 MS features including [M+H]+ m/z 753.3243 and fragment 623.2319)
- Config file generated via 'npanalyst get_config' (config.json) matches npanalyst.configuration.DEFAULT_CONFIG or documents intentional parameter changes
- Feature scores are reproducible: re-running the full pipeline (pytest tests/integration) gives consistent Activity Score / Cluster Score assignments

## Limitations

- Replicate comparison is only possible where replicate data are available ('strongly recommended where possible'); for n replicates the default requirement is presence in n − 1 samples
- Running NP Analyst without data preprocessing increases the number of candidate bioactive MS features because raw mass lists include junk features
- NP Analyst scores each MS feature independently, so all MS features from one bioactive molecule (adducts, fragments, multiply-charged species) get similar Activity and Cluster Scores but different m/z ratios, requiring manual interpretation of feature stripes; some communities lack clear individual bioactive MS features and require a retention-time stripe strategy instead
- Low minimum signal intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features, especially when the bioassay is insensitive

## Evidence

- [readme] README import command for GNPS .graphml: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"
- [readme] README R-tree basketing rationale: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
- [intro] Accepted MS input formats: [evidence span withheld — anchor status: not_found]
- [discussion] Normalization requirement for bioassay data: "these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale"
- [intro] CLI availability via Docker: "the underlying processing algorithms are freely available as a command line tool via a Docker container at https://github.com/liningtonlab/npanalyst"
