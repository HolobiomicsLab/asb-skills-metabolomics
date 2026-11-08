---
name: tanimoto-similarity-evaluation
description: Use when spectral matches must be judged by molecular structure — e.g.
  benchmarking Spec2Vec vs cosine scores, or checking whether top-N library hits for
  unknown queries are true structural analogues (Tanimoto > 0.6).
when_to_use_negative:
- Spectra lack InChIKey annotations and cannot be resolved to molecular structures
  via PubChem lookup
- The goal is peak-level spectral comparison itself (cosine/modified cosine/Spec2Vec),
  not structural validation of matches
- Defining the 'best' structural similarity measure is the research objective — the
  authors state this lies outside the study's scope and adopt Tanimoto only as a practical
  choice
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0092
tools:
- name: matchms
  role: Processes/cleans spectra metadata and computes pairwise similarity measures,
    including molecular fingerprint-based comparisons used for Tanimoto scoring
  repo: https://github.com/matchms/matchms
- name: Spec2Vec
  role: Produces the spectral similarity scores whose agreement with Tanimoto structural
    similarity is being evaluated
  repo: https://github.com/iomega/spec2vec
- name: pubchempy
  role: Automated lookup against PubChem to link spectra to InChIKeys from which molecular
    fingerprints are derived
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:eecb27c39500e231c277c58d3184ee197a2f544414f7dab9e993dc0e29b27561
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Spec2Vec
  - matchms
  - pubchempy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# tanimoto-similarity-evaluation

## Summary

Validates spectral similarity scores (e.g. Spec2Vec, cosine) by computing Tanimoto structural similarity between molecular fingerprints of matched query–library pairs, since structural similarity is the ground truth for whether a library hit is a true analogue.

## When to use

Use when spectral matches must be judged by molecular structure — e.g. benchmarking Spec2Vec vs cosine scores, or checking whether top-N library hits for unknown queries are true structural analogues (Tanimoto > 0.6).

## When NOT to use

- Spectra lack InChIKey annotations and cannot be resolved to molecular structures via PubChem lookup
- The goal is peak-level spectral comparison itself (cosine/modified cosine/Spec2Vec), not structural validation of matches
- Defining the 'best' structural similarity measure is the research objective — the authors state this lies outside the study's scope and adopt Tanimoto only as a practical choice

## Inputs

- MS/MS spectra with InChIKey annotations (e.g. AllPositive dataset, 77,092 of 95,320 spectra annotated)
- InChIKey-to-structure mappings retrieved via PubChem lookup (pubchempy)
- Matched query–library spectrum pairs with spectral similarity scores (Spec2Vec, cosine, or modified cosine)

## Outputs

- Tanimoto structural similarity scores for each query–library spectrum pair
- Fraction of queries with a best hit above Tanimoto 0.6 (e.g. ~60%, vs random-draw p = 0.0103)
- Correlation of spectral similarity scores against Tanimoto structural similarity

## How to apply

First resolve each spectrum to a molecule by linking its InChIKey annotation (obtained via an automated PubChem lookup with pubchempy, covering 128,042 of 154,820 GNPS spectra in this study). Then compute Tanimoto similarity between the molecular fingerprints of each query spectrum and its candidate library matches. Report the fraction of queries whose best (or top-10) hit exceeds a structural similarity cutoff of 0.6, and compare against the random-draw probability (p = 0.0103 for this 76,062-spectrum library and 1030 held-out queries). Also correlate spectral similarity scores (Spec2Vec, cosine, modified cosine) against Tanimoto across score distributions to quantify how well each score tracks structural similarity (Fig 3). Success criterion in the reference experiment: ~60% of the 1030 query spectra had a top-10 match with Tanimoto > 0.6, with per-query runtime ~0.14 s.

## Related tools

- **matchms** (Processes/cleans spectra metadata and computes pairwise similarity measures, including molecular fingerprint-based comparisons used for Tanimoto scoring) — https://github.com/matchms/matchms
- **Spec2Vec** (Produces the spectral similarity scores whose agreement with Tanimoto structural similarity is being evaluated) — https://github.com/iomega/spec2vec
- **pubchempy** (Automated lookup against PubChem to link spectra to InChIKeys from which molecular fingerprints are derived)

## Evaluation signals

- Fraction of queries whose best/top-10 hit exceeds Tanimoto 0.6 matches the reported ~60% (well above random-draw p = 0.0103)
- Spec2Vec score distributions show higher correlation with Tanimoto than cosine or modified cosine scores
- Every Tanimoto computation traces to a valid InChIKey-linked molecule (no unannotated spectra silently included)
- Training and query sets show no molecule overlap (200 randomly chosen InChIKeys removed, 1030 spectra held out)

## Limitations

- Tanimoto on fingerprints is a practical proxy; the authors note that finding the best measure of structural similarity lies outside their study scope
- Coverage depends on InChIKey availability — only 77,092 of 95,320 AllPositive spectra were annotated, and spectra without InChIKey must be removed to avoid train/query molecule overlap
- The Tanimoto cutoff of 0.6 for 'structural analogue' is a benchmark-specific threshold, not a universal definition

## Evidence

- [discussion] Tanimoto as ground-truth measure: "Finding the best measure to define structural similarity between two molecules lies outside the scope of this study; most recent studies converge to the Tanimoto similarity"
- [abstract] Spec2Vec tracks structural similarity better: "we show how Spec2Vec scores correlate better with structural similarity than cosine-based scores"
- [discussion] InChIKey resolution via PubChem: "an extensive automated lookup search was run against PubChem [35] (via pubchempy [36]). As a result, 128,042 out of 154,820 spectra could be linked to an InChIKey"
- [intro] structural analogue detection benchmark: "Based on the finding that Spec2Vec similarity correlates well with structural similarity (Fig 3), we investigated how Spec2Vec could be applied to molecular networking"
- [readme] matchms fingerprint comparisons: [evidence span withheld — anchor status: external_doc]
