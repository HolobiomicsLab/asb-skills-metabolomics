---
name: clustering-cutoff-parameter-sensitivity-analysis
description: Decision support for choosing a clustering threshold or alignment mode
  (e.g., BiG-SCAPE 0.30 vs 0.50, global vs glocal) when an independent validation
  signal such as known metabolite-GCF pairs is available to score each parameter combination.
when_to_use_negative:
- Only one parameter combination can be computed or no external validation criterion
  (e.g., known molecule-GCF pairs) exists to rank the combinations — the comparison
  is then unjustifiable.
- The input is not a pairwise similarity/distance network from which families can
  be re-derived (e.g., GCF assignments are already fixed and cannot be recomputed
  at alternative cutoffs).
- The downstream question does not involve linking gene cluster families to molecular
  features, since the selection criterion here is metabologenomic correlation performance.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SCAPE
  role: Computes BGC sequence similarity networks at each mode/cutoff combination
    and applies Affinity Propagation to form GCFs
  repo: https://github.com/medema-group/BiG-SCAPE
- name: HMMER 3.1b2
  role: Detects conserved Pfam v31 domains that underlie the pairwise distances compared
    across cutoff settings
- name: antiSMASH
  role: Produces the BGC GenBank predictions used as BiG-SCAPE inputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:cb324a903eab858047aa4e8b2d7ab65c9e4fd1bf9583e06466e70c0a0ff3f37a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/medema-group/BiG-SCAPE
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# clustering-cutoff-parameter-sensitivity-analysis

## Summary

Systematically tests a clustering pipeline (here BiG-SCAPE alignment mode × distance cutoff) across parameter combinations and selects the default by an external validation criterion — metabologenomic correlation tails of known molecule-GCF pairs — rather than by network statistics alone.

## When to use

Decision support for choosing a clustering threshold or alignment mode (e.g., BiG-SCAPE 0.30 vs 0.50, global vs glocal) when an independent validation signal such as known metabolite-GCF pairs is available to score each parameter combination.

## When NOT to use

- Only one parameter combination can be computed or no external validation criterion (e.g., known molecule-GCF pairs) exists to rank the combinations — the comparison is then unjustifiable.
- The input is not a pairwise similarity/distance network from which families can be re-derived (e.g., GCF assignments are already fixed and cannot be recomputed at alternative cutoffs).
- The downstream question does not involve linking gene cluster families to molecular features, since the selection criterion here is metabologenomic correlation performance.

## Inputs

- antiSMASH-predicted BGC GenBank files for 363 strains
- BiG-SCAPE filtered sequence similarity networks per mode/cutoff (global/glocal × 0.30/0.50)
- Binary GCF presence/absence matrix (363 strains × GCFs)
- Ion presence/absence table (5,824 ions detected in two or more strains)

## Outputs

- GCF assignments via Affinity Propagation per mode/cutoff combination
- Binary metabologenomic correlation scores per known molecule-GCF pair
- Selection summary and default mode/cutoff choice (glocal/0.30)
- metabologenomic_correlations.csv with per-pair correlation scores

## How to apply

Run the clustering tool at each parameter combination on the same inputs — here BiG-SCAPE (HMMER 3.1b2, Pfam 31) in global and glocal modes, filtering networks at distance cutoffs 0.30 and 0.50 on antiSMASH-predicted BGC GenBank files from the 363 strains. For each combination, assign GCFs by Affinity Propagation clustering on the filtered network and build a binary 363-strain × GCF presence/absence matrix. Restrict to correlatable GCFs (representatives in two or more strains) and correlate presence/absence against the 5,824-ion table (ions in ≥2 strains, mass within 4 ppm, cosine similarity > 0.75). Select the default combination (glocal/0.30) by checking that at least 6 of the 9 known molecule-GCF pairs fall in the rightmost tail of the correlation-score distribution, and write per-pair scores to metabologenomic_correlations.csv.

## Related tools

- **BiG-SCAPE** (Computes BGC sequence similarity networks at each mode/cutoff combination and applies Affinity Propagation to form GCFs) — https://github.com/medema-group/BiG-SCAPE
- **HMMER 3.1b2** (Detects conserved Pfam v31 domains that underlie the pairwise distances compared across cutoff settings)
- **antiSMASH** (Produces the BGC GenBank predictions used as BiG-SCAPE inputs)

## Evaluation signals

- All four mode/cutoff combinations (global/glocal × 0.30/0.50) are computed on identical BGC inputs before any selection is made
- Only GCFs with representative gene clusters in two or more strains enter the correlation step (correlatable GCFs)
- The selected combination (glocal/0.30) reproduces the validation result: at least 6 of the 9 known molecule-GCF combinations fall in the rightmost tail of the correlation-score distribution
- Per-pair correlation scores are written to metabologenomic_correlations.csv and the selection summary is recorded
- The ion table matches the paper's filter: 5,824 ions with mass within 4 ppm and fragmentation cosine similarity above 0.75, detected in two or more strains

## Limitations

- The cutoff is empirical: BiG-SCAPE filters the network with a predefined cutoff distance (0.3 in the paper's default), and the value that maximizes nodes while preserving network structural integrity was reported separately as 0.75 in a network-integrity analysis, so cutoff choice is context-dependent.
- Selection quality depends on the validation set: here only 9 known molecule-GCF pairs, and at least 6/9 in the right tail is a threshold-based judgement, not a statistical test.
- Domain-detection parameters must be held constant (HMMER 3.1b2 with Pfam version 31) or cutoff comparisons conflate tool-version effects with cutoff effects.

## Evidence

- [readme] BiG-SCAPE's clustering role across networks: [evidence span withheld — anchor status: external_doc]
