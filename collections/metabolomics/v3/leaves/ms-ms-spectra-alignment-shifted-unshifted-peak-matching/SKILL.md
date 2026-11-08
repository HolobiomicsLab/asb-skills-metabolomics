---
name: ms-ms-spectra-alignment-shifted-unshifted-peak-matching
description: Use when you have MS/MS spectra of a known compound and a structural
  analog (same adduct, one modification site) and need to classify matching/shifted/unshifted
  fragment peaks to localize where the structural modification occurs.
when_to_use_negative:
- The pair has no shifted peak explainable by a substructure of the parent compound
  — after this filter only 62% of benchmark pairs (8033) remain, so pairs failing
  it cannot be localized this way
- The compounds differ at more than one modification site or use different adducts
  (benchmark eligibility requires exactly one modification site via RDKit GetSubstructMatch
  edge count and identical M+H adduct)
- Precursor mass exceeds 2000 Da or the precursor mass difference exceeds 50% of the
  smaller compound (benchmark eligibility checks i–iv)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ModiFinder
  role: Implements the MS2-known vs MS2-unknown spectral alignment (bipartite greedy
    matching) and consumes the shifted/unshifted peak classification for modification
    site scoring
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Provides combinatorial fragmentation (breadth-first, depth 2) used to assign
    candidate substructures to the aligned shifted/unshifted peaks within 40 ppm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:88d946056bd59a5aae99cebf8a9f7b12f7d9c52674fa198e33468f5b68e70c29
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ms-ms-spectra-alignment-shifted-unshifted-peak-matching

## Summary

Aligns MS/MS spectra of a structurally related known and unknown compound pair by greedy bipartite-graph matching, classifying peak matches as shifted or unshifted; shifted peaks mark substructures of the known molecule that putatively contain the modification site, which is the core signal ModiFinder exploits for modification site localization.

## When to use

Use when you have MS/MS spectra of a known compound and a structural analog (same adduct, one modification site) and need to classify matching/shifted/unshifted fragment peaks to localize where the structural modification occurs.

## When NOT to use

- The pair has no shifted peak explainable by a substructure of the parent compound — after this filter only 62% of benchmark pairs (8033) remain, so pairs failing it cannot be localized this way
- The compounds differ at more than one modification site or use different adducts (benchmark eligibility requires exactly one modification site via RDKit GetSubstructMatch edge count and identical M+H adduct)
- Precursor mass exceeds 2000 Da or the precursor mass difference exceeds 50% of the smaller compound (benchmark eligibility checks i–iv)

## Inputs

- MS/MS spectrum (peak list of m/z, intensity) of the structurally known compound (MS2-known)
- MS/MS spectrum (peak list of m/z, intensity) of the unknown analog compound (MS2-unknown)
- Parent/adduct metadata for the pair (e.g., identical M+H adduct, precursor masses)

## Outputs

- Set of shifted peak matches (m/z-differing pairs) indicating substructures containing the modification site
- Set of unshifted peak matches (m/z-consistent pairs within 40 ppm)
- Substructure annotations per peak for downstream atom-likelihood scoring

## How to apply

First preprocess both spectra: remove all peaks with intensity below 1% of the base peak and normalize intensities to a Euclidean norm of 1. Then build a bipartite graph whose nodes are the peaks of the MS2-known and MS2-unknown spectra and run the GNPS greedy bipartite-graph matching with a 40 ppm m/z tolerance to obtain peak matches. Matches whose m/z differ (beyond tolerance) are the shifted peaks — they putatively represent substructures of the known molecule containing the modification site — while matches within tolerance are unshifted peaks that anchor the shared scaffold. Annotate the shifted and unshifted peaks with possible substructures using MAGMa combinatorial fragmentation (breadth-first, maximum depth 2) before scoring atoms. Downstream, ModiFinder rewards atoms in substructures assigned to shifted peaks and penalizes atoms in substructures assigned to unshifted peaks.

## Related tools

- **ModiFinder** (Implements the MS2-known vs MS2-unknown spectral alignment (bipartite greedy matching) and consumes the shifted/unshifted peak classification for modification site scoring) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Provides combinatorial fragmentation (breadth-first, depth 2) used to assign candidate substructures to the aligned shifted/unshifted peaks within 40 ppm)

## Evaluation signals

- Preprocessing invariant: all peaks below 1% of the base peak are removed and remaining intensities normalize to Euclidean norm 1 before alignment
- Alignment correctness: peak matches are recovered with the GNPS greedy bipartite-graph matching at 40 ppm tolerance; unshifted matches should be within tolerance and shifted matches exceed it
- Benchmark reproduction: filtering to pairs with at least one shifted peak explainable by a substructure retains 62% of pairs (8033 pairs)
- End-to-end reproduction: mean Average-distance Evaluation Scores of 0.603 (MF-N), 0.625 (MF-R), and 0.652 (MF-O) on the benchmark (Fig. 3-A)

## Limitations

- Only pairs with at least one shifted peak explainable by a substructure of the known compound can be evaluated — ~38% of benchmark pairs are excluded by this filter
- Shifted peaks only 'putatively' represent substructures containing the modification site; the oracle variant (MF-O) must remove substructures not containing the true site, showing raw annotations are imperfect
- Results depend on the 40 ppm tolerance choice and on library spectra sharing the same adduct and instrument conditions (required for helper-compound refinement)

## Evidence

- [readme] ModiFinder is a tool for site localization of structural modifications using MS/MS data.: [evidence span withheld — anchor status: not_found]
