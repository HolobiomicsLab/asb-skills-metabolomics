---
name: cosine-score-threshold-edge-pruning
description: Use when you have a deposited GNPS molecular networking job (.mgf spectra
  plus edge table) and need to isolate spectra unique to one condition, e.g. extracting
  SPF-specific spectra by removing GF-connected features.
when_to_use_negative:
- Input is already a filtered feature table or the spectra have no group metadata
  to distinguish GF from SPF animals
- When the goal is to keep all connected components rather than condition-specific
  spectra — this skill deletes edges to the control group by design
- When spectra come from a single condition with no control group to contrast against
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Source of the .mgf consensus spectra and molecular networking edges used for
    filtering
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Scripting the scan-to-group metadata parsing and edge-based removal of GF-connected
    spectra
- name: microbeMASST
  role: Downstream batch search applied to the pruned SPF-specific spectra set
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:21e5ed94ae2ccefa5f960d6a41c8eb57c851ff87f8f5645477e67704f072e77a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - MZmine
  - Python 3.10
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cosine-score-threshold-edge-pruning

## Summary

Prunes spectra from a GNPS molecular networking job by removing MS/MS features connected via cosine-similarity edges (cosine > 0.7, delta parent mass ±0.02 Da) to spectra found in the control group, retaining the group-specific (SPF-only) consensus spectra for downstream microbeMASST batch searching.

## When to use

Use when you have a deposited GNPS molecular networking job (.mgf spectra plus edge table) and need to isolate spectra unique to one condition, e.g. extracting SPF-specific spectra by removing GF-connected features.

## When NOT to use

- Input is already a filtered feature table or the spectra have no group metadata to distinguish GF from SPF animals
- When the goal is to keep all connected components rather than condition-specific spectra — this skill deletes edges to the control group by design
- When spectra come from a single condition with no control group to contrast against

## Inputs

- GNPS molecular networking job outputs: .mgf consensus spectra file and edge table (e.g. job 893fd89b52dc4c07a292485404f97723)
- Metadata mapping scans to animal group (GF vs SPF)
- Edge table with cosine scores and parent mass deltas

## Outputs

- Filtered .mgf file containing exactly the condition-specific spectra (10,047 SPF-unique spectra)
- Set of scans retained for microbeMASST batch search input

## How to apply

Download the spectra (.mgf) and molecular networking edges files from the deposited GNPS job (e.g. 893fd89b52dc4c07a292485404f97723) and parse the metadata to assign each scan to an animal group (SPF vs GF). Using the edge table, remove any SPF spectra that share a molecular networking edge with a GF-only spectrum, where the edge is defined by cosine similarity > 0.7 and delta parent ion mass within ±0.02 Da. The rationale is that such connected spectra are likely shared microbial/chemical features whose parent ion or adducts differ only slightly, so deleting them isolates condition-specific metabolite signals. Retain the remaining SPF-present spectra and write them into a single .mgf file ready as microbeMASST batch input (generated via MZmine or the GNPS molecular networking workflow).

## Related tools

- **MZmine** (Source of the .mgf consensus spectra and molecular networking edges used for filtering) — https://github.com/mzmine/mzmine
- **Python 3.10** (Scripting the scan-to-group metadata parsing and edge-based removal of GF-connected spectra)
- **microbeMASST** (Downstream batch search applied to the pruned SPF-specific spectra set) — https://github.com/robinschmid/microbe_masst

## Examples

```
python filter_spf_spectra.py --edges edges.tsv --metadata metadata.tsv --cosine 0.7 --delta-mz 0.02 --input spectra.mgf --output spf_specific.mgf
```

## Evaluation signals

- Output .mgf contains exactly 10,047 spectra, matching the filtered input set used for microbeMASST in the paper
- Every retained spectrum has no edge in the edge table with cosine > 0.7 and delta parent ion mass ±0.02 Da to a GF-only spectrum
- All removed spectra are confirmable in the edge table as connected to GF spectra
- Retention set corresponds to spectra explicitly annotated as present in SPF animals

## Limitations

- Thresholds are study-specific: cosine > 0.7 and delta m/z ±0.02 Da; other jobs may need different cutoffs
- Requires group metadata (SPF vs GF) at the scan level; without it, no pruning is possible
- Relies on the deposited GNPS job outputs being available and consistent in format
- Only removes spectra connected to the control group; unconnected spectra present in both groups are retained

## Evidence

- [methods] cosine threshold 0.7 and delta mass 0.02 Da define the edge pruning rule: [evidence span withheld — anchor status: not_found]
- [methods] edge removal targets GF-connected spectra: [evidence span withheld — anchor status: not_found]
- [readme] input .mgf generated via MZmine for batch MASST search: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of"
