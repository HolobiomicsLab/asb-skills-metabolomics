---
name: mzml-centroided-peak-import
description: Use when raw metabolomics data are supplied as individual centroided
  mzML files (or per-replicate peak-list CSVs) and a deduplicated list of unique MS
  features is needed for downstream Compound Activity Mapping or bioactivity integration.
when_to_use_negative:
- Input is already an MZmine 2 output peak list or a GNPS network (.graphml) file
  — use the GNPS/MZmine import type instead of the mzML replicate pipeline
- Replicate injections are not available — replicate comparison requires n replicates
  and the paper notes it is only possible 'where possible'
- Data have not been peak-picked/centroided — the pipeline expects centroided peak
  lists, not profile-mode spectra
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP Analyst
  role: CLI tool-chain whose `import` step (-t mzML) standardizes mzML-derived peak
    lists and whose replicate-comparison and R-tree basketing tasks produce the unique-feature
    list
  repo: https://github.com/liningtonlab/npanalyst
- name: Docker
  role: Container runtime for building the npanalyst-cli image and running import/basketing
    with a bind-mounted data volume
  repo: https://www.docker.com/get-started
- name: pandas
  role: Used to load the per-replicate centroided peak-list CSVs prior to import
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:1139c45ff1a86d0c96c1e117c300526f9d23261ca4fb8132438729af88bed452
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Docker
  - NP Analyst
  - pandas
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# mzml-centroided-peak-import

## Summary

Imports peak-picked centroided mzML-derived peak lists into the NP Analyst tool-chain and reduces them to a single list of unique MS features via replicate comparison (present in n−1 of n replicates) followed by R-tree based basketing of overlapping m/z and retention-time error ranges.

## When to use

Use when raw metabolomics data are supplied as individual centroided mzML files (or per-replicate peak-list CSVs) and a deduplicated list of unique MS features is needed for downstream Compound Activity Mapping or bioactivity integration.

## When NOT to use

- Input is already an MZmine 2 output peak list or a GNPS network (.graphml) file — use the GNPS/MZmine import type instead of the mzML replicate pipeline
- Replicate injections are not available — replicate comparison requires n replicates and the paper notes it is only possible 'where possible'
- Data have not been peak-picked/centroided — the pipeline expects centroided peak lists, not profile-mode spectra

## Inputs

- Peak-picked centroided mzML files from major instrument manufacturers
- One CSV peak list per replicate injection derived from the mzML data
- Optional npanalyst config.json (generated via `npanalyst get_config`) with rt/m/z tolerance settings

## Outputs

- A single CSV file listing all unique (basketed) MS features and their sample distribution, e.g. basketed_mzml_features.csv

## How to apply

Load the peak-picked centroided mzML-derived peak lists (one CSV per replicate injection, e.g. with pandas) and standardize them with the npanalyst CLI import step (`-t mzML`), either via a local conda/pip install or the Docker image with a bind-mounted /data volume. Apply replicate comparison with the default requirement that a signal must be present in n−1 samples for n replicates so only replicated features are retained; the paper notes this step 'can significantly reduce noise peaks in untargeted metabolomics data sets'. Then run R-tree based basketing: the README describes using 'an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)' constrained by rt and m/z tolerances, and combining (averaging) each connected component into a single basketed feature. Consider a signal intensity cutoff tuned to the biological assay's likely limit of detection, since overly permissive cutoffs inflate network complexity without adding biologically informative MS features. Write the resulting unique-feature peak list (e.g. basketed_mzml_features.csv) for downstream activity mapping.

## Related tools

- **NP Analyst** (CLI tool-chain whose `import` step (-t mzML) standardizes mzML-derived peak lists and whose replicate-comparison and R-tree basketing tasks produce the unique-feature list) — https://github.com/liningtonlab/npanalyst
- **Docker** (Container runtime for building the npanalyst-cli image and running import/basketing with a bind-mounted data volume) — https://www.docker.com/get-started
- **pandas** (Used to load the per-replicate centroided peak-list CSVs prior to import)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/<mzml_peaklist>.csv -o /data -t mzML -v
```

## Evaluation signals

- The output file exists and parses as a CSV peak list of unique MS features
- Feature count is reduced relative to the replicate-comparison input, confirming junk/noise peaks were removed
- Each retained replicated feature satisfies the n−1-of-n replicate presence rule
- Basketed features result from averaging connected components of features with overlapping error ranges in both m/z and rt dimensions

## Limitations

- Running NP Analyst without data preprocessing typically increases the number of candidate bioactive MS features because raw mass lists include junk features
- Replicate comparison is only possible when replicate data are available ('strongly recommended where possible')
- Low signal intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features when the assay is insensitive
- Parameters live in a config file (default npanalyst.configuration.DEFAULT_CONFIG) that may need tuning as things get tuned

## Evidence

- [intro] For n replicates, the default requirement is present in n – 1 samples: "For n replicates, the default requirement is present in n – 1 samples"
- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions).: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
- [discussion] Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets29,31,32 and is strongly recommended where possible: "Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets29,31,32 and is strongly recommended where possible"
- [abstract] compatible with mass spectrometry data from major instrument manufacturers via the mzML format: "accepts bioassay data of almost any type, and is compatible with mass spectrometry data from major instrument manufacturers via the mzML format"
- [methods] You can use [Docker](https://www.docker.com/get-started) to run the `npanalyst` CLI by binding a local volume.: "You can use [Docker](https://www.docker.com/get-started) to run the `npanalyst` CLI by binding a local volume."
