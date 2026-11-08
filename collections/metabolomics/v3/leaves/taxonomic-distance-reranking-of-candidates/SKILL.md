---
name: taxonomic-distance-reranking-of-candidates
description: Use when annotating LC-MS/MS features of a biological extract against
  an in silico structure database and you want to reweight spectral matches by how
  closely the candidate's source organism relates taxonomically to the sample's biosource.
when_to_use_negative:
- Samples lack reliable taxonomic metadata for the originating organism (distance
  cannot be computed).
- Analyzing non-biological or mixed/purchased samples whose chemotype is not expected
  to reflect the declared biosource taxonomy.
- You only need raw spectral similarity without organism context, e.g. matching against
  experimental spectral libraries rather than in silico structure–organism databases.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Python spectral matching (cosine, parent mass tolerance 0.01, min cosine 0.2,
    6 matched fragments) and modified-cosine molecular networking used before and
    alongside taxonomic reweighting
  repo: https://github.com/enpkg/enpkg_workflow
- name: ISDB-LOTUS
  role: In silico natural products candidate database whose structure–organism pairs
    supply candidate taxa for distance computation
- name: Open Tree of Life
  role: Provides the taxonomy (ottIDs) against which sample and candidate organisms
    are resolved to compute taxonomic distance
- name: MZmine
  role: Upstream feature detection and MS2 spectrum export feeding the matching step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:868fef009ce02528206f8b789e39a2d95fc8da7d72cf80d47d601b5677664b63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ISDB-LOTUS
  - MZmine
  - Open Tree of Life
  - matchms
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# taxonomic-distance-reranking-of-candidates

## Summary

Reranks spectral-matching candidate structures from ISDB-LOTUS according to the taxonomic distance between each candidate's reported producing organism and the sample's organism, boosting annotations that are biogeographically plausible for the analyzed extract.

## When to use

Use when annotating LC-MS/MS features of a biological extract against an in silico structure database and you want to reweight spectral matches by how closely the candidate's source organism relates taxonomically to the sample's biosource.

## When NOT to use

- Samples lack reliable taxonomic metadata for the originating organism (distance cannot be computed).
- Analyzing non-biological or mixed/purchased samples whose chemotype is not expected to reflect the declared biosource taxonomy.
- You only need raw spectral similarity without organism context, e.g. matching against experimental spectral libraries rather than in silico structure–organism databases.

## Inputs

- MZmine-exported feature MS2 spectra per sample (sample-wise directory architecture)
- Sample metadata file with the originating taxon
- ISDB-LOTUS in silico natural products database with structure–organism pairs
- Open Tree of Life ottIDs / Wikidata taxon IDs for the sample's species

## Outputs

- Reranked candidate-annotation table with one best-ranked candidate per feature
- Taxonomically reweighted spectral matches (spectral similarity + taxonomic distance + chemical-consistency scores)

## How to apply

After MZmine feature detection, load the sample's MS2 spectra from the sample-wise directory and run matchms (v0.20.0) spectral matching against ISDB-LOTUS with parent-mass and fragment m/z tolerance of 0.01, minimal cosine score of 0.2, and 6 minimal matching fragments; also consider adducts of compounds reported for the sample's species for MS1-only annotation at 10 ppm. Resolve both the sample's taxon and each candidate's producing organism to Open Tree of Life/Wikidata IDs (the workflow's 02_enpkg_taxo_enhancer step), compute the taxonomic distance between them, and use it to rerank the candidate list. Then reweight candidates by chemical consistency of the molecular network cluster (matchms molecular networking with modified cosine score cutoff 0.7, max_links 10, top_n 15) following the Network Annotation Propagation principle. Select the best-ranked candidate per feature and write the reranked candidate-annotation table (scripts 03_enpkg_mn_isdb_isdb_taxo).

## Related tools

- **matchms** (Python spectral matching (cosine, parent mass tolerance 0.01, min cosine 0.2, 6 matched fragments) and modified-cosine molecular networking used before and alongside taxonomic reweighting) — https://github.com/enpkg/enpkg_workflow
- **ISDB-LOTUS** (In silico natural products candidate database whose structure–organism pairs supply candidate taxa for distance computation)
- **Open Tree of Life** (Provides the taxonomy (ottIDs) against which sample and candidate organisms are resolved to compute taxonomic distance)
- **MZmine** (Upstream feature detection and MS2 spectrum export feeding the matching step)

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git && bash src/install_sirius.sh /opt/sirius && sh workflow/00_workflow_all.sh
```

## Evaluation signals

- The reranked candidate-annotation table is produced with one annotated candidate per feature.
- Scripts in 03_enpkg_mn_isdb_isdb_taxo run to completion without error.
- Candidates reported from the sample's own species (taxonomic distance 0) rank above candidates from distant taxa at comparable cosine scores.
- All matched features carry both a spectral cosine score ≥ 0.2 with ≥ 6 matching fragments and a resolved taxonomic distance value.
- MS1-only adduct annotations respect the 10 ppm parent-mass tolerance.

## Limitations

- Taxonomic reweighting assumes biosource taxonomy predicts chemistry; it can misrank when compounds are reported from unrelated organisms (convergent chemistries, database gaps in LOTUS coverage).
- The workflow's discussion notes no reported rate of false positives/false negatives from merging experimental data with public knowledge resources, so reranked annotations are not error-bounded.
- Species resolution depends on Open Tree of Life/Wikidata ID availability; unresolved taxa cannot contribute to the distance signal.

## Evidence

- [results] matching against the ISDB-LOTUS in silico natural products database, with candidate structures reranked via taxonomical and chemical consistency reweighting: [evidence span withheld — anchor status: not_found]
- [results] matchms matching parameters: [evidence span withheld — anchor status: not_found]
- [methods] taxonomic resolution via Open Tree of Life: [evidence span withheld — anchor status: not_found]
- [methods] scripts location: "The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo."
- [results] chemical consistency / Network Annotation Propagation: [evidence span withheld — anchor status: not_found]
