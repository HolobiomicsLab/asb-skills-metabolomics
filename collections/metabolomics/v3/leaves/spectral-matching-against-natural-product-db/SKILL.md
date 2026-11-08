---
name: spectral-matching-against-natural-product-db
description: Use when MZmine-exported feature MS2 spectra (with parent m/z and fragment
  peaks) must be structurally annotated against in silico natural products, and candidates
  should be reweighted using the sample's organism and the molecular network cluster.
when_to_use_negative:
- When the goal is de novo structure elucidation without a database — use the complementary
  Sirius/CSI:FingerID annotation path instead (results from the two approaches can
  be combined to strengthen confidence).
- When features are being detected or filtered from raw data — this skill operates
  on MZmine-exported feature MS2 spectra and does not perform feature detection.
- When matching against experimental spectral libraries rather than the in silico
  ISDB-LOTUS database, since scoring and reweighting are designed around in silico
  candidates.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python package (v0.20.0) used for spectral matching of feature MS2 spectra
    against ISDB-LOTUS and for modified cosine molecular networking
  repo: https://github.com/enpkg/enpkg_workflow
- name: MZmine
  role: Upstream feature detection and export producing the feature MS2 spectra consumed
    by the matching step
  repo: https://github.com/enpkg/enpkg_full
- name: ISDB-LOTUS
  role: In silico database of natural products used as the spectral matching reference
- name: Open Tree of Life
  role: Taxonomy reference used to resolve the sample's species and compute taxonomic
    distance for candidate reranking
- name: ENPKG
  role: Hosts the scripts implementing this matching/reranking step (03_enpkg_mn_isdb_isdb_taxo)
  repo: https://github.com/enpkg/enpkg_full
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:7c3dc8e97fa2e973c592bdc3cd86dea7d571686664205329b47a8458925d5ec2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ENPKG
  - ISDB-LOTUS
  - MZmine
  - Open Tree of Life
  - matchms
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-matching-against-natural-product-db

## Summary

Annotate LC-MS/MS feature spectra by spectral matching against the ISDB-LOTUS in silico natural products database with matchms, then rerank candidates by taxonomic distance and chemical consistency to select the best candidate per feature.

## When to use

Use when MZmine-exported feature MS2 spectra (with parent m/z and fragment peaks) must be structurally annotated against in silico natural products, and candidates should be reweighted using the sample's organism and the molecular network cluster.

## When NOT to use

- When the goal is de novo structure elucidation without a database — use the complementary Sirius/CSI:FingerID annotation path instead (results from the two approaches can be combined to strengthen confidence).
- When features are being detected or filtered from raw data — this skill operates on MZmine-exported feature MS2 spectra and does not perform feature detection.
- When matching against experimental spectral libraries rather than the in silico ISDB-LOTUS database, since scoring and reweighting are designed around in silico candidates.

## Inputs

- MZmine-exported feature MS2 spectra per sample (sample-wise directory architecture, .mgf)
- Feature quantification table from MZmine feature detection
- Sample metadata file containing the sample's originating taxon
- ISDB-LOTUS in silico natural products spectral database
- Taxonomically resolved sample organism (Open Tree of Life ottID / Wikidata ID)

## Outputs

- Reranked candidate-annotation table with one best-ranked candidate per feature
- Molecular network (modified cosine score edges) supporting chemical-consistency reweighting

## How to apply

Load the MZmine-exported feature MS2 spectra for each sample from the sample-wise directory architecture and run spectral matching in Python with matchms (v0.20.0) against ISDB-LOTUS, using an m/z tolerance of 0.01 for both parent mass and fragments, a minimal cosine score of 0.2, and 6 minimal matching fragments. Also consider adducts of compounds reported for the sample's species for MS1-only annotation at 10 ppm tolerance. Rerank matched candidates by the taxonomic distance between the candidate's producing organism and the sample's organism, with taxa resolved via Open Tree of Life (ottID) and Wikidata IDs. Reweight candidates by chemical consistency of the molecular network cluster (modified cosine score cutoff 0.7, max_links 10, top_n 15) following the Network Annotation Propagation principle, then select the best-ranked candidate per feature and write the reranked candidate-annotation table. The rationale is that taxonomic and chemical-consistency reweighting prioritizes plausible in silico hits in a database that is otherwise large and noisy.

## Related tools

- **matchms** (Python package (v0.20.0) used for spectral matching of feature MS2 spectra against ISDB-LOTUS and for modified cosine molecular networking) — https://github.com/enpkg/enpkg_workflow
- **MZmine** (Upstream feature detection and export producing the feature MS2 spectra consumed by the matching step) — https://github.com/enpkg/enpkg_full
- **ISDB-LOTUS** (In silico database of natural products used as the spectral matching reference)
- **Open Tree of Life** (Taxonomy reference used to resolve the sample's species and compute taxonomic distance for candidate reranking)
- **ENPKG** (Hosts the scripts implementing this matching/reranking step (03_enpkg_mn_isdb_isdb_taxo)) — https://github.com/enpkg/enpkg_full

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git && sh workflow/00_workflow_all.sh
```

## Evaluation signals

- The reranked candidate-annotation table is produced with exactly one annotated candidate per feature.
- Scripts used (03_enpkg_mn_isdb_isdb_taxo) run to completion without error.
- Matched candidates satisfy the configured thresholds: cosine score >= 0.2 with >= 6 matching fragments at m/z tolerance 0.01.
- Candidate ordering changes after taxonomic reranking relative to raw cosine score ordering, and reweighting reflects the molecular network cluster (modified cosine cutoff 0.7, max_links 10, top_n 15).
- MS1-only annotations are restricted to adducts of compounds reported in the sample's species at 10 ppm tolerance.

## Limitations

- The article reports no statistical metrics (e.g., FDR, p-values, effect sizes) validating the anti-T. cruzi / anti-L. donovani compound annotations produced by this pipeline.
- MS1-only annotation is limited to potential adducts of compounds reported in the sample's species, so unknown chemistry in that taxon is missed.
- Annotation confidence depends on ISDB-LOTUS coverage; combining ISDB and SIRIUS structural annotations is recommended to strengthen confidence in spectral annotation.

## Evidence

- [readme] annotates features using two different methods (spectral matching to in silico DB coupled to taxonomical reweighting and Sirius/CSI:FingerID): [evidence span withheld — anchor status: external_doc]
- [methods] The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo.: "The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo."
- [results] an in silico database of natural products (ISDB-LOTUS): "an in silico database of natural products (ISDB-LOTUS)"
- [readme] the workflow automatically resolves the species taxonomy against Open Tree of Life (ottID): [evidence span withheld — anchor status: external_doc]
- [results] Comparing structural annotations results from two different approaches...allows the user to strengthen confidence in spectral annotation: "Comparing structural annotations results from two different approaches...allows the user to strengthen confidence
in spectral annotation"
