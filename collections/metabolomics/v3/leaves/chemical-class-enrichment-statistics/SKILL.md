---
name: chemical-class-enrichment-statistics
description: Use when a batch of MS/MS spectra (e.g. .mgf exports from HILIC- or CSH-Q
  Exactive HF runs) has been assigned putative compound names at MSI level 3 and you
  need chemical-ontology class distributions or enrichment statistics rather than
  single-compound IDs.
when_to_use_negative:
- The compounds are already identified at MSI level 1 or 2 with authentic standards
  or library MS/MS matches — classification of those verified IDs does not require
  in-silico class inference and pooling them with level 3 calls inflates confidence.
- The input is raw LC-MS/MS data (e.g. Thermo .raw or mzML) rather than a list of
  annotated structures — spectral deconvolution and database search must run first.
- Only a handful of compounds are being analyzed — enrichment statistics over 21 superclasses/184
  classes are unreliable on small compound counts; prefer full per-compound ClassyFire
  annotation.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0092
tools:
- name: NIST17 hybrid search
  role: Assigns structure annotations (MSI level 3) to deconvoluted MS/MS spectra
    that lack library matches; these annotations are the input to the ClassyFire classification
    step.
- name: Sirius/CSI:FingerID
  role: In-silico fragmentation and structure prediction producing candidate structures
    that are likewise classified by ClassyFire; tested top-ranked annotations gave
    40% accuracy against MSI level 1 compounds.
  repo: https://github.com/sirius-ms/sirius
- name: ClassyFire
  role: Ontology service that maps each annotated structure's InChI key into 21 superclasses
    and 184 chemical classes, providing the categories on which proportions and enrichment
    statistics are computed.
- name: Chemical Translation Service (CTS)
  role: Converts compound names/formulas from the search results into InChI keys accepted
    by the ClassyFire API.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:7047bae529b9bcad5205b072192d3a84d71f72731c866c12604e4754cd096c81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - Chemical Translation Service (CTS)
  - ClassyFire
  - NIST17
  - NIST17 hybrid search
  - Sirius/CSI:FingerID
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# chemical-class-enrichment-statistics

## Summary

Classify MSI level 3 structure annotations from hybrid/in-silico spectral search into the ClassyFire ontology (21 superclasses, 184 chemical classes) and compute class proportions/enrichment statistics over the previously unannotated portion of the urine metabolome.

## When to use

Use when a batch of MS/MS spectra (e.g. .mgf exports from HILIC- or CSH-Q Exactive HF runs) has been assigned putative compound names at MSI level 3 and you need chemical-ontology class distributions or enrichment statistics rather than single-compound IDs.

## When NOT to use

- The compounds are already identified at MSI level 1 or 2 with authentic standards or library MS/MS matches — classification of those verified IDs does not require in-silico class inference and pooling them with level 3 calls inflates confidence.
- The input is raw LC-MS/MS data (e.g. Thermo .raw or mzML) rather than a list of annotated structures — spectral deconvolution and database search must run first.
- Only a handful of compounds are being analyzed — enrichment statistics over 21 superclasses/184 classes are unreliable on small compound counts; prefer full per-compound ClassyFire annotation.

## Inputs

- MS/MS spectral assignments from NIST17 hybrid search or Sirius/CSI:FingerID (compound names + molecular formulas, MSI level 3)
- InChI keys or structures from the Chemical Translation Service
- Condition/group metadata to test class enrichment (e.g. patient vs control urine samples)

## Outputs

- Per-compound ClassyFire annotation table (superclass + chemical class per structure)
- Class proportion/summary table across 21 superclasses and 184 classes
- Chemical class enrichment statistics comparing groups or the annotated subset vs the full signal set

## How to apply

First obtain MSI level 3 structure annotations for all unassigned MS/MS spectra using the NIST17 hybrid search and/or Sirius/CSI:FingerID in-silico fragmentation; the article reports ~95% spectral assignment coverage this way. Convert the retrieved compound names/formulas to InChI keys via the Chemical Translation Service, then submit the InChI keys in batch to the ClassyFire API, which assigns each structure to one of 21 superclasses and 184 chemical classes. Aggregate the class assignments into a proportion table and compare against the reported major-class baseline for unannotated urine metabolome signals: 28% derivatives of organic acids, 16% heterocyclic compounds, and 16% lipids. Order MSI level 3 classes by ClassyFire and use the resulting counts for chemical class enrichment statistics across conditions (e.g. diet or chemical-exposure groups in epidemiology studies). Keep MSI levels separate: classes computed from level 1 (authentic-standard confirmed, 175 compounds) or level 2 (accurate-mass/MS-MS matching, 578 compounds incl. CarniBlast acylcarnitines) subsets should not be pooled with the level 3 bulk.

## Related tools

- **NIST17 hybrid search** (Assigns structure annotations (MSI level 3) to deconvoluted MS/MS spectra that lack library matches; these annotations are the input to the ClassyFire classification step.)
- **Sirius/CSI:FingerID** (In-silico fragmentation and structure prediction producing candidate structures that are likewise classified by ClassyFire; tested top-ranked annotations gave 40% accuracy against MSI level 1 compounds.) — https://github.com/sirius-ms/sirius
- **ClassyFire** (Ontology service that maps each annotated structure's InChI key into 21 superclasses and 184 chemical classes, providing the categories on which proportions and enrichment statistics are computed.)
- **Chemical Translation Service (CTS)** (Converts compound names/formulas from the search results into InChI keys accepted by the ClassyFire API.)

## Examples

```
sirius --input patient_urine.mgf --project urine_class --canopus --csi-fingerid; python classify_classyfire.py urine_class/canopus.tsv --ontology-level superclass --proportions
```

## Evaluation signals

- Superclass counts sum to 21 and per-superclass classes resolve into the 184-class taxonomy without unmapped structures.
- Class proportions of the level 3 bulk reproduce the reported baseline: derivatives of organic acids ≈28%, heterocyclics ≈16%, lipids ≈16% for unannotated urine metabolome signals.
- Every structure entering ClassyFire has a resolvable InChI key (CTS conversion success rate ~100%); failures are logged rather than silently dropped.
- Enrichment statistics are computed only on the MSI level 3 subset; level 1 (175 compounds) and level 2 (578 compounds) annotations are tracked separately.
- Class distribution is stable across the two acquisition assays (HILIC-Q Exactive HF vs CSH-Q Exactive HF) within the range expected for the urine matrix.

## Limitations

- Structure assignments from Sirius/CSI:FingerID and NIST17 hybrid search are putative (MSI level 3) and should be validated with authentic chemicals before class-level conclusions are treated as definitive.
- Class enrichment differences between subjects can reflect diet and chemical exposure rather than biology — the article notes MSI level 3 classifications may detail such differences in epidemiology studies.
- The field lacks generally accepted, validated automated confidence-level calculations with false-discovery-rate assessment for metabolomics MS/MS annotations, so enrichment results carry unquantified FDR.
- The majority of acquired MS/MS spectra cannot be annotated with certainty; the ClassyFire distribution only covers the annotatable fraction (~95% of spectra in the article's workflow).

## Evidence

- [intro] SIRIUS is the software framework for the LC-MS/MS analysis feeding these annotations: [evidence span withheld — anchor status: external_doc]
