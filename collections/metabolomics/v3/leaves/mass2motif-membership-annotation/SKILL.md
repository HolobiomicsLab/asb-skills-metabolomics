---
name: mass2motif-membership-annotation
description: Use when you have decomposition output (motifset.json / ms2lda_viz.gz)
  from MS2LDA or the GNPS–MS2LDA repository, MS/MS spectra from complex mixtures,
  and you want motif-based annotation instead of library matching.
when_to_use_negative:
- The input is already a fully annotated feature table and you only need to re-quantify
  peaks — motif discovery is redundant.
- Spectra come from DDA data where MS1-to-MS2 matching is unreliable, because motif
  decomposition depends on reliable MS1–MS2 peak matching.
- You need to compare conditions across samples with PLAGE or differential-expression
  analyses; this skill covers annotation only, not motif-level differential expression.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2LDA
  role: Performs LDA decomposition of MS/MS spectra into Mass2Motifs and provides
    the posterior probabilities used to assign motif membership
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAvis
  role: Visualizes the motif–spectrum network to inspect which spectra are connected
    by shared Mass2Motifs
- name: Molecular Networking
  role: Produces molecular-network output used alongside MS2LDAvis to contextualize
    motif-based substructure grouping
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:eba636255f66d4be9b67b791b0ffc7a323478efb5a88fd7c78270444aa0d4426
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  - MS2LDAvis
  - Molecular Networking
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mass2motif-membership-annotation

## Summary

Annotate metabolite MS/MS spectra by their membership in Mass2Motifs — recurring fragment/neutral-loss substructures learned by LDA — rather than by whole-spectrum library matching, yielding up to three-fold more annotations in complex mixtures like beer extracts.

## When to use

Use when you have decomposition output (motifset.json / ms2lda_viz.gz) from MS2LDA or the GNPS–MS2LDA repository, MS/MS spectra from complex mixtures, and you want motif-based annotation instead of library matching.

## When NOT to use

- The input is already a fully annotated feature table and you only need to re-quantify peaks — motif discovery is redundant.
- Spectra come from DDA data where MS1-to-MS2 matching is unreliable, because motif decomposition depends on reliable MS1–MS2 peak matching.
- You need to compare conditions across samples with PLAGE or differential-expression analyses; this skill covers annotation only, not motif-level differential expression.

## Inputs

- motifset.json / ms2lda_viz.json.gz MS2LDA decomposition output
- Mass2Motifs with structural characterizations (e.g., the 30 beer Mass2Motifs)
- MS/MS spectra from the experiment being annotated
- Mass Bank and NIST MS/MS library spectra for benchmarking

## Outputs

- Per-spectrum motif membership probabilities
- List of motif-annotated molecules (spectrum–motif pairs above 0.1 probability)
- Annotation count comparison (motif-based vs. library-based) for reproducibility check

## How to apply

Load the precomputed MS2LDA decomposition of your spectra together with a set of structurally characterized Mass2Motifs (e.g., the 30 characterized beer motifs from SI Appendix section S2.1). Compute the posterior probability that each spectrum loads on each characterized motif from the variational Bayesian LDA posterior; mark a molecule as motif-annotated when at least one characterized motif (fragment and neutral-loss pattern) has load above 0.1 probability. Tabulate annotation counts per molecule in positive and negative ionization mode and compare them with spectral matching (90% normalized score threshold against Mass Bank/NIST, omitting neutral losses since spectral matching tools do not support them) to assess gain. Rationale: motif membership captures substructure-level biochemical similarity independent of whole-spectrum spectral similarity, enabling grouping and annotation of molecules that library matching misses.

## Related tools

- **MS2LDA** (Performs LDA decomposition of MS/MS spectra into Mass2Motifs and provides the posterior probabilities used to assign motif membership) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAvis** (Visualizes the motif–spectrum network to inspect which spectra are connected by shared Mass2Motifs)
- **Molecular Networking** (Produces molecular-network output used alongside MS2LDAvis to contextualize motif-based substructure grouping)

## Examples

```
pip install ms2lda
```

## Evaluation signals

- Motif-annotation count should be roughly 3× the number of library hits (Mass Bank hits 25%, NIST 6% of spectra) when benchmarked against the characterized beer motifs.
- Probability threshold: only motif memberships with probability > 0.1 are accepted, consistent with the paper's 81.5% and 63.3% substructure-annotation validation rates in positive and negative mode respectively.
- Each annotated molecule should show at least one characterized motif load > 0.1; if no motif crosses the threshold, the molecule is excluded from the motif-annotated set.
- Positive- and negative-ionization-mode results should be tabulated separately to reproduce the reported mode-specific gains.

## Limitations

- Neutral losses are not supported by standard spectral-matching tools and had to be omitted during library-matching comparison, so the comparison is not strictly symmetric.
- MS2LDA works best for complex mixtures with information-rich MS2 spectra (ramped or stepped collision energy); simple spectra with few fragments yield weak motif decompositions.
- Computational cost is higher than simpler tools, though not prohibitive.
- Only a subset of Mass2Motifs is structurally characterized; uncharacterized motifs cannot be used for direct structural annotation.
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs; other modes/matrices were not reported.

## Evidence

- [results] Three-fold annotation gain over library matching: [evidence span withheld — anchor status: not_found]
- [discussion] Limitation: MS2LDA works best on information-rich spectra: [evidence span withheld — anchor status: not_found]
