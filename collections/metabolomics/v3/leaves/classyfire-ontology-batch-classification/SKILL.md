---
name: classyfire-ontology-batch-classification
description: Use when you have a batch of MSI level 3 structure annotations (e.g.
  from NIST hybrid search or CSI:FingerID) with InChI keys or formulas and need a
  chemical-class distribution of the unannotated metabolome.
when_to_use_negative:
- Compounds already identified at MSI level 1 with authentic standards or annotated
  at MSI level 2 by precursor accurate mass and MS/MS matching — ClassyFire here targets
  only the remaining MSI level 3 annotations
- You have no InChI keys or translatable compound names/formulas, so the ClassyFire
  API cannot be queried in batch
- The analysis goal is compound identification confidence rather than chemical-class
  composition of unannotated spectra
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ClassyFire
  role: Ontology service queried with batch InChI keys to assign each MSI level 3
    annotation to one of 21 superclasses and 184 chemical classes
- name: CSI:FingerID
  role: In silico fragmentation tool whose top-ranked structure annotations provide
    the compound identities that are subsequently ClassyFire-classified
  repo: github.com/sirius-ms/sirius
- name: NIST hybrid search
  role: Hybrid spectral search against NIST17 that produces the MSI level 3 compound
    names and formulas converted to InChI keys for classification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:b7794b70f486b767e29dc699c66bc11fc95366ae57813a03b087b70b698c5882
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSI:FingerID
  - ClassyFire
  - NIST hybrid search
  - NIST17
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# ClassyFire ontology batch classification

## Summary

Batch-annotates computationally assigned (MSI level 3) metabolite structures with the ClassyFire chemical ontology, resolving each compound into 21 superclasses and 184 chemical classes to give a chemical-class distribution of previously unannotated spectra.

## When to use

Use when you have a batch of MSI level 3 structure annotations (e.g. from NIST hybrid search or CSI:FingerID) with InChI keys or formulas and need a chemical-class distribution of the unannotated metabolome.

## When NOT to use

- Compounds already identified at MSI level 1 with authentic standards or annotated at MSI level 2 by precursor accurate mass and MS/MS matching — ClassyFire here targets only the remaining MSI level 3 annotations
- You have no InChI keys or translatable compound names/formulas, so the ClassyFire API cannot be queried in batch
- The analysis goal is compound identification confidence rather than chemical-class composition of unannotated spectra

## Inputs

- Batch of compound names and formulas from NIST17 hybrid search or CSI:FingerID MSI level 3 annotations
- InChI keys retrieved via the Chemical Translation Service (CTS)
- Deconvoluted MS/MS spectra from HILIC-Q Exactive HF and CSH-Q Exactive HF untargeted metabolomics runs (source of the annotations)

## Outputs

- ClassyFire ontology classification per compound (superclass and chemical class)
- Summary table of chemical class proportions (e.g. 21 superclasses / 184 classes)
- CSV of annotated compound list with ClassyFire classes for downstream chemical class enrichment statistics

## How to apply

Collect the top-ranked compound names and formulas returned by the NIST17 hybrid search (or CSI:FingerID) for the remaining MS/MS spectra, then convert them to InChI keys using the Chemical Translation Service (CTS). Submit the batch of InChI keys to the ClassyFire API to obtain the ontology classification, which in this urine metabolomics study resolved annotations into 21 superclasses and 184 chemical classes. Aggregate the returned superclass/class assignments into a summary table of class proportions and inspect the top classes for biological plausibility (here 28% derivatives of organic acids, 16% heterocyclics, 16% lipids). The resulting class-ordered MSI level 3 annotations can also feed chemical class enrichment statistics.

## Related tools

- **ClassyFire** (Ontology service queried with batch InChI keys to assign each MSI level 3 annotation to one of 21 superclasses and 184 chemical classes)
- **CSI:FingerID** (In silico fragmentation tool whose top-ranked structure annotations provide the compound identities that are subsequently ClassyFire-classified) — github.com/sirius-ms/sirius
- **NIST hybrid search** (Hybrid spectral search against NIST17 that produces the MSI level 3 compound names and formulas converted to InChI keys for classification)

## Evaluation signals

- Every MSI level 3 annotation has a resolvable InChI key via CTS before the ClassyFire batch submission
- ClassyFire output partitions annotations into exactly 21 superclasses and 184 chemical classes, matching the reported ontology granularity
- The aggregated class proportions reproduce the reported major classes: 28% derivatives of organic acids, 16% heterocyclics, 16% lipids
- Summary table superclasses/classes sum to 100% of classified annotations with no unclassified leftovers
- Final annotated compound list and classification summary are exported as CSV outputs

## Limitations

- ClassyFire classification is applied to MSI level 3 (putative, in silico/hybrid-search) annotations, which per the article require validation with authentic chemicals before structural claims are made
- MSI level 3 class distributions may yield differences in urinary chemicals driven by diet and chemical exposure, so class proportions are cohort- and study-dependent
- The article notes the field lacks validated automated confidence-level calculations with false-discovery rate assessment for metabolomics MS/MS annotations, so class-level results inherit annotation uncertainty

## Evidence

- [readme] SIRIUS framework description: [evidence span withheld — anchor status: external_doc]
