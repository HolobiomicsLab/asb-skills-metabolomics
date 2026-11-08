---
name: ecdomainminer-biosynthetic-domain-mining
description: Use when building or rebuilding a curated pHMM feature library restricted
  to biosynthetic protein families (e.g., for BiG-SLiCE feature set 1) from a full
  Pfam release plus antiSMASH domain models.
when_to_use_negative:
- The goal is broad domain annotation of all protein families rather than a biosynthesis-restricted
  feature space — the curated list deliberately discards non-biosynthetic Pfam domains.
- A newer BiG-SLiCE release is already in use whose pHMM databases target PFAM 35.0
  and antiSMASH v7.0.0 class definitions — do not re-assemble the older v32/antiSMASH
  5.0 set unless reproducing the original study.
- Downstream analysis requires pairwise BGC similarity networks (e.g., BiG-SCAPE-style
  comparison) rather than Euclidean-space feature projection.
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ECDomainMiner
  role: Filters Pfam domains down to those related to enzymatic (biosynthetic) functions
    before manual curation
- name: BiG-SLiCE
  role: Consumes the assembled biosynthetic-pfam and sub-pfam model sets as its HMM
    database; validated via bigslice --version md5 identifiers
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH 5.0
  role: Source of the ~250 biosynthetic pHMM models merged into the biosynthetic-Pfam
    set
- name: pyhmmer
  role: Python HMMER3 scanning library used by BiG-SLiCE to score input proteins against
    the assembled pHMM libraries
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:19227dd275878cd99c38490437ecc0207a58a00196e23a750ed2ff71ab50c36a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - ECDomainMiner
  - antiSMASH 5.0
  - pyhmmer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ecdomainminer-biosynthetic-domain-mining

## Summary

Constructs a reduced 'biosynthetic-Pfam' pHMM library (feature set 1) by filtering all Pfam domains down to biosynthetically related protein families via ECDomainMiner plus manual curation, then merging antiSMASH pHMM models and packaging the result into the BiG-SLiCE HMM database.

## When to use

Use when building or rebuilding a curated pHMM feature library restricted to biosynthetic protein families (e.g., for BiG-SLiCE feature set 1) from a full Pfam release plus antiSMASH domain models.

## When NOT to use

- The goal is broad domain annotation of all protein families rather than a biosynthesis-restricted feature space — the curated list deliberately discards non-biosynthetic Pfam domains.
- A newer BiG-SLiCE release is already in use whose pHMM databases target PFAM 35.0 and antiSMASH v7.0.0 class definitions — do not re-assemble the older v32/antiSMASH 5.0 set unless reproducing the original study.
- Downstream analysis requires pairwise BGC similarity networks (e.g., BiG-SCAPE-style comparison) rather than Euclidean-space feature projection.

## Inputs

- Pfam v32 HMM library (full domain set)
- antiSMASH 5.0 pHMM models (~250 models)
- Full textual descriptions of candidate Pfam domains (for manual curation)

## Outputs

- Curated biosynthetic-Pfam pHMM model set (catalogued in Supplementary Table S1)
- Fragmented sub-pfam model set
- BiG-SLiCE HMM database release with biosynthetic-pfam md5 37495cac452bf1dd8aff2c4ad92065fe and sub-pfam md5 2e6b41d06f3c318c61dffb022798091e

## How to apply

Retrieve the reference HMM libraries — the Pfam v32 HMM library and the antiSMASH 5.0 pHMM models — as raw material. Apply ECDomainMiner to the Pfam v32 domains to keep those encoding biosynthetic protein families (ECDomainMiner filters for domains related to enzymatic functions), then manually curate the candidate list based on each domain's full description to remove false positives. Merge the ~250 antiSMASH pHMM models with the curated Pfam-derived set to assemble the biosynthetic-Pfam model set catalogued in Supplementary Table S1. In parallel, build the fragmented sub-pfam model set alongside the full-length biosynthetic-pfam models and package both into the BiG-SLiCE HMM database release. Fetch the bundled release with the download_bigslice_hmmdb utility and verify bigslice --version reports the expected integrity identifiers.

## Related tools

- **ECDomainMiner** (Filters Pfam domains down to those related to enzymatic (biosynthetic) functions before manual curation)
- **BiG-SLiCE** (Consumes the assembled biosynthetic-pfam and sub-pfam model sets as its HMM database; validated via bigslice --version md5 identifiers) — https://github.com/medema-group/bigslice
- **antiSMASH 5.0** (Source of the ~250 biosynthetic pHMM models merged into the biosynthetic-Pfam set)
- **pyhmmer** (Python HMMER3 scanning library used by BiG-SLiCE to score input proteins against the assembled pHMM libraries) — https://github.com/althonos/pyhmmer

## Examples

```
mamba env create -n bigslice -c conda-forge -c bioconda bigslice && download_bigslice_hmmdb && bigslice --version
```

## Evaluation signals

- bigslice --version must report biosynthetic-pfam md5 = 37495cac452bf1dd8aff2c4ad92065fe
- bigslice --version must report sub-pfam md5 = 2e6b41d06f3c318c61dffb022798091e
- The merged set should contain the antiSMASH 5.0-derived models (~250) on top of the ECDomainMiner+manual-curation Pfam set
- The final catalog must be documentable as a complete list (as done in Supplementary Table S1) with each domain's full description retained for curation traceability

## Limitations

- Manual curation based on each domain's full description is a subjective step; false positives from ECDomainMiner must be removed by expert judgment, so the list is not fully reproducible from the tool alone.
- The described construction targets Pfam v32 and antiSMASH 5.0; BiG-SLiCE 2.0 ships pHMM databases updated to PFAM 35.0 with antiSMASH v7.0.0 class definitions, so the v32-era set will not match newer releases.
- BiG-SLiCE 1.x required a standalone HMMER installation, whereas v2.0 switched to PyHMMER — environment assumptions differ across versions when validating the database.

## Evidence

- [other] The biosynthetic-Pfam list was constructed by filtering all Pfam domains for biosynthetically related protein families using the combination of ECDomainMiner: [evidence span withheld — anchor status: not_found]
- [other] Merge the 250 antiSMASH pHMM models with the curated Pfam-derived set: [evidence span withheld — anchor status: not_found]
- [other] Validation: the rebuilt database is accepted when its biosynthetic-pfam md5 (37495cac452bf1dd8aff2c4ad92065fe) and sub-pfam md5: [evidence span withheld — anchor status: not_found]
- [supplementary] An archived v1.0.0 release of the BiG-SLiCE software including the pHMM models used for this study can be downloaded from Zenodo: "An archived v1.0.0 release of the BiG-SLiCE software including the pHMM models used for this study can be downloaded from Zenodo"
- [readme] pHMM databases have been updated to __PFAM 35.0__: "pHMM databases have been updated to __PFAM 35.0__"
- [readme] Switching from HMMER to [pyHMMER](https://github.com/althonos/pyhmmer) (__speed-ups__, can now be fully installed via __pip__): "Switching from HMMER to [pyHMMER](https://github.com/althonos/pyhmmer) (__speed-ups__, can now be fully installed via __pip__)"
