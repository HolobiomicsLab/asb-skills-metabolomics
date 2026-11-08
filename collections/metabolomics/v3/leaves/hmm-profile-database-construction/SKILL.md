---
name: hmm-profile-database-construction
description: Use when you need to rebuild or customize the biosynthetic pHMM reference
  library (feature set 1) that BiG-SLiCE uses to project BGCs into Euclidean feature
  space, e.g. after a Pfam release update.
when_to_use_negative:
- You only need to run BiG-SLiCE analyses with the standard database — use the bundled
  download_bigslice_hmmdb utility instead of rebuilding models
- You are building GCF models from already-annotated BGCs (that is BiG-SLiCE's clustering
  phase, not pHMM library construction)
- Your feature space is not based on Pfam domains (e.g. ClusterFinder probability
  or DeepBGC deep-learning features from a different model set)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: Consumes the constructed biosynthetic-Pfam and sub-pfam databases to compute
    BGC feature vectors; ships download_bigslice_hmmdb and reports database md5 integrity
    via --version
  repo: https://github.com/medema-group/bigslice
- name: pyHMMER
  role: Python-native HMM3 scanning engine used by BiG-SLiCE 2.0 to apply the constructed
    pHMM database to protein sequences
  repo: https://github.com/althonos/pyhmmer
- name: antiSMASH 5.0
  role: Source of the 250 pHMM models merged with the curated Pfam-derived set
- name: ECDomainMiner
  role: Filtering tool that reduces the full Pfam domain list to biosynthetically
    related, enzymatic-function protein families prior to manual curation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:68be374306d2258ec5aabbe1bc584c442a6577e384454ab37d062de5b882d0c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - ECDomainMiner
  - antiSMASH 5.0
  - pyHMMER
  - pyhmmer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# hmm-profile-database-construction

## Summary

Constructs BiG-SLiCE's reduced 'biosynthetic-Pfam' pHMM feature database by filtering Pfam domains down to biosynthetically related protein families and merging them with antiSMASH pHMM models, then packages the full-length and fragmented sub-pfam model sets for BGC feature vector computation.

## When to use

Use when you need to rebuild or customize the biosynthetic pHMM reference library (feature set 1) that BiG-SLiCE uses to project BGCs into Euclidean feature space, e.g. after a Pfam release update.

## When NOT to use

- You only need to run BiG-SLiCE analyses with the standard database — use the bundled download_bigslice_hmmdb utility instead of rebuilding models
- You are building GCF models from already-annotated BGCs (that is BiG-SLiCE's clustering phase, not pHMM library construction)
- Your feature space is not based on Pfam domains (e.g. ClusterFinder probability or DeepBGC deep-learning features from a different model set)

## Inputs

- Pfam HMM library (v32 for the original study; PFAM 35.0 for BiG-SLiCE 2.0)
- antiSMASH 5.0 pHMM models (250 models)
- Domain descriptions from Pfam for manual curation
- ECDomainMiner filtering output (biosynthetically related domain candidates)

## Outputs

- Biosynthetic-Pfam model set (full-length pHMMs, catalogued in Supplementary Table S1)
- Fragmented sub-pfam model set
- Packaged BiG-SLiCE HMM database release with integrity md5 identifiers

## How to apply

Retrieve the raw HMM libraries — the Pfam release (v32 for the original study; PFAM 35.0 in BiG-SLiCE 2.0) and the antiSMASH 5.0 pHMM models. Apply ECDomainMiner to filter the Pfam domains down to those encoding biosynthetic protein families, then manually curate the candidate list using each domain's full description to remove false positives. Merge the 250 antiSMASH pHMM models with the curated Pfam-derived set to assemble the biosynthetic-Pfam model set catalogued in Supplementary Table S1, and additionally build the fragmented sub-pfam model set alongside the full-length models. Package both model sets into the BiG-SLiCE HMM database release and verify integrity by comparing the md5 checksums reported by 'bigslice --version'. BiG-SLiCE 2.0 performs the actual HMM scanning with pyHMMER, so no standalone HMMER installation is needed.

## Related tools

- **BiG-SLiCE** (Consumes the constructed biosynthetic-Pfam and sub-pfam databases to compute BGC feature vectors; ships download_bigslice_hmmdb and reports database md5 integrity via --version) — https://github.com/medema-group/bigslice
- **pyHMMER** (Python-native HMM3 scanning engine used by BiG-SLiCE 2.0 to apply the constructed pHMM database to protein sequences) — https://github.com/althonos/pyhmmer
- **antiSMASH 5.0** (Source of the 250 pHMM models merged with the curated Pfam-derived set)
- **ECDomainMiner** (Filtering tool that reduces the full Pfam domain list to biosynthetically related, enzymatic-function protein families prior to manual curation)

## Examples

```
mamba env create -n bigslice -c conda-forge -c bioconda bigslice; mamba activate bigslice; download_bigslice_hmmdb; bigslice --version
```

## Evaluation signals

- The rebuilt database is accepted when its biosynthetic-pfam md5 (37495cac452bf1dd8aff2c4ad92065fe) and sub-pfam md5 (2e6b41d06f3c318c61dffb022798091e) match the values printed by bigslice --version
- The assembled model set matches the catalog in Supplementary Table S1 entry-for-entry
- The merged library contains the 250 antiSMASH pHMM models in addition to the curated Pfam-derived domains
- The fragmented sub-pfam model set is built alongside (not instead of) the full-length biosynthetic-Pfam models in the packaged release
- Manual curation removed ECDomainMiner false positives: each retained domain's full description supports a biosynthetic function

## Limitations

- Manual curation step means the list is not fully reproducible from ECDomainMiner alone; false-positive domains must be removed by reading full domain descriptions
- The md5 integrity identifiers are version-specific — updating to a newer Pfam release (e.g. PFAM 35.0 in BiG-SLiCE 2.0) changes the model set and invalidates v1 checksums
- BiG-SLiCE 1.x required a separately installed standalone HMMER suite; only v2.0 (pyHMMER) installs fully via pip/conda

## Evidence

- [supplementary] An archived v1.0.0 release of the BiG-SLiCE software including the pHMM models used for this study can be downloaded from Zenodo: "An archived v1.0.0 release of the BiG-SLiCE software including the pHMM models used for this study can be downloaded from Zenodo"
- [readme] pHMM databases have been updated to PFAM 35.0: "pHMM databases have been updated to __PFAM 35.0__"
- [readme] BiG-SLiCE 2.0 uses PyHMMER replacing the standalone HMMER suite: "BiG-SLiCE 2.0 uses PyHMMER — a Python-native HMM scanning library — replacing the standalone HMMER suite required in BiG-SLiCE 1.x."
- [intro] BiG-SLiCE projects BGCs into Euclidean space: "we introduce BiG-SLiCE (Biosynthetic Genes Super-Linear Clustering Engine), which projects BGCs into Euclidean space"
