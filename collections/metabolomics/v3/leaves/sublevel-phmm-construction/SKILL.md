---
name: sublevel-phmm-construction
description: Use when building a BiG-SLiCE feature space that needs finer resolution
  than the 293 core biosynthetic pHMMs, i.e. when sub-clade-level domain fingerprinting
  of BGCs is required.
when_to_use_negative:
- Generic chemical-class-level BGC classification suffices — the 293 biosynthetic-Pfam
  models already capture the domain content needed.
- Your goal is BGC detection/prediction rather than domain fingerprinting — use antiSMASH,
  DeepBGC, or ClusterFinder instead of building sublevel HMMs.
- A core domain resolves into fewer than 10 or more than 100 clades — it is excluded
  from the sub-Pfam set by design.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: BiG-SLiCE
  role: Provides the HMM database and packaging of the 3,889 sub-Pfam models into
    subpfam.hmm; sub-Pfam models form its feature space for Euclidean BGC projection.
  repo: https://github.com/medema-group/bigslice
- name: PyHMMER
  role: Used for HMM scanning/alignment of the 9,451,490 RP15 protein sequences against
    the 293 core biosynthetic domain pHMMs; replaces standalone HMMER in BiG-SLiCE
    2.0.
  repo: https://github.com/althonos/pyhmmer
- name: HMMER
  role: Underlying pHMM search/build engine used for aligning representative sequences
    and building sublevel HMMs from clade alignments.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:90322c4c4d573d637ee98018a6abdf2b5600848b13ebb9a4de605d6186576d50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - HMMER
  - PyHMMER
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sublevel-phmm-construction

## Summary

Constructs granular sub-Pfam (signature domain) pHMMs by hierarchically clustering aligned representative protein sequences from core biosynthetic domains into clades and building a sublevel HMM per clade, capturing sequence-level diversity that generic biosynthetic-Pfam models miss.

## When to use

Use when building a BiG-SLiCE feature space that needs finer resolution than the 293 core biosynthetic pHMMs, i.e. when sub-clade-level domain fingerprinting of BGCs is required.

## When NOT to use

- Generic chemical-class-level BGC classification suffices — the 293 biosynthetic-Pfam models already capture the domain content needed.
- Your goal is BGC detection/prediction rather than domain fingerprinting — use antiSMASH, DeepBGC, or ClusterFinder instead of building sublevel HMMs.
- A core domain resolves into fewer than 10 or more than 100 clades — it is excluded from the sub-Pfam set by design.

## Inputs

- 9,451,490 representative RP15 (Release 2020_01) protein sequences
- 293 core biosynthetic domain pHMMs (Biosynthetic-pfam, md5 37495cac452bf1dd8aff2c4ad92065fe) from bigslice-models-2022-11-30
- HMMER/PyHMMER environment

## Outputs

- 3,889 sub-Pfam models (Supplementary Table S2), 10–100 clades per core domain
- subpfam.hmm packaged HMM library (Sub-pfam md5 2e6b41d06f3c318c61dffb022798091e)

## How to apply

Retrieve the BiG-SLiCE HMM database (bigslice-models-2022-11-30) via download_bigslice_hmmdb and verify the Biosynthetic-pfam md5 (37495cac452bf1dd8aff2c4ad92065fe). Align the 9,451,490 representative RP15 (Release 2020_01) protein sequences against the 293 preselected core biosynthetic domain pHMMs using HMMER/PyHMMER. For each core domain, hierarchically cluster the aligned sequences into clades and retain only domains that resolve into 10–100 clades. Build a sublevel pHMM from the alignment of each accepted clade to yield 3,889 sub-Pfam models, package them as subpfam.hmm, and verify against the Sub-pfam md5 2e6b41d06f3c318c61dffb022798091e. The 10–100 clade window balances granularity against model proliferation per domain.

## Related tools

- **BiG-SLiCE** (Provides the HMM database and packaging of the 3,889 sub-Pfam models into subpfam.hmm; sub-Pfam models form its feature space for Euclidean BGC projection.) — https://github.com/medema-group/bigslice
- **PyHMMER** (Used for HMM scanning/alignment of the 9,451,490 RP15 protein sequences against the 293 core biosynthetic domain pHMMs; replaces standalone HMMER in BiG-SLiCE 2.0.) — https://github.com/althonos/pyhmmer
- **HMMER** (Underlying pHMM search/build engine used for aligning representative sequences and building sublevel HMMs from clade alignments.)

## Examples

```
download_bigslice_hmmdb  # fetch bigslice-models-2022-11-30; verify Biosynthetic-pfam md5 37495cac452bf1dd8aff2c4ad92065fe, build clade HMMs into subpfam.hmm (md5 2e6b41d06f3c318c61dffb022798091e)
```

## Evaluation signals

- Rebuilt sub-Pfam library contains exactly 3,889 sub-Pfam models matching Supplementary Table S2.
- subpfam.hmm md5 checksum equals 2e6b41d06f3c318c61dffb022798091e.
- Biosynthetic-pfam input HMM database md5 equals 37495cac452bf1dd8aff2c4ad92065fe.
- Every retained core domain resolves into 10–100 clades; domains outside this range are excluded.
- Each sublevel pHMM is built from the multiple alignment of a single accepted clade's sequences.

## Limitations

- Only domains resolving into 10–100 clades contribute sub-Pfam models, so core domains outside this window are not refined.
- The model set is tied to a specific sequence release (RP15, Release 2020_01, 9,451,490 proteins) and HMM database version (bigslice-models-2022-11-30); rebuilding with other data may not reproduce the checksums.
- BiG-SLiCE 2.0 updated the pHMM databases to PFAM 35.0 and switched to PyHMMER, so v1-era model sets and checksums are version-specific.

## Evidence

- [methods] Sub-Pfam model construction from clades: [evidence span withheld — anchor status: not_found]
- [methods] 3,889 models from 10–100 clades per domain: [evidence span withheld — anchor status: not_found]
- [readme] PyHMMER used for scanning: "Switching from HMMER to [pyHMMER](https://github.com/althonos/pyhmmer) (__speed-ups__, can now be fully installed via __pip__)"
- [readme] pHMM databases updated in v2: "pHMM databases have been updated to __PFAM 35.0__"
- [abstract] Tool repository: "BiG-SLiCE is available via https://github.com/medema-group/bigslice."
