---
name: metabolite-identifier-mapping
description: Use when you have a list of top-ranked compound names and formulas from
  MS/MS structure annotation (e.g. NIST hybrid search or CSI:FingerID output) and
  need standardized InChI keys and chemical-ontology classes for the annotations.
when_to_use_negative:
- Compounds already identified at MSI level 1 or 2 with authentic-standard or library-spectrum
  matching, where the class can be assigned directly without name-to-InChI translation
- When only raw MS/MS spectra are available and no compound names/formulas have yet
  been produced by a structure search — run NIST hybrid search or SIRIUS/CSI:FingerID
  first
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: Chemical Translation Service (CTS)
  role: Converts top-ranked compound names and formulas from NIST17 hybrid search
    results into standardized InChI keys for ontology submission
- name: ClassyFire
  role: Batch API that classifies the batch of InChI keys into the ChemOnt ontology
    of 21 superclasses and 184 chemical classes
- name: NIST hybrid search
  role: Upstream source of the annotated compound names that feed the identifier mapping;
    classifies MSI level 3 annotations before ClassyFire ordering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:289aa789d4b1d13320bfdb2cd5dd6c6a9c5a39224ac1f67b9adda493f1229c53
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chemical Translation Service (CTS)
  - ClassyFire
  - NIST hybrid search
  - NIST17
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# metabolite-identifier-mapping

## Summary

Maps compound names and molecular formulas from structure-annotation search results (e.g. NIST17 hybrid search at MSI level 3) onto standardized InChI keys via the Chemical Translation Service (CTS), then batches those InChI keys to the ClassyFire API to assign ChemOnt classes (21 superclasses / 184 chemical classes) for downstream class-distribution analysis.

## When to use

Use when you have a list of top-ranked compound names and formulas from MS/MS structure annotation (e.g. NIST hybrid search or CSI:FingerID output) and need standardized InChI keys and chemical-ontology classes for the annotations.

## When NOT to use

- Compounds already identified at MSI level 1 or 2 with authentic-standard or library-spectrum matching, where the class can be assigned directly without name-to-InChI translation
- When only raw MS/MS spectra are available and no compound names/formulas have yet been produced by a structure search — run NIST hybrid search or SIRIUS/CSI:FingerID first

## Inputs

- Top-ranked compound names and molecular formulas from NIST17 hybrid search results (MSI level 3 annotations)
- Annotated compound list from spectral search output

## Outputs

- InChI keys corresponding to each annotated compound
- ClassyFire/ChemOnt classification per compound (superclass and chemical class)
- Summary table of chemical class proportions (e.g. 28% derivatives of organic acids, 16% heterocyclics, 16% lipids)
- CSV outputs: final annotated compound list and classification summary

## How to apply

First retrieve the top-ranked compound names and molecular formulas from the structure-search results (in this workflow, the NIST17 hybrid search that annotates deconvoluted MS/MS spectra at MSI level 3). Convert each compound name to its InChI key using the Chemical Translation Service (CTS), which acts as the identifier-normalization bridge between search-output names and the ontology API. Batch-submit the InChI keys to the ClassyFire API to obtain ChemOnt classification, expecting results spanning 21 superclasses resolved into 184 chemical classes. Aggregate the per-compound classifications into a summary table of class proportions, checking the dominant classes (in the urine metabolomics reference: 28% derivatives of organic acids, 16% heterocyclics, 16% lipids). Save both the annotated compound list and the classification summary as CSV outputs.

## Related tools

- **Chemical Translation Service (CTS)** (Converts top-ranked compound names and formulas from NIST17 hybrid search results into standardized InChI keys for ontology submission)
- **ClassyFire** (Batch API that classifies the batch of InChI keys into the ChemOnt ontology of 21 superclasses and 184 chemical classes)
- **NIST hybrid search** (Upstream source of the annotated compound names that feed the identifier mapping; classifies MSI level 3 annotations before ClassyFire ordering)

## Examples

```
python map_classyfire.py --compounds nist17_hybrid_hits.csv --name_col compound_name --via cts --api classyfire --out inchi_keys.csv class_summary.csv
```

## Evaluation signals

- Every compound name in the annotated list yields a resolvable InChI key from CTS (no unmapped entries in the intermediate table)
- ClassyFire superclass counts over the full annotation set sum to the 21 defined superclasses
- The distribution of chemical classes is dominated by the expected major classes (reference values: 28% derivatives of organic acids, 16% heterocyclics, 16% lipids)
- Classification output resolves into 184 distinct chemical classes as defined by the ChemOnt ontology
- Final outputs are well-formed CSV files: one annotated compound list and one classification-proportion summary

## Limitations

- MSI level 3 classifications may yield differences in urinary chemicals that reflect differences in subjects due to diet and chemical exposure in epidemiology studies
- Structure assignments underlying the names being mapped require validation with authentic chemicals; the majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals
- The field of metabolomics currently lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments

## Evidence

- [full_text] Workflow step: retrieve names, map via CTS, submit to ClassyFire: [evidence span withheld — anchor status: not_found]
- [full_text] Workflow step: batch ClassyFire API submission: [evidence span withheld — anchor status: not_found]
