---
name: chemont-hierarchy-mapping
description: Use when top-ranked per-feature annotations (e.g. from MAW's mergedResults-with-one-Candidates.csv)
  carry CANOPUS/ClassyFire class strings that must be rolled up into ChemOnt superclass/class/subclass
  proportions.
when_to_use_negative:
- Features are unannotated or lack chemical class assignments — CANOPUS/ClassyFire
  output is a prerequisite for ChemOnt roll-up
- A single compound needs structure confirmation rather than dataset-level class composition
  — use spectral matching and candidate scoring instead
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3336
tools:
- name: MAW
  role: Source of the merged top-ranking candidates file and the classification step
    whose CANOPUS/ClassyFire output is mapped onto ChemOnt
  repo: https://github.com/zmahnoor14/MAW
- name: CANOPUS
  role: Produces the per-candidate chemical class predictions that are aggregated
    at ChemOnt superclass/class/subclass levels
- name: ClassyFire
  role: Defines the ChemOnt ontology used to classify candidates and place class assignments
    in the superclass/class/subclass hierarchy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:3c8351b759653f6038ad2ad924895e589b8726a96322787a6c5cee44cde3822e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - CANOPUS
  - ClassyFire
  - MAW
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chemont-hierarchy-mapping

## Summary

Maps chemical class assignments from CANOPUS/ClassyFire output onto the ChemOnt hierarchy (superclass, class, subclass) and aggregates annotation counts per level, e.g. for sunburst or bar summaries of classified metabolome annotation candidates. Rationale: MAW classifies tentative candidates computationally so structures can be grouped by chemical ontology rather than judged individually.

## When to use

Use when top-ranked per-feature annotations (e.g. from MAW's mergedResults-with-one-Candidates.csv) carry CANOPUS/ClassyFire class strings that must be rolled up into ChemOnt superclass/class/subclass proportions.

## When NOT to use

- Features are unannotated or lack chemical class assignments — CANOPUS/ClassyFire output is a prerequisite for ChemOnt roll-up
- A single compound needs structure confirmation rather than dataset-level class composition — use spectral matching and candidate scoring instead

## Inputs

- MAW final merged candidates table (mergedResults-with-one-Candidates.csv) with one top-ranking annotation per feature
- Per-feature chemical class assignments from CANOPUS and ClassyFire classification

## Outputs

- subclass_abundances.csv with ChemOnt subclass frequencies as percentages of annotated features
- Sunburst chart of superclass/class/subclass hierarchy (class_sunburst.png)

## How to apply

Load MAW's final candidate table (mergedResults-with-one-Candidates.csv), keep only the top-ranking annotation per MS2 feature, and parse each candidate's chemical class assignment produced by the computational classification step (CANOPUS + ClassyFire). Map every assignment onto the ClassyFire ChemOnt ontology so each feature can be counted at the superclass, class, and subclass nodes. Aggregate counts and express subclass abundances as percentages of annotated features (e.g. subclass_abundances.csv), then render the hierarchy — e.g. a sunburst with subclass segments sized by abundance (class_sunburst.png). Validate by comparing the reproduced subclass fractions against published values (bryophytes dataset: Lipids 33.92%, Organic acids 17.02%, Benzenoids 13.94%, Phenylpropanoid and polyketides 11.34%) within rounding tolerance.

## Related tools

- **MAW** (Source of the merged top-ranking candidates file and the classification step whose CANOPUS/ClassyFire output is mapped onto ChemOnt) — https://github.com/zmahnoor14/MAW
- **CANOPUS** (Produces the per-candidate chemical class predictions that are aggregated at ChemOnt superclass/class/subclass levels)
- **ClassyFire** (Defines the ChemOnt ontology used to classify candidates and place class assignments in the superclass/class/subclass hierarchy)

## Examples

```
python -m maw.classify --input mergedResults-with-one-Candidates.csv --hierarchy chemont --output subclass_abundances.csv
```

## Evaluation signals

- Reproduced subclass percentages match reported values: Lipids 33.92%, Organic acids 17.02%, Benzenoids 13.94%, Phenylpropanoid and polyketides 11.34% within rounding tolerance
- All four reported subclass labels appear in the aggregated output
- Sum of subclass-level shares reconciles with the number of classified top-ranking features (no double counting across hierarchy levels)
- Sunburst renders superclass→class→subclass with segment areas proportional to computed abundances

## Limitations

- CANOPUS class predictions can disagree with later candidate verification, so class distribution describes tentative candidates, not confirmed structures
- Lipid classes annotated via SIRIUS/CANOPUS can produce false positives because of the complex nature of lipids and co-eluting isomers
- Public databases are not fully standardized, so class coverage depends on which features obtained class assignments at all

## Evidence

- [full_text] Subclass percentages validation: "Lipids (33.92%), Organic acids (17.02%), Benzenoids (13.94%), and Phenylpropanoid and polyketides (11.34%)"
- [full_text] Classification toolchain: [evidence span withheld — anchor status: not_found]
- [full_text] Final candidate selection: "Final candidate selection is done in Python using [RDKit] and [PubChemPy]"
- [abstract] Annotation integration: "MAW can improve candidate ranking by integrating spectral databases with annotation tools like SIRIUS"
