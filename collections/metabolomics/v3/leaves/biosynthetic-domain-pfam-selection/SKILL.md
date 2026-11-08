---
name: biosynthetic-domain-pfam-selection
description: Use when constructing or curating the domain feature space for BGC feature
  fingerprinting — e.g., selecting core biosynthetic Pfam pHMMs for BiG-SLiCE runs,
  or rebuilding/validating the sub-Pfam (sublevel) signature model library.
when_to_use_negative:
- Input is already a per-BGC domain feature vector or GCF feature table — domain selection
  has already been performed upstream
- The goal is pairwise BGC network comparison (BiG-SCAPE-style O(n²) distance analysis)
  rather than building a Euclidean-space feature set
- BGCs were called by tools with their own domain vocabularies (e.g., DeepBGC's word2vec
  Pfam-domain embedding, ClusterFinder's Pfam probability scheme) whose domain calls
  do not map cleanly onto the 293 preselected biosynthetic pHMMs
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: Provides the curated Biosynthetic-pfam (293 domains) and Sub-pfam (3,889 models)
    pHMM databases and the download_bigslice_hmmdb command used to select/install
    the feature-space HMMs
  repo: https://github.com/medema-group/bigslice
- name: PyHMMER
  role: Python-native HMMER3 bindings used in BiG-SLiCE 2.0 to scan representative
    protein sequences against the core biosynthetic domain pHMMs and build sublevel
    models
  repo: https://github.com/althonos/pyhmmer
- name: HMMER
  role: Underlying profile HMM search/build engine (hmmscan/hmmbuild) required by
    BiG-SLiCE 1.x for aligning sequences against the selected domain pHMMs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:4fac8da2a8cf6325849aa1591ad403ffce9c4644b70fe2e62042b095f8ca8863
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - HMMER
  - PyHMMER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# biosynthetic-domain-pfam-selection

## Summary

Selects the curated set of core biosynthetic domain pHMMs (BiG-SLiCE's 293-domain Biosynthetic-pfam set) that defines the domain-level feature space for BGC fingerprinting, and supports rebuilding the sub-Pfam signature models (3,889 sublevel pHMMs) derived from it. This feature-space choice is what lets BiG-SLiCE capture sequence-level diversity that generic chemical-class Pfam models miss.

## When to use

Use when constructing or curating the domain feature space for BGC feature fingerprinting — e.g., selecting core biosynthetic Pfam pHMMs for BiG-SLiCE runs, or rebuilding/validating the sub-Pfam (sublevel) signature model library.

## When NOT to use

- Input is already a per-BGC domain feature vector or GCF feature table — domain selection has already been performed upstream
- The goal is pairwise BGC network comparison (BiG-SCAPE-style O(n²) distance analysis) rather than building a Euclidean-space feature set
- BGCs were called by tools with their own domain vocabularies (e.g., DeepBGC's word2vec Pfam-domain embedding, ClusterFinder's Pfam probability scheme) whose domain calls do not map cleanly onto the 293 preselected biosynthetic pHMMs

## Inputs

- BiG-SLiCE HMM database (bigslice-models-2022-11-30) containing the 293 core biosynthetic domain pHMMs
- Representative protein sequences from a BGC release (e.g., 9,451,490 RP15/Release 2020_01 representatives)
- Pfam pHMM release used by the installed BiG-SLiCE version (PFAM 35.0 in BiG-SLiCE 2.0)

## Outputs

- Biosynthetic-pfam feature space: 293 core biosynthetic domain pHMMs with verified md5 37495cac452bf1dd8aff2c4ad92065fe
- subpfam.hmm: library of 3,889 sublevel (sub-Pfam) pHMMs, 10–100 clades per core domain, md5 2e6b41d06f3c318c61dffb022798091e
- Per-clade alignments of representative domain sequences used to build each sublevel pHMM

## How to apply

Retrieve the curated biosynthetic pHMM database (bigslice-models-2022-11-30) with the download_bigslice_hmmdb command and verify the Biosynthetic-pfam md5 (37495cac452bf1dd8aff2c4ad92065fe) before use. Align the representative protein sequences (e.g., the 9,451,490 RP15/Release 2020_01 representatives) against the 293 preselected core biosynthetic domain pHMMs using HMMER/PyHMMER rather than ad-hoc Pfam sets, because the curated set concentrates feature dimensions on domains diagnostic of secondary metabolism. To extend the space to sub-Pfam granularity, hierarchically cluster the aligned sequences within each core domain and retain only domains that resolve into 10–100 clades; build a sublevel pHMM (hmmbuild) from each accepted clade's alignment, yielding 3,889 sub-Pfam models (Supplementary Table S2). Package the models as subpfam.hmm and check integrity against the Sub-pfam md5 (2e6b41d06f3c318c61dffb022798091e). Note version sensitivity: BiG-SLiCE 2.0 updated its pHMM databases to PFAM 35.0 and switched from standalone HMMER to PyHMMER, so rebuilds against a different release will not byte-match the published checksums.

## Related tools

- **BiG-SLiCE** (Provides the curated Biosynthetic-pfam (293 domains) and Sub-pfam (3,889 models) pHMM databases and the download_bigslice_hmmdb command used to select/install the feature-space HMMs) — https://github.com/medema-group/bigslice
- **PyHMMER** (Python-native HMMER3 bindings used in BiG-SLiCE 2.0 to scan representative protein sequences against the core biosynthetic domain pHMMs and build sublevel models) — https://github.com/althonos/pyhmmer
- **HMMER** (Underlying profile HMM search/build engine (hmmscan/hmmbuild) required by BiG-SLiCE 1.x for aligning sequences against the selected domain pHMMs)

## Examples

```
mamba env create -n bigslice -c conda-forge -c bioconda bigslice && mamba activate bigslice && download_bigslice_hmmdb
```

## Evaluation signals

- Biosynthetic-pfam md5 checksum equals 37495cac452bf1dd8aff2c4ad92065fe for the bigslice-models-2022-11-30 release
- Rebuilt subpfam.hmm md5 equals 2e6b41d06f3c318c61dffb022798091e
- Rebuilt sub-Pfam library contains exactly 3,889 models whose identities match Supplementary Table S2
- Every retained core domain resolves into 10–100 hierarchical clades; domains outside this range are excluded
- Feature-space HMM version matches the BiG-SLiCE release in use (BiG-SLiCE 2.0 expects PFAM 35.0 models)

## Limitations

- Sub-Pfam models were derived from RP15 (Release 2020_01) representatives; rebuilding against a different protein release or Pfam version will not reproduce the published checksums or the 3,889-model count
- Only core biosynthetic domains resolving into 10–100 clades yield sublevel models — highly conserved or ultra-diverse domains are not subdivided
- BiG-SLiCE 1.x requires a standalone HMMER installation for the pHMM scans, whereas 2.0 uses PyHMMER; mixing versions changes results and checksums

## Evidence

- [methods] Sub-Pfam feature set construction from 293 core domains: [evidence span withheld — anchor status: not_found]
- [methods] 10–100 clades filter for sublevel model building: [evidence span withheld — anchor status: not_found]
- [readme] BiG-SLiCE 2.0 updated pHMM databases to PFAM 35.0: "pHMM databases have been updated to __PFAM 35.0__"
- [supplementary] Archived pHMM models downloadable from Zenodo: "An archived v1.0.0 release of the BiG-SLiCE software including the pHMM models used for this study can be downloaded from Zenodo"
- [readme] PyHMMER replaces standalone HMMER in BiG-SLiCE 2.0: [evidence span withheld — anchor status: not_found]
