---
name: gnps-graphml-feature-import
description: Use when the input mass spectrometry data is a GNPS network file (GNPS.graphml)
  and MS features must be imported and basketed as the first npanalyst CLI step before
  activity-score computation.
when_to_use_negative:
- The input is already a basketed/replicated feature CSV folder — skip import and
  go straight to scoring/basketing of the CSVs.
- The input is MZmine 2 output peak lists rather than a GNPS network file — import
  with the MZmine type instead of `-t GNPS`.
- The data is raw vendor or unprocessed mzML that has not yet been turned into a processed
  MS feature list (via MZmine 2 or GNPS), which would inflate the candidate bioactive
  features with junk peaks.
tools:
- name: NP Analyst
  role: npanalyst CLI provides the `import` subcommand that parses the GNPS graphML
    and performs R-tree based basketing of m/z features
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Source platform that produced the graphML network file of MS features consumed
    by the import step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:efd5851e914ba237331bbe1a2ff317c56b9aedfc4bce6a5e289d2420b3df8e7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - NP Analyst
  - npanalyst CLI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gnps-graphml-feature-import

## Summary

Import a GNPS molecular-networking graphML file into the npanalyst CLI, which parses its m/z features and bins overlapping features via R-tree basketing so MS features can later be scored against bioassay data (Activity Score and Cluster Score).

## When to use

Use when the input mass spectrometry data is a GNPS network file (GNPS.graphml) and MS features must be imported and basketed as the first npanalyst CLI step before activity-score computation.

## When NOT to use

- The input is already a basketed/replicated feature CSV folder — skip import and go straight to scoring/basketing of the CSVs.
- The input is MZmine 2 output peak lists rather than a GNPS network file — import with the MZmine type instead of `-t GNPS`.
- The data is raw vendor or unprocessed mzML that has not yet been turned into a processed MS feature list (via MZmine 2 or GNPS), which would inflate the candidate bioactive features with junk peaks.

## Inputs

- GNPS molecular networking graphML file (GNPS.graphml)
- Docker image built from the npanalyst repository (npanalyst-cli)
- Local data directory bind-mounted to /data

## Outputs

- Imported/basketed MS feature CSVs in the output folder, ready for activity-scoring and community analysis
- Optional config.json from `npanalyst get_config`

## How to apply

Run the npanalyst CLI inside Docker with a bind-mounted data volume, invoking the `import` subcommand with `-i /data/GNPS.graphml`, `-o /data`, and type `-t GNPS` so the parser reads the GNPS graphML structure. The importer builds connected-component graphs of m/z features with overlapping error ranges in all dimensions (R-tree Based Basketing), then combines and averages them into basketed/replicated features written to the output folder. These imported, basketed feature CSVs are the substrate for the downstream workflow: replicate comparison, bioactivity-profile scoring (Activity Score and Cluster Score), and community/community-12-style dereplication (e.g., prioritizing m/z 452.2788 at RT 3.12 min present in 16 of 23 prefractions). Generate a `config.json` via `npanalyst get_config` if defaults need tuning, since many parameters are specified through `npanalyst.configuration.DEFAULT_CONFIG`.

## Related tools

- **NP Analyst** (npanalyst CLI provides the `import` subcommand that parses the GNPS graphML and performs R-tree based basketing of m/z features) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Source platform that produced the graphML network file of MS features consumed by the import step)

## Examples

```
docker build -t npanalyst-cli . && docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The import command runs with `-t GNPS` and `-v` and completes without errors on the bound /data volume.
- Output folder contains basketed feature CSVs derived from the graphML, with m/z and retention-time attributes preserved (e.g., features resolvable to m/z 452.2788 at RT 3.12 min).
- Basketed features overlap-error-range in all dimensions per the R-tree basketing description — replicated features are averaged, not duplicated.
- Downstream recomputation of Activity Score and Cluster Score on the imported features reproduces the expected prioritization (e.g., a single feature connecting 16 of 23 prefractions in community 12).
- `npanalyst --help` shows the import subcommand and `npanalyst get_config` produces a usable config.json when defaults need inspection.

## Limitations

- Replicate comparison, which reduces noise peaks, requires n-1-of-n replicate presence (default for n replicates) and is only possible where replicate data exist.
- Running NP Analyst without data preprocessing (e.g., importing raw mass lists) increases the number of candidate bioactive MS features with junk features.
- NP Analyst scores each MS feature independently, so adduct/fragment/multiply-charged stripes of the same molecule receive similar scores at different m/z and need manual interpretation.
- Low signal-intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features.

## Evidence

- [readme] docker import invocation: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"
- [readme] R-tree basketing description: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
- [readme] npanalyst purpose: "Tool-chain for relating metabolite MS data with bio-activity data."
- [intro] GNPS network file input support: [evidence span withheld — anchor status: not_found]
- [abstract] processed MS data compatibility: "NP Analyst will accept processed mass spectrometry data from the MZmine 2 and GNPS open-source platforms"
