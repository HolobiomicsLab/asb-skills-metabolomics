---
name: antismash-phmm-model-merging
description: 'Use when reconstructing BiG-SLiCE feature set 1: a curated biosynthetic
  pHMM library (full-length biosynthetic-Pfam plus fragmented sub-pfam models) that
  replaces full Pfam for BGC feature computation.'
when_to_use_negative:
- Running BiG-SLiCE 2.0, which already ships an updated HMM database built on PFAM
  35.0 — rebuilding from Pfam v32 would produce a library inconsistent with v2's cosine-like
  clustering and antiSMASH v7.0.0 class definitions
- The goal is de novo BGC detection from genome sequences — that is antiSMASH's job;
  this skill only assembles the feature-model library used after BGC prediction
- Input is already a per-cluster domain/feature table — no HMM model merging is needed
  at that stage
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: Consumes the merged biosynthetic-Pfam/sub-pfam HMM database and ships the
    download_bigslice_hmmdb utility plus bigslice --version integrity identifiers
    used to validate the rebuild
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH 5.0
  role: Source of the 250 pHMM models merged with the curated Pfam-derived set to
    form the biosynthetic-Pfam library
- name: pyhmmer
  role: Python-native HMM scanning engine used by BiG-SLiCE 2.0 to apply the merged
    pHMM database, replacing the standalone HMMER suite of v1.x
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:b863d88e67fdc4fc8cb8b6fa36c64a9482d662b278298c9ccf20d7216879c436
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - antiSMASH 5.0
  - pyhmmer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# antismash-phmm-model-merging

## Summary

Assembles BiG-SLiCE's reduced 'biosynthetic-Pfam' pHMM feature library by merging 250 antiSMASH 5.0 model HMMs with Pfam domains filtered for biosynthetic protein families via ECDomainMiner and manual curation, then packaging the result into the BiG-SLiCE HMM database release.

## When to use

Use when reconstructing BiG-SLiCE feature set 1: a curated biosynthetic pHMM library (full-length biosynthetic-Pfam plus fragmented sub-pfam models) that replaces full Pfam for BGC feature computation.

## When NOT to use

- Running BiG-SLiCE 2.0, which already ships an updated HMM database built on PFAM 35.0 — rebuilding from Pfam v32 would produce a library inconsistent with v2's cosine-like clustering and antiSMASH v7.0.0 class definitions
- The goal is de novo BGC detection from genome sequences — that is antiSMASH's job; this skill only assembles the feature-model library used after BGC prediction
- Input is already a per-cluster domain/feature table — no HMM model merging is needed at that stage

## Inputs

- Pfam v32 HMM library (full set of pHMM domain models)
- antiSMASH 5.0 pHMM models (250 model HMMs)
- ECDomainMiner enzymatic-function domain filtering results
- Manual curation of each candidate domain's full description

## Outputs

- Curated biosynthetic-Pfam model set (catalogued in Supplementary Table S1)
- Fragmented sub-pfam model set built alongside the full-length models
- Packaged BiG-SLiCE HMM database release with verifiable integrity identifiers (biosynthetic-pfam md5 37495cac452bf1dd8aff2c4ad92065fe; sub-pfam md5 2e6b41d06f3c318c61dffb022798091e)

## How to apply

Start from two reference HMM libraries: the Pfam v32 HMM library and the antiSMASH 5.0 pHMM models, which serve as raw material for the biosynthetic model set. Apply ECDomainMiner to filter Pfam v32 domains down to those encoding biosynthetically related protein families (it allows filtering for domains related to enzymatic functions), then manually curate the candidate list based on each domain's full description to remove false positives. Merge the 250 antiSMASH pHMM models with the curated Pfam-derived set to assemble the biosynthetic-Pfam model set catalogued in Supplementary Table S1. In parallel, build the fragmented sub-pfam model set alongside the full-length biosynthetic-pfam models and package both into a BiG-SLiCE HMM database release. Validate the rebuilt database by running the bundled download_bigslice_hmmdb utility and checking that bigslice --version reports the expected integrity identifiers: biosynthetic-pfam md5 37495cac452bf1dd8aff2c4ad92065fe and sub-pfam md5 2e6b41d06f3c318c61dffb022798091e.

## Related tools

- **BiG-SLiCE** (Consumes the merged biosynthetic-Pfam/sub-pfam HMM database and ships the download_bigslice_hmmdb utility plus bigslice --version integrity identifiers used to validate the rebuild) — https://github.com/medema-group/bigslice
- **antiSMASH 5.0** (Source of the 250 pHMM models merged with the curated Pfam-derived set to form the biosynthetic-Pfam library)
- **pyhmmer** (Python-native HMM scanning engine used by BiG-SLiCE 2.0 to apply the merged pHMM database, replacing the standalone HMMER suite of v1.x) — https://github.com/althonos/pyhmmer

## Examples

```
download_bigslice_hmmdb && bigslice --version
```

## Evaluation signals

- bigslice --version prints biosynthetic-pfam md5 equal to 37495cac452bf1dd8aff2c4ad92065fe
- bigslice --version prints sub-pfam md5 equal to 2e6b41d06f3c318c61dffb022798091e
- The antiSMASH contribution to the merged library is exactly the 250 pHMM models, and the total model list matches Supplementary Table S1
- Every Pfam-derived model in the library is traceable to an ECDomainMiner-filtered candidate that survived manual curation (no false positives retained)
- A sub-pfam (fragmented) model set is emitted alongside the full-length biosynthetic-pfam models, both packaged into the single HMM database release

## Limitations

- The published model library was built from Pfam v32; BiG-SLiCE 2.0 updated its pHMM databases to PFAM 35.0, so the v1-era library is not the one shipped with v2
- ECDomainMiner filtering alone is insufficient — manual curation of each domain's full description is required to remove false positives, which introduces curation-dependent variability
- The archived v1.0.0 software release including the exact pHMM models used in the study is distributed via Zenodo, so exact reproducibility depends on fetching that archived release rather than rebuilding

## Evidence

- [supplementary] Merge the 250 antiSMASH pHMM models with the curated Pfam-derived set to assemble the biosynthetic-Pfam model set catalogued in Supplementary Table S1.: [evidence span withheld — anchor status: not_found]
- [methods] ECDomainMiner filtering plus manual curation: [evidence span withheld — anchor status: not_found]
- [readme] BiG-SLiCE 2.0 updated HMM database: "pHMM databases have been updated to __PFAM 35.0__"
- [readme] pyhmmer replaces standalone HMMER: "BiG-SLiCE 2.0 uses PyHMMER — a Python-native HMM scanning library — replacing the standalone HMMER suite required in BiG-SLiCE 1.x."
- [supplementary] Archived v1.0.0 release with the pHMM models: "An archived v1.0.0 release of the BiG-SLiCE software including the pHMM models used for this study can be downloaded from Zenodo"
