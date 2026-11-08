---
name: sunburst-hierarchy-visualization
description: Use when MAW's final per-feature candidates (e.g. file_id_mergedResults-with-one-Candidates.csv)
  carry CANOPUS/ClassyFire class assignments and the goal is a hierarchical sunburst
  plus subclass abundance percentages.
when_to_use_negative:
- Input is raw MS2 (.mzML) spectra or MAW candidates that lack CANOPUS/ClassyFire
  class assignments — the hierarchy aggregation requires classified top-ranking annotations.
- Annotation dereplication itself is the goal — this skill only summarizes and visualizes
  MAW's already prioritized per-feature candidates, it does not perform spectral or
  compound database dereplication.
- A flat (non-hierarchical) summary is needed — mapping onto the ChemOnt superclass/class/subclass
  hierarchy adds structure that a simple class count table does not require.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: CANOPUS
  role: Predicts the chemical classes of tentative candidates that are aggregated
    into the ChemOnt hierarchy for the sunburst
  repo: https://bio.informatik.uni-jena.de/software/canopus/
- name: ClassyFire
  role: Provides the ChemOnt superclass/class/subclass classification scheme onto
    which each assignment is mapped
- name: MAW-Py
  role: Produces the per-feature final candidate selection (via RDKit) whose class
    assignments feed the subclass abundance aggregation
  repo: https://github.com/zmahnoor14/MAW
- name: RDKit
  role: Used in MAW-Py for the final candidate selection that determines which annotations
    are summarized
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:a232d65e4016b55402ad015690a3caa34679bd239b743722ca065a1b2c7384c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - ClassyFire
  - MAW-Py
  - RDKit
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sunburst-hierarchy-visualization

## Summary

Summarizes the chemical class assignments of MAW's classified top-ranking annotations (produced via CANOPUS/ClassyFire) into a ChemOnt superclass–class–subclass sunburst chart, reporting subclass abundances as percentages of annotated features.

## When to use

Use when MAW's final per-feature candidates (e.g. file_id_mergedResults-with-one-Candidates.csv) carry CANOPUS/ClassyFire class assignments and the goal is a hierarchical sunburst plus subclass abundance percentages.

## When NOT to use

- Input is raw MS2 (.mzML) spectra or MAW candidates that lack CANOPUS/ClassyFire class assignments — the hierarchy aggregation requires classified top-ranking annotations.
- Annotation dereplication itself is the goal — this skill only summarizes and visualizes MAW's already prioritized per-feature candidates, it does not perform spectral or compound database dereplication.
- A flat (non-hierarchical) summary is needed — mapping onto the ChemOnt superclass/class/subclass hierarchy adds structure that a simple class count table does not require.

## Inputs

- MAW merged candidates file with one final candidate per feature (file_id_mergedResults-with-one-Candidates.csv)
- Chemical class assignments per candidate from MAW's computational classification step (CANOPUS predictions resolved against the ClassyFire ChemOnt hierarchy)

## Outputs

- subclass_abundances.csv (subclass abundance percentages of annotated features)
- class_sunburst.png (sunburst chart of the superclass/class/subclass hierarchy with subclass segments sized by abundance)

## How to apply

Load the final MAW merged candidates results (file_id_mergedResults-with-one-Candidates.csv) keeping only the top-ranking annotation per feature, since MAW prioritizes one final candidate per feature. Extract the chemical class assignments produced by the workflow's computational classification step, which classifies candidates using CANOPUS and ClassyFire. Map each assignment onto the ClassyFire ChemOnt hierarchy and aggregate counts at the superclass, class, and subclass levels. Compute subclass abundances as percentages of annotated features and write subclass_abundances.csv, then render the three-level hierarchy as a sunburst chart (class_sunburst.png) with subclass segments sized by abundance.

## Related tools

- **CANOPUS** (Predicts the chemical classes of tentative candidates that are aggregated into the ChemOnt hierarchy for the sunburst) — https://bio.informatik.uni-jena.de/software/canopus/
- **ClassyFire** (Provides the ChemOnt superclass/class/subclass classification scheme onto which each assignment is mapped)
- **MAW-Py** (Produces the per-feature final candidate selection (via RDKit) whose class assignments feed the subclass abundance aggregation) — https://github.com/zmahnoor14/MAW
- **RDKit** (Used in MAW-Py for the final candidate selection that determines which annotations are summarized) — https://www.rdkit.org/

## Evaluation signals

- Reproduced subclass abundances match the reported values — Lipids 33.92%, Organic acids 17.02%, Benzenoids 13.94%, Phenylpropanoid and polyketides 11.34% — within rounding tolerance.
- All four subclass labels (Lipids, Organic acids, Benzenoids, Phenylpropanoid and polyketides) appear in the output.
- Every assignment in subclass_abundances.csv maps cleanly onto the ChemOnt superclass/class/subclass hierarchy with no unmapped classes.
- Sunburst rings correspond exactly to superclass, class, and subclass levels, with subclass segment sizes proportional to the computed abundances.
- Only one top-ranking annotation per feature is counted (duplicates from the candidates list excluded).

## Limitations

- SIRIUS/CANOPUS lipid class annotation can lead to false positives because of the complex nature of lipids, which can inflate the dominant Lipids subclass share.
- Features whose candidates cannot be computationally classified are excluded, so percentages are relative to annotated (classified) features only.
- Candidate ranking depends on MAW-defined thresholds (e.g. ≥0.5 matching score plus high cosine similarity), so threshold changes alter which annotations enter the sunburst.

## Evidence

- [readme] The classification of the tentative candidates from the input data are classified using CANOPUS: [evidence span withheld — anchor status: external_doc]
- [intro] MAW performs the five workflow stages ending in chemical compound classification: "MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication, (4) candidate selection, and (5) chemical compound classification"
- [intro] MAW prioritizes one final candidate per feature: "It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature."
- [abstract] MAW results are reproducible and traceable: "The results from MAW are also reproducible and traceable, compliant with the FAIR guidelines."
