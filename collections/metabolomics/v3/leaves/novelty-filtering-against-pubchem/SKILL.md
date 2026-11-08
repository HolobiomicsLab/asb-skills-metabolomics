---
name: novelty-filtering-against-pubchem
description: Use when COSMIC or CSI:FingerID annotations must be reduced to compounds
  never reported before, e.g. before ranking 'truly novel' bile acid conjugates or
  HMDB-absent metabolites.
when_to_use_negative:
- When the goal is de-replication against known compounds rather than discovery of
  novel structures — filtering against PubChem removes exactly the hits a library
  search would report
- When annotations have not yet been scored by COSMIC (confidence scores and q-values
  are needed to rank and filter novel candidates)
- When stereochemistry resolution is required — only two-dimensional structures are
  considered by the annotation workflow
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: COSMIC
  role: Produces the confidence-scored structure annotations that are ordered and
    then novelty-filtered against PubChem
  repo: github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Processes the 278 mzML LC–MS/MS runs of MSV000082973 (feature detection, spectrum
    merging, formula ranking) upstream of annotation and novelty filtering
  repo: github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates (default parameters, 40-eV fragmentation) that
    are subsequently cross-referenced against PubChem
  repo: github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:a7837da076cc27a83c090ad6b79c041fca8a2d1b57f24f18b47012f5af4857b4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Novelty filtering against PubChem

## Summary

Cross-references structure annotations produced by COSMIC/CSI:FingerID against PubChem and discards every candidate already present in the database, retaining only 'truly novel' structures absent from known chemical space.

## When to use

Use when COSMIC or CSI:FingerID annotations must be reduced to compounds never reported before, e.g. before ranking 'truly novel' bile acid conjugates or HMDB-absent metabolites.

## When NOT to use

- When the goal is de-replication against known compounds rather than discovery of novel structures — filtering against PubChem removes exactly the hits a library search would report
- When annotations have not yet been scored by COSMIC (confidence scores and q-values are needed to rank and filter novel candidates)
- When stereochemistry resolution is required — only two-dimensional structures are considered by the annotation workflow

## Inputs

- Ranked structure annotation table from COSMIC (candidate structures, confidence scores, estimated q-values)
- PubChem structure database snapshot for cross-referencing
- Optional: combinatorially generated bile acid structure database used as the annotation search space

## Outputs

- novel_bile_acid_annotations.tsv — ranked table of top-12 'truly novel' structures with COSMIC confidence scores and estimated q-values (FDR threshold 0.64 ≈ 10%)
- List of discarded annotations whose structures matched PubChem entries

## How to apply

First run the full SIRIUS 4 pipeline (10 ppm mass accuracy, feature detection, agglomerative clustering-based spectrum merging, de novo formula ranking, CSI:FingerID candidate ranking at 40 eV) and compute COSMIC confidence scores, since novelty filtering only makes sense over ranked annotations. Order all annotations by COSMIC confidence score descending, then cross-reference each annotated candidate structure against PubChem and remove any structure present there, keeping only 'truly novel' hits. For the mice fecal bile acid study, extract the top-12 novel annotations and record each structure's confidence score and estimated q-value, using the 0.64 score threshold (≈10% FDR) as the reliability cutoff. Validate the survivors manually by fragmentation-spectrum interpretation, since the confidence score itself is not a probability of correctness. Note the same novelty logic underlies the parallel HMDB filter in which the authors 'excluded all hits with structures in the HMDB structure database'.

## Related tools

- **COSMIC** (Produces the confidence-scored structure annotations that are ordered and then novelty-filtered against PubChem) — github.com/sirius-ms/sirius
- **SIRIUS 4** (Processes the 278 mzML LC–MS/MS runs of MSV000082973 (feature detection, spectrum merging, formula ranking) upstream of annotation and novelty filtering) — github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates (default parameters, 40-eV fragmentation) that are subsequently cross-referenced against PubChem) — github.com/sirius-ms/sirius

## Evaluation signals

- Phe-CDCA and Trp-CDCA appear among the top-12 hits with COSMIC confidence scores above the 0.64 threshold
- All retained novel annotations have estimated q-values below 0.1 (≈10% FDR)
- No structure in the retained table matches any PubChem entry (novelty check passes)
- Manual interpretation of fragmentation spectra confirms the majority of top-12 annotations (>90% correct in the published evaluation)
- Ranked output is sorted by COSMIC confidence score in descending order before filtering

## Limitations

- The confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- The 0.64 threshold is practical guidance only and gives no guarantee of FDR in either direction, especially for biological datasets with no ground truth
- Small structure modifications can be hard or impossible to tell apart using MS/MS data alone, so two 'novel' candidates may differ only subtly
- Establishing stereochemistry from fragmentation spectra is beyond automated search engines; only the two-dimensional structure is annotated

## Evidence

- [methods] After searching PubChem to discard known structures, the top 12 best-scoring 'truly novel' bile acid conjugate structures: [evidence span withheld — anchor status: not_found]
- [discussion] we excluded all hits with structures in the HMDB structure database: "we excluded all hits with structures in the HMDB structure database"
- [discussion] more than 90% of the annotations for the top 12 bile acid conjugates suggested by COSMIC turned out to be correct.: "more than 90% of the annotations for the top 12 bile acid conjugates suggested by COSMIC turned out to be correct."
- [discussion] We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites: [evidence span withheld — anchor status: external_doc]
