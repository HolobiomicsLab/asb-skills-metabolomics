---
name: mass-spectrometry-feature-retention-curation
description: Use when a GNPS molecular networking job has produced .mgf and edges
  files and you need to select a subset of spectra (e.g. SPF-unique) before batch
  MASST searching.
when_to_use_negative:
- The spectra set is already a curated .mgf ready for microbeMASST input (no control-based
  filtering needed)
- The analysis goal is to search all detected features rather than condition-specific
  ones
- The underlying data are raw instrument files (mzML/RAW) rather than GNPS molecular
  networking outputs
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: generates the single .mgf file (via molecular networking) that serves as the
    input for filtering
  repo: https://github.com/mzmine/mzmine
- name: microbeMASST
  role: the batch search tool that consumes the curated .mgf after feature retention
    filtering
  repo: https://masst.gnps2.org/microbemasst/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:82bcd9557b11df2b83c3373e06bc4f3f9e46a717675c94acc189143d8945c179
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - MZmine
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

# mass-spectrometry-feature-retention-curation

## Summary

Curates the set of MS/MS spectra destined for a batch search by retaining only features of experimental interest and removing spectra linked to a control condition through molecular-network edges, so that downstream MASST-style searching is focused and interpretable.

## When to use

Use when a GNPS molecular networking job has produced .mgf and edges files and you need to select a subset of spectra (e.g. SPF-unique) before batch MASST searching.

## When NOT to use

- The spectra set is already a curated .mgf ready for microbeMASST input (no control-based filtering needed)
- The analysis goal is to search all detected features rather than condition-specific ones
- The underlying data are raw instrument files (mzML/RAW) rather than GNPS molecular networking outputs

## Inputs

- .mgf spectra file from a GNPS molecular networking job
- molecular networking edges file (cosine similarity, delta parent ion mass) from the GNPS job
- sample metadata indicating which scans appear in which biological condition (e.g. SPF vs GF)

## Outputs

- single filtered .mgf file containing only the curated spectra (e.g. 10,047 SPF-unique spectra)
- optionally a table of removed spectra and the edges that triggered removal

## How to apply

Download the spectra (.mgf) and molecular networking edges files from the deposited GNPS job (e.g. 893fd89b52dc4c07a292485404f97723). Parse sample metadata to determine which scans are present in the condition of interest (e.g. SPF animals) and which occur only in the control (GF animals). Using the edge table, remove spectra that connect — via cosine similarity > 0.7 and delta parent ion mass within ± 0.02 Da — to control-only spectra, then write the remaining condition-present spectra to a single .mgf file. The rationale is that edge-linked features often represent analogues or adducts of control-derived metabolites; excluding them prevents microbeMASST searches being dominated by such relatives.

## Related tools

- **MZmine** (generates the single .mgf file (via molecular networking) that serves as the input for filtering) — https://github.com/mzmine/mzmine
- **microbeMASST** (the batch search tool that consumes the curated .mgf after feature retention filtering) — https://masst.gnps2.org/microbemasst/

## Examples

```
python code/jobs.py  # with the filtered 10,047-spectrum .mgf added as ("input/input.mgf", "output/spf_unique")
```

## Evaluation signals

- the resulting .mgf contains exactly the expected number of spectra (e.g. 10,047 in the GF/SPF mouse case)
- every removed scan has at least one edge with cosine > 0.7 and delta m/z within ± 0.02 Da to a control-condition spectrum
- no scan retained in the output .mgf is edge-connected to a control-only spectrum under those thresholds
- all retained scans correspond to features observed in the condition-of-interest samples according to the metadata
- re-running the filtering on the same inputs reproduces the identical filtered set

## Limitations

- Filtering depends on correct condition metadata; mislabeled samples propagate errors into the curated set
- Thresholds (cosine > 0.7, Δm/z ± 0.02 Da) are study-specific and may need re-tuning for other datasets
- The Fast Search API used downstream can fail on some entries, requiring repeated re-runs of jobs.py with skip_existing=True; the filtering itself does not address that

## Evidence

- [results] From the GNPS molecular networking job of the GF/SPF mouse study, spectra present in SPF animals (n = 10,047) were retained, and all MS/MS spectra with an edge (cosine similarity > 0.7) and a delta parent ion mass of ± 0.02 Da with MS/MS spectra present in GF animals were removed.: [evidence span withheld — anchor status: not_found]
- [methods] Batch search of multiple MS/MS spectra against indexed data and domainMASSTs: "execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE"
- [readme] You can run either a single .mgf file generated via MZmine, from the molecular networking in GNPS workflow, or a list of USIs provided either via a .csv or .tsv file.: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow"
