---
name: spectral-library-matching-isdb-lotus
description: Use when MZmine-processed LC-HRMS2 DDA features need structural annotation
  against a natural-products library, especially when taxonomic metadata (e.g., Open
  Tree of Life ottIDs) can be exploited to reweight spectral matches.
when_to_use_negative:
- Input features lack associated MS2 scans (the ENPKG feature filtering keeps only
  features with an MS2 scan and RT 0.5–8.0 min), so there is nothing to match
- Samples have no resolvable taxonomic origin, defeating the taxonomic-distance reranking
  that distinguishes this matching from plain spectral search
- The data are not LC-MS/MS DDA metabolomics spectra (e.g., already-aligned feature
  tables without spectra, or MS1-only acquisitions)
edam_operation: http://edamontology.org/operation_0404
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Computes modified cosine similarity between experimental MS2 spectra and ISDB-LOTUS
    library entries (score cutoff 0.7 for network edges; min score 0.2, ≥6 matching
    fragments, m/z tolerance 0.01)
- name: MZmine
  role: Upstream feature detection and export of per-feature MS2 spectra that serve
    as matching input
  repo: http://mzmine.github.io/
- name: SIRIUS/CSI:FingerID
  role: Independent structural annotation whose results are compared with ISDB-LOTUS
    matches to strengthen annotation confidence
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: ENPKG workflow
  role: Hosts the 03_enpkg_mn_isdb_isdb_taxo scripts implementing molecular networking,
    ISDB matching, and taxonomic reranking
  repo: https://github.com/enpkg/enpkg_workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:83dfef2a93150a9f72eae06bbcf0685694c1a3a665ac14950c94c0a995fa765f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ENPKG
  - ENPKG workflow
  - MZmine
  - RDF
  - SIRIUS/CSI:FingerID
  - matchms
  edam_operation: http://edamontology.org/operation_0404
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-library-matching-isdb-lotus

## Summary

Annotate MS2 fragmentation spectra from sample-wise LC-HRMS2 metabolomics runs by spectral matching against the ISDB-LOTUS in silico database of natural products, reranking candidate structures by taxonomic proximity and molecular-network annotation propagation.

## When to use

Use when MZmine-processed LC-HRMS2 DDA features need structural annotation against a natural-products library, especially when taxonomic metadata (e.g., Open Tree of Life ottIDs) can be exploited to reweight spectral matches.

## When NOT to use

- Input features lack associated MS2 scans (the ENPKG feature filtering keeps only features with an MS2 scan and RT 0.5–8.0 min), so there is nothing to match
- Samples have no resolvable taxonomic origin, defeating the taxonomic-distance reranking that distinguishes this matching from plain spectral search
- The data are not LC-MS/MS DDA metabolomics spectra (e.g., already-aligned feature tables without spectra, or MS1-only acquisitions)

## Inputs

- Per-sample MZmine feature lists with associated MS2 spectra (.mgf and quantification table exports), filtered to features with an MS2 scan and RT between 0.5 and 8.0 min
- ISDB-LOTUS in silico spectral library of natural products
- Taxonomic resolution per sample (Open Tree of Life ottID, Wikidata IDs) from 02_enpkg_taxo_enhancer
- Molecular network edge list (modified cosine score cutoff 0.7, max_links 10, top_n 15)

## Outputs

- Per-feature structural annotation candidates from ISDB-LOTUS with cosine scores, reranked by taxonomic distance and network annotation propagation
- Molecular network annotations that can be combined with SIRIUS/CSI:FingerID results for confidence assessment
- Structured annotation records formatted as RDF (.ttl) by the 06_enpkg_graph_builder for the ENPKG knowledge graph

## How to apply

Run the matchms-based matching in the enpkg_full step 03_enpkg_mn_isdb_isdb_taxo: build molecular-network edges with a modified cosine score cutoff of 0.7 (max_links 10, top_n 15), then match each feature's MS2 spectrum to the ISDB-LOTUS library using a minimum cosine score of 0.2 with at least 6 matching fragments and parent/fragment m/z tolerances of 0.01. Because each sample was processed individually and its species taxonomy resolved against Open Tree of Life (ott3.5) and Wikidata in step 02, rerank candidate structures by taxonomic distance and propagate annotations across the molecular network. Combine ISDB results with SIRIUS/CSI:FingerID candidates (ppm tolerance 10, maximum m/z 800, adducts [M+H]+/[M+NH4]+/[M+K]+/[M+Na]+ in PI and [M−H]−/[M+K-2H]−/[M+Na-2H]− in NI) to strengthen confidence in spectral annotation before formatting results as RDF for the knowledge graph.

## Related tools

- **matchms** (Computes modified cosine similarity between experimental MS2 spectra and ISDB-LOTUS library entries (score cutoff 0.7 for network edges; min score 0.2, ≥6 matching fragments, m/z tolerance 0.01))
- **MZmine** (Upstream feature detection and export of per-feature MS2 spectra that serve as matching input) — http://mzmine.github.io/
- **SIRIUS/CSI:FingerID** (Independent structural annotation whose results are compared with ISDB-LOTUS matches to strengthen annotation confidence) — https://bio.informatik.uni-jena.de/software/sirius/
- **ENPKG workflow** (Hosts the 03_enpkg_mn_isdb_isdb_taxo scripts implementing molecular networking, ISDB matching, and taxonomic reranking) — https://github.com/enpkg/enpkg_workflow

## Examples

```
sh workflow/00_workflow_all.sh
```

## Evaluation signals

- All matched features passed the upstream filter: they have an associated MS2 scan and an RT between 0.5 and 8.0 min
- Spectral matches respect the matchms thresholds: minimum cosine score 0.2 with at least 6 matching fragments and parent/fragment m/z tolerance 0.01
- Candidate lists are reranked (not raw top-N 15) by taxonomic distance using the OTT-resolved species of the sample and by network annotation propagation
- ISDB annotations agree at least partially with SIRIUS/CSI:FingerID results for high-confidence compounds (two-approach comparison strengthens confidence)
- At graph scale, annotations attach to the expected feature counts (788,623 features in PI and 364,967 in NI modes) and are queryable in the SPARQL endpoint of the generated knowledge graph

## Limitations

- Taxonomic reranking depends on correct resolution of the originating species against Open Tree of Life (ott3.5) and Wikidata; misresolved taxa will skew candidate reweighting
- No statistical confidence metrics (e.g., FDR, p-values) are reported for the structural annotations, including known anti-trypanosomatid compound identifications
- The merging of experimental annotations with public knowledge resources may introduce false positives whose rate is not quantified in the paper

## Evidence

- [readme] annotates features using two different methods (spectral matching to *in silico* DB coupled to taxonomical reweighting and [Sirius/CSI:FingerID]: [evidence span withheld — anchor status: external_doc]
- [methods] The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo.: "The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo."
- [results] Comparing structural annotations results from two different approaches...allows the user to strengthen confidence: [evidence span withheld — anchor status: not_found]
- [results] an in silico database of natural products (ISDB-LOTUS): "an in silico database of natural products (ISDB-LOTUS)"
