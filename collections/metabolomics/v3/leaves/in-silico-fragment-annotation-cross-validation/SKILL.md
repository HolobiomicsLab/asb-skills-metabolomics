---
name: in-silico-fragment-annotation-cross-validation
description: Use when Mass2Motif substructure assignments (e.g., ring or tail motifs
  from MS2LDA) need independent confirmation against in silico fragment annotations
  before being trusted for molecular family interpretation.
when_to_use_negative:
- When the molecular family has no MS2LDA motif assignments to validate (no motif-document
  relations pass the prob/overlap thresholds) — there is nothing to cross-validate.
- When DEREPLICATOR or the chosen in silico tool cannot annotate the compound class
  (e.g., non-NRP/polyketide scaffolds outside its rule-based scope), forcing cross-validation
  against irrelevant fragments.
- When the goal is pure chemical-class propagation rather than substructure-level
  validation — ClassyFire-based chemical annotation through MolNetEnhancer is the
  appropriate skill instead.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEREPLICATOR
  role: Provides independent in silico structure/fragment annotations used as the
    cross-validation reference for MS2LDA Mass2Motif assignments
- name: MS2LDA
  role: Source of the Mass2Motif substructure assignments (ring, tail motifs) being
    validated against fragment annotations
- name: pyMolNetEnhancer
  role: Maps the filtered Mass2Motif assignments onto GNPS network nodes/edges (EdgesMotifs2Network)
    so motif–feature links can be checked against fragment annotations
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: GNPS
  role: Platform supplying the molecular network and clustered MGF spectra underlying
    both the MS2LDA experiment and the in silico annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:021027df279875772ee5398a481e672d548842fbeb3d4088c205c13baa1aeb97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - DEREPLICATOR
  - GNPS
  - MS2LDA
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# in-silico fragment annotation cross-validation

## Summary

Cross-validates MS2LDA Mass2Motif substructure assignments against independent in silico fragment annotations (e.g., DEREPLICATOR/VarQuest) to confirm that motif-derived substructures are supported by predicted fragments, as done for the xenoamicin ring and tail motifs in Xenorhabdus molecular families.

## When to use

Use when Mass2Motif substructure assignments (e.g., ring or tail motifs from MS2LDA) need independent confirmation against in silico fragment annotations before being trusted for molecular family interpretation.

## When NOT to use

- When the molecular family has no MS2LDA motif assignments to validate (no motif-document relations pass the prob/overlap thresholds) — there is nothing to cross-validate.
- When DEREPLICATOR or the chosen in silico tool cannot annotate the compound class (e.g., non-NRP/polyketide scaffolds outside its rule-based scope), forcing cross-validation against irrelevant fragments.
- When the goal is pure chemical-class propagation rather than substructure-level validation — ClassyFire-based chemical annotation through MolNetEnhancer is the appropriate skill instead.

## Inputs

- MS2LDA Mass2Motif–document relations table (e.g., MotifFS2.csv) filtered at prob >= 0.01 and overlap >= 0.3
- Mass2Motif-to-network mapping output from pyMolNetEnhancer (per-node/per-edge motif annotations, .graphml or TSV)
- GNPS molecular network (classical or feature-based) for the molecular family of interest
- In silico annotation output from DEREPLICATOR (or NAP or another tool of preference) with predicted structures/fragments
- MGF clustered spectra used for the MS2LDA experiment

## Outputs

- Per-feature consistency table linking each Mass2Motif assignment to DEREPLICATOR/VarQuest fragment annotations
- Fragment-consistency percentage per motif (must exceed 75% for validated ring/tail motifs)
- Validated substructure-membership breakdown per molecular family (e.g., 4 ring A + tail, 10 ring A only, 3 tail only, 2 ring A + ring B + tail, 2 ring B + tail in the 32-member xenoamicin family)

## How to apply

