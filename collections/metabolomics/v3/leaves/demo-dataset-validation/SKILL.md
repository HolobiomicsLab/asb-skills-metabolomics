---
name: demo-dataset-validation
description: 'Use when a reconstructed TurboPutative module (Tagger, REname, RowMerger,
  TPMetrics) must be validated: a candidate implementation exists and the demo dataset
  from https://proteomics.cnic.es/TurboPutative/ serves as the reference input.'
when_to_use_negative:
- When processing a researcher's real untargeted-metabolomics putative-annotation
  dataset for production analysis — the demo dataset is a validation fixture, not
  an analytical input
- When the code under test is not one of the four TurboPutative modules (Tagger, REname,
  RowMerger, TPMetrics), since the demo dataset and acceptance criteria are module-specific
- When no access to the TurboPutative web server is available to obtain the demo dataset
  referenced by the workflow
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: TurboPutative
  role: Source of the demo annotation dataset and of the released module behavior
    (Tagger classification, REname abbreviation/redundancy removal) against which
    the reconstruction is validated
  repo: CNIC-Proteomics/TurboPutative-web
- name: Goslin
  role: Parses lipid compound names in the demo dataset during REname validation to
    extract head group, carbon count, double bonds, linkage type, and hydroxyl/methyl
    substituents
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:d855d2613a95601452a23ced6a00d2961c47b5e3afa5c4a431b7afc55ead04c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - Goslin
  - TurboPutative
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# demo-dataset-validation

## Summary

Validates a reimplemented TurboPutative module (e.g. Tagger or REname) by running it end-to-end on the demo annotation dataset distributed through the TurboPutative web server and checking outputs against the released module's documented behavior.

## When to use

Use when a reconstructed TurboPutative module (Tagger, REname, RowMerger, TPMetrics) must be validated: a candidate implementation exists and the demo dataset from https://proteomics.cnic.es/TurboPutative/ serves as the reference input.

## When NOT to use

- When processing a researcher's real untargeted-metabolomics putative-annotation dataset for production analysis — the demo dataset is a validation fixture, not an analytical input
- When the code under test is not one of the four TurboPutative modules (Tagger, REname, RowMerger, TPMetrics), since the demo dataset and acceptance criteria are module-specific
- When no access to the TurboPutative web server is available to obtain the demo dataset referenced by the workflow

## Inputs

- Demo annotation dataset downloaded from the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/), loadable with pandas
- Candidate reimplemented module code (e.g. REname abbreviation/grouping logic or Tagger classifiers)
- Reference lookup structures used by the module (e.g. LMSD/Goslin abbreviation dictionary for REname; HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, Metabolomics Data Explorer MDM lists for Tagger)

## Outputs

- renamed_annotations.csv — deduplicated, abbreviated annotation table from the REname validation run
- tagger_output.tsv — compound-to-origin-label table from the Tagger validation run
- Pass/fail assessment of whether module behavior matches the released TurboPutative implementation

## How to apply

Download the demo dataset from the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/) and load it with pandas. Run the module under reconstruction end-to-end on this dataset, then check module-specific invariants: for REname, every set of annotation entries that shares both the same LMSD/Goslin-derived abbreviated lipid name and the same accurate mass must collapse into a single entry, and the non-lipid fallback regular expressions must yield a well-formed abbreviation for every name Goslin cannot parse; for Tagger, every compound must receive one non-empty origin label (nutrient, drug, microbiota-dependent metabolite (MDM), or natural product/plant) consistent with the classification behavior of the released module. Write the validated result table (renamed_annotations.csv for REname, tagger_output.tsv for Tagger) and treat agreement with the released module's behavior as the acceptance criterion. The demo dataset is deliberately a stand-in for the massive putative-annotation datasets TurboPutative targets, so pass criteria mirror the modules' redundant-data-removal and compound-origin-classification semantics rather than absolute content matching.

## Related tools

- **TurboPutative** (Source of the demo annotation dataset and of the released module behavior (Tagger classification, REname abbreviation/redundancy removal) against which the reconstruction is validated) — CNIC-Proteomics/TurboPutative-web
- **Goslin** (Parses lipid compound names in the demo dataset during REname validation to extract head group, carbon count, double bonds, linkage type, and hydroxyl/methyl substituents)

## Examples

```
import pandas as pd; demo = pd.read_csv('turboputative_demo_dataset.csv'); labels = tagger_classify(demo); labels.to_csv('tagger_output.tsv', sep='\t')
```

## Evaluation signals

- On the demo dataset, all REname annotation entries sharing the same abbreviated form and the same accurate mass are grouped into a single entry (redundant rows collapsed)
- The non-lipid fallback regexes produce a well-formed abbreviation for every name that Goslin cannot parse
- Every demo-dataset compound receives a non-empty origin label from Tagger
- Tagger label assignments are consistent with the classification behavior of the released Tagger module (expert_review)
- The validation output files (renamed_annotations.csv, tagger_output.tsv) are written and schema-consistent with the module's expected table structure

## Limitations

- Validation is bounded by the demo dataset's content; TurboPutative reports 80%-90% entry reduction on massive datasets, so demo-set pass does not guarantee identical behavior at full scale
- REname's fallback abbreviation path covers only non-lipid names that Goslin cannot parse; lipid parsing quality depends on the LMSD/Goslin-derived abbreviation dictionary shipped with REname
- Acceptance is behavioral consistency with the released module rather than an independent gold-standard annotation set

## Evidence

- [abstract] Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics): "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
- [abstract] REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal: "REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal"
- [other] CNIC-Proteomics/TurboPutative-web: "CNIC-Proteomics/TurboPutative-web"
