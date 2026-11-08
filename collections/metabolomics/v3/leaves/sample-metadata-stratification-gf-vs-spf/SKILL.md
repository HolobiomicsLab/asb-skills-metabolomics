---
name: sample-metadata-stratification-gf-vs-spf
description: Use when you have a GNPS molecular networking job (spectra .mgf plus
  edges file) with sample metadata distinguishing treatment groups and need to retain
  spectra from one group while removing spectra network-connected to another.
when_to_use_negative:
- Input is already a feature table or consensus .mgf without accompanying edge table
  and group metadata — the cosine/±0.02 Da network filter cannot be applied
- The analysis goal is to search ALL spectra from a job rather than spectra unique
  to one sample-metadata group
- Sample groups cannot be distinguished from the metadata (no GF/SPF or equivalent
  treatment labels), making stratification impossible
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine
  role: Generates the single .mgf consensus file from molecular networking in the
    GNPS workflow that is then filtered by group metadata
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Used to parse .mgf/metadata/edge tables, apply the cosine > 0.7 and ±0.02
    Da delta-m/z filters, and write the filtered .mgf
- name: microbeMASST
  role: Downstream batch search target consuming the stratified, SPF-unique .mgf
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:7cae08f0e11ba182960d2b8d2dad407d93abfcba2a70f26d1005633a86cd0b61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - MZmine
  - Python 3.10
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sample-metadata-stratification-gf-vs-spf

## Summary

Stratifies MS/MS spectra from a molecular networking job by sample metadata (germ-free vs SPF mouse groups) to isolate SPF-present spectra, then uses the networking edge table (cosine > 0.7, delta m/z within 0.02 Da) to remove spectra connected to GF-only spectra, producing the SPF-unique consensus set.

## When to use

Use when you have a GNPS molecular networking job (spectra .mgf plus edges file) with sample metadata distinguishing treatment groups and need to retain spectra from one group while removing spectra network-connected to another.

## When NOT to use

- Input is already a feature table or consensus .mgf without accompanying edge table and group metadata — the cosine/±0.02 Da network filter cannot be applied
- The analysis goal is to search ALL spectra from a job rather than spectra unique to one sample-metadata group
- Sample groups cannot be distinguished from the metadata (no GF/SPF or equivalent treatment labels), making stratification impossible

## Inputs

- GNPS molecular networking job outputs: spectra (.mgf) and edge table (molecular networking edges files)
- Sample metadata mapping scans/spectra to germ-free (GF) vs SPF mouse treatment groups
- GNPS job accession (e.g., 893fd89b52dc4c07a292485404f97723)

## Outputs

- Filtered single .mgf file of SPF-unique spectra (10,047 spectra in the paper) suitable as microbeMASST batch input
- Log/count of removed spectra (those with edges to GF-only spectra, cosine > 0.7, delta m/z ± 0.02 Da)

## How to apply

Download the spectra (.mgf) and molecular networking edges files from the deposited GNPS job (e.g., job 893fd89b52dc4c07a292485404f97723), and parse the sample metadata to tag each scan by its group membership (here, presence in SPF vs GF animals). Load the molecular networking edge table and remove any SPF-present spectrum that shares an edge with a GF-only spectrum where the edge cosine similarity exceeds 0.7 and the delta parent ion mass is within ±0.02 Da — these edges indicate the ion is also present in germ-free animals and therefore not microbe-specific. Retain the remaining spectra that are present in SPF animals and write the filtered set to a single .mgf file (10,047 spectra in the paper) formatted as an MZmine-derived molecular networking input for downstream microbeMASST batch searching. Rationale: metadata stratification first, then network-topology filtering, ensures the searched spectra are both observed in SPF mice and not structurally adjacent (by cosine > 0.7, ±0.02 Da) to germ-free spectra.

## Related tools

- **MZmine** (Generates the single .mgf consensus file from molecular networking in the GNPS workflow that is then filtered by group metadata) — https://github.com/mzmine/mzmine
- **Python 3.10** (Used to parse .mgf/metadata/edge tables, apply the cosine > 0.7 and ±0.02 Da delta-m/z filters, and write the filtered .mgf)
- **microbeMASST** (Downstream batch search target consuming the stratified, SPF-unique .mgf) — https://github.com/robinschmid/microbe_masst

## Examples

```
python filter_spf_unique.py --mgf GNPS_893fd89b52dc4c07a292485404f97723.mgf --edges edges.tsv --group SPF --exclude-group GF --cosine 0.7 --delta-mz 0.02 --output spf_unique_10047.mgf
```

## Evaluation signals

- The resulting .mgf must contain exactly 10,047 spectra for the GF/SPF mouse job, matching the filtered input used for microbeMASST searching in the paper
- Every removed spectrum must trace to at least one edge with cosine similarity > 0.7 and delta parent ion mass within ±0.02 Da to a GF-only spectrum in the edge table
- Every retained spectrum must be annotated as present in SPF animals by the sample metadata
- The output .mgf must remain valid as a single MZmine-generated molecular networking file for microbeMASST batch searching
- A spectrum with any GF-connected edge outside the 0.7/0.02 Da cutoff thresholds should be retained, not removed

## Limitations

- Requires complete sample metadata and the full molecular networking edge table; without them the network-topology filter (cosine > 0.7, delta m/z ± 0.02 Da) cannot be reproduced
- Only spectra with edges above cosine 0.7 within ±0.02 Da delta parent ion mass are excluded, so weakly connected or near-identical spectra outside those thresholds may persist as false SPF-unique hits
- The Fast Search API used downstream may fail entries on individual runs; the microbeMASST README advises running jobs.py repeatedly with skip_existing=True until no new output is generated

## Evidence

- [results] SPF-presence and GF-connection filtering with explicit thresholds and expected count: [evidence span withheld — anchor status: not_found]
- [readme] Input format for the microbeMASST batch search is a single MZmine-generated .mgf: [evidence span withheld — anchor status: not_found]
- [readme] Search parameter thresholds that must be checked/adjusted per research question: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks"
- [readme] Downstream Fast Search API reliability caveat and mitigation: "Due to the Fast Search API some of the entries will fail. Nevertheless sequent re-runs should catch all the possible matches."