After mapping MS2LDA Mass2Motifs onto a GNPS molecular network with pyMolNetEnhancer using prob=0.01 and overlap=0.3, run an independent in silico structure annotation pipeline such as DEREPLICATOR (with VarQuest fragment matching) on the same spectra. For each motif-linked molecular feature, check whether the fragments explained by the Mass2Motif correspond to fragments predicted/annotated by the in silico tool, and compute the fraction of linked features where the motif fragments are consistent with the annotation. The article's validation standard requires this fragment-consistency to exceed 75% for the ring and tail motifs (which were 'consistently present in more than 75% of their linked molecular features'). Treat the two pipelines as complementary: the article explicitly concludes that PyMolNetEnhancer and DEREPLICATOR outputs 'are complementary, therefore, researchers should use both tools to annotate their molecular networks,' so agreement between them strengthens substructure claims while disagreement flags motifs needing manual spectral inspection.

## Related tools

- **DEREPLICATOR** (Provides independent in silico structure/fragment annotations used as the cross-validation reference for MS2LDA Mass2Motif assignments)
- **MS2LDA** (Source of the Mass2Motif substructure assignments (ring, tail motifs) being validated against fragment annotations)
- **pyMolNetEnhancer** (Maps the filtered Mass2Motif assignments onto GNPS network nodes/edges (EdgesMotifs2Network) so motif–feature links can be checked against fragment annotations) — https://github.com/madeleineernst/pyMolNetEnhancer
- **GNPS** (Platform supplying the molecular network and clustered MGF spectra underlying both the MS2LDA experiment and the in silico annotation)

## Examples

```
pip install pyMolNetEnhancer  # then in Mass2Motifs_2_Network_Classical.ipynb: map Motifs2Networks with prob=0.01, overlap=0.3, top=5, and compare motif links against DEREPLICATOR/VarQuest fragment annotations
```

## Evaluation signals

- Fragment-consistency of each validated Mass2Motif (e.g., xenoamicin ring and tail motifs) against DEREPLICATOR/VarQuest annotations must exceed 75% of linked molecular features.
- Motif-membership counts across the molecular family must match the expected breakdown exactly (4 both ring A + tail, 10 ring A only, 3 tail only, 2 ring A + ring B + tail, 2 ring B + tail of 32 members).
- All retained motif–document relations must pass the thresholds prob >= 0.01 and overlap >= 0.3; the ms2lda.org summary table must reflect the same thresholds set under Experimental Options.
- The annotated .graphml must contain per-node motif presence and per-edge 'interaction' (shared Mass2Motif) attributes plus a TopSharedMotifs column (top=5 per network component).
- PyMolNetEnhancer and DEREPLICATOR outputs should be compared side-by-side and treated as complementary; systematic contradiction between motif fragments and in silico fragments flags unreliable motifs.

## Limitations

- Server connection timeouts can occur when retrieving large MS2LDA summary files from ms2lda.org; download the summary manually via Export MS2LDA Summary as a fallback.
- The ms2lda.org summary table already contains filtered motif-document relations using thresholds set in the web app, so thresholds set there must match those used downstream to avoid double or inconsistent filtering.
- In silico annotation tools differ in output format and efficient cross-tool data matching is lacking, which is precisely the integration gap MolNetEnhancer addresses; manual format reconciliation may be needed when adding non-default tools.
- The article recommends using both PyMolNetEnhancer and DEREPLICATOR since neither alone fully annotates the network — cross-validation is complementary evidence, not a replacement for either tool.

## Evidence

- [abstract] MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR): "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR)"
- [methods] prob minimal probability score: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [methods] overlap minimal overlap score: "<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [discussion] comparison of outputs complementary: [evidence span withheld — anchor status: not_found]
- [readme] summary table contains filtered motif-document relations: "Importantly, the summary table contains filtered motif-document relations using the set thresholds in the web app."
- [methods] another tool of preference for in silico structure annotation: "[DEREPLICATOR](https://ccms-ucsd.github.io/GNPSDocumentation/dereplicator/) or another tool of preference for <i>in silico</i> structure annotation"
