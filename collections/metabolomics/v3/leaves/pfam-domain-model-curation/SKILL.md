---
name: pfam-domain-model-curation
description: Use when building or rebuilding a reduced pHMM feature library (e.g.,
  BiG-SLiCE feature set 1) and you need to select only biosynthetically relevant Pfam
  families rather than scanning against all of Pfam.
when_to_use_negative:
- The goal is a full unbiased domain scan — this skill deliberately reduces Pfam to
  biosynthetic families, so non-biosynthetic domains will be missed
- The HMM database already exists and matches the required version — use the bundled
  download_bigslice_hmmdb utility instead of rebuilding
- BGC detection is needed rather than feature annotation — use BGC predictors such
  as antiSMASH, ClusterFinder, or DeepBGC, not pHMM library curation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0780
tools:
- name: BiG-SLiCE
  role: Hosts and distributes the curated biosynthetic-Pfam and sub-pfam model sets
    via download_bigslice_hmmdb; integrity is checked with bigslice --version
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH 5.0
  role: Supplies 250 pHMM models merged with the curated Pfam-derived set to form
    the biosynthetic-Pfam model library
- name: pyhmmer
  role: Python HMMER3 bindings used by BiG-SLiCE 2.0 to scan BGC proteins against
    the curated pHMM database
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:a4eb853f32c4702cef029baacd1a3c26e5928d318677f78cf3986f09ebc41859
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - antiSMASH 5.0
  - pyhmmer
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pfam-domain-model-curation

## Summary

Constructs a reduced 'biosynthetic-Pfam' pHMM library by filtering the full Pfam domain database down to biosynthetically related protein families, merging curated Pfam-derived models with antiSMASH pHMM models, and packaging them with fragmented sub-pfam models into a reference HMM database for BGC feature annotation.

## When to use

Use when building or rebuilding a reduced pHMM feature library (e.g., BiG-SLiCE feature set 1) and you need to select only biosynthetically relevant Pfam families rather than scanning against all of Pfam.

## When NOT to use

- The goal is a full unbiased domain scan — this skill deliberately reduces Pfam to biosynthetic families, so non-biosynthetic domains will be missed
- The HMM database already exists and matches the required version — use the bundled download_bigslice_hmmdb utility instead of rebuilding
- BGC detection is needed rather than feature annotation — use BGC predictors such as antiSMASH, ClusterFinder, or DeepBGC, not pHMM library curation

## Inputs

- Full Pfam HMM library (e.g., Pfam v32; PFAM 35.0 in BiG-SLiCE 2.0)
- antiSMASH 5.0 pHMM models (250 models)
- ECDomainMiner filter output (candidate biosynthetic domain list)
- Per-domain full descriptions for manual curation

## Outputs

- Curated biosynthetic-Pfam pHMM model set (Supplementary Table S1)
- Fragmented sub-pfam pHMM model set
- Packaged BiG-SLiCE HMM database release with integrity checksums

## How to apply

Retrieve the reference HMM libraries — the Pfam v32 HMM library (or later, e.g., PFAM 35.0 in BiG-SLiCE 2.0) and the antiSMASH pHMM models — as raw material. Apply ECDomainMiner to filter the full Pfam domain set down to families related to enzymatic/biosynthetic functions, then manually curate the candidate list based on each domain's full description to remove false positives. Merge the 250 antiSMASH pHMM models with the curated Pfam-derived set to assemble the biosynthetic-Pfam model set catalogued in Supplementary Table S1, and build the fragmented sub-pfam model set alongside the full-length models. Package both sets into the BiG-SLiCE HMM database release, which is distributed via the bundled download_bigslice_hmmdb utility and verified with bigslice --version integrity identifiers.

## Related tools

- **BiG-SLiCE** (Hosts and distributes the curated biosynthetic-Pfam and sub-pfam model sets via download_bigslice_hmmdb; integrity is checked with bigslice --version) — https://github.com/medema-group/bigslice
- **antiSMASH 5.0** (Supplies 250 pHMM models merged with the curated Pfam-derived set to form the biosynthetic-Pfam model library)
- **pyhmmer** (Python HMMER3 bindings used by BiG-SLiCE 2.0 to scan BGC proteins against the curated pHMM database) — https://github.com/althonos/pyhmmer

## Examples

```
mamba env create -n bigslice -c conda-forge -c bioconda bigslice; mamba activate bigslice; download_bigslice_hmmdb; bigslice --version
```

## Evaluation signals

- bigslice --version prints the expected integrity identifiers: biosynthetic-pfam md5 37495cac452bf1dd8aff2c4ad92065fe and sub-pfam md5 2e6b41d06f3c318c61dffb022798091e
- The assembled model set matches the entries catalogued in Supplementary Table S1
- download_bigslice_hmmdb completes and the fetched release contains both the biosynthetic-pfam and sub-pfam model sets
- Manual curation has removed ECDomainMiner false positives — every retained domain's full description supports a biosynthetic function
- The antiSMASH-derived subset contributes exactly 250 pHMM models to the merged library

## Limitations

- Requires manual curation of ECDomainMiner candidates — the automated filter alone produces false positives that must be removed by inspecting each domain's full description
- BiG-SLiCE 2.0 switched to PFAM 35.0 and pyHMMER, so a library rebuilt from Pfam v32 will not match the v2.0 release checksums unless the older release (archived v1.0.0 on Zenodo) is used

## Evidence

- [supplementary] biosynthetic-Pfam list construction: [evidence span withheld — anchor status: not_found]
- [supplementary] merge with antiSMASH models: [evidence span withheld — anchor status: not_found]
- [supplementary] validation checksums: [evidence span withheld — anchor status: not_found]
- [readme] pyHMMER switch: "BiG-SLiCE 2.0 uses PyHMMER — a Python-native HMM scanning library — replacing the standalone HMMER suite required in BiG-SLiCE 1.x."
- [readme] PFAM 35.0 update: "pHMM databases have been updated to __PFAM 35.0__"
