---
name: fdr-target-decoy-cutoff-validation
description: Use when a scored target-decoy TG annotation list (e.g. ResultsTG from
  ScoreLipids) must be filtered to an empirical FDR (default 0.1) and validated against
  a standards file.
when_to_use_negative:
- The annotation list has not yet been scored against the decoy database (no target-decoy
  merge from ScoreLipids), so no FDR estimate is meaningful.
- The lipid class is not triacylglycerol — DIATAGeR and its FDRCutoff step are specific
  to TAG only.
- No standards file exists and the goal is a validated recovery rate rather than a
  plain empirical-FDR-filtered list.
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DIATAGeR
  role: R package whose FDRCutoff function implements the empirical FDR filtering
    of merged target/decoy TG annotations and standards-based validation
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: ScoreLipids
  role: Upstream DIATAGeR step that merges target and decoy annotations and scores
    them with the machine learning algorithm before FDRCutoff
  repo: https://github.com/Velenosi-Lab/DIATAGeR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.aca.2025.344698
    title: 'DIATAGeR: Triacylglycerol annotation of data-independent acquisition based
      lipidomics'
  source_content_hash: sha256:8662177e88bfb232aea3601369496e31f7c43b1313e39a52bdae377c17974b02
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Velenosi-Lab/DIATAGeR
  tools:
  - DIATAGeR
  - DecoyIdentifier
  - FDRCutoff
  - LipidIdentifier
  - RemoveIsotopes
  - ScoreLipids
  - SpectraImport
  - allDecoys
  - caret
  - devtools
  - measureFileImport
  - pROC
  - tidyverse
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 30
---

# fdr-target-decoy-cutoff-validation

## Summary

Applies an empirical FDR cutoff to merged target/decoy triacylglycerol annotations from DIATAGeR, retaining TG species below the predetermined FDR and, when a standard list is supplied, reporting how many standards pass and the FDR required to recover all of them.

## When to use

Use when a scored target-decoy TG annotation list (e.g. ResultsTG from ScoreLipids) must be filtered to an empirical FDR (default 0.1) and validated against a standards file.

## When NOT to use

- The annotation list has not yet been scored against the decoy database (no target-decoy merge from ScoreLipids), so no FDR estimate is meaningful.
- The lipid class is not triacylglycerol — DIATAGeR and its FDRCutoff step are specific to TAG only.
- No standards file exists and the goal is a validated recovery rate rather than a plain empirical-FDR-filtered list.

## Inputs

- Combined scored target and decoy TG annotation list from ScoreLipids (e.g. ResultsTG)
- Standards CSV file listing known TG standards (Standards.csv; Standards_File_Format_example.csv available via MetaboLights MTBLS11927)

## Outputs

- FDR-filtered TG annotation list (ResultsTG_FDR) saved as an RDS file (ResultsTG_FDR.RDS)
- Per-iteration statistics: average number of species, molecular species and AUC values
- When a standards file is supplied: number of standards found and the FDR required to find all standards

## How to apply

Run FDRCutoff on the combined target/decoy scored annotations produced by ScoreLipids, passing standard.file = "Standards.csv", iteration = 10, cutoff = 0.1 and rttol = 10. The function iterates up to 10 times, reporting the average number of species, molecular species and AUC values across iterations, and outputs the file containing the highest number of molecular species. With a standards file provided, it reports the number of standards found and the FDR required to find all standards; retention-time matching between the standards file and the results uses rttol = 10 seconds. Save the filtered result as ResultsTG_FDR.RDS via saveRDS. In the reported validation on liver and plasma samples from MASH and healthy control mice, all 9 TG standards passed at FDR below 0.1 in both datasets, which is the expected outcome when the cutoff and inputs are correct.

## Related tools

- **DIATAGeR** (R package whose FDRCutoff function implements the empirical FDR filtering of merged target/decoy TG annotations and standards-based validation) — https://github.com/Velenosi-Lab/DIATAGeR
- **ScoreLipids** (Upstream DIATAGeR step that merges target and decoy annotations and scores them with the machine learning algorithm before FDRCutoff) — https://github.com/Velenosi-Lab/DIATAGeR

## Examples

```
ResultsTG_FDR <- FDRCutoff(identified.lipids = ResultsTG, standard.file = "Standards.csv", iteration = 10, cutoff = 0.1, rttol = 10); saveRDS(ResultsTG_FDR, file = "ResultsTG_FDR.RDS")
```

## Evaluation signals

- All supplied TG standards are annotated at FDR below 0.1 (9 of 9 standards in the reported liver and plasma validation datasets).
- The average number of species, molecular species and AUC values are reported across the 10 iterations, and the output file chosen contains the highest number of molecular species.
- ResultsTG_FDR.RDS is produced via saveRDS from the FDRCutoff output.
- Retention-time matching between standards and results succeeds within the rttol = 10 s tolerance.
- The scored input list contains both target and decoy annotations merged by ScoreLipids, so decoy-based empirical FDR is actually computed.

## Limitations

- The standards file is optional; without it the function only reports average species/molecular species counts and AUC, with no standards-recovery validation.
- The empirical FDR cutoff default is 0.1 and must be re-tuned per dataset; the reported AUC and standards-recovery figures were obtained with cutoff = 0.1 and iteration = 10.
- FDRCutoff is limited to triacylglycerols (lipid = "TAG"), consistent with DIATAGeR's overall scope.
- FDR estimates depend on the decoy annotation quality generated upstream by LipidIdentifier/ScoreLipids; errors there propagate to the cutoff.

## Evidence

- [other] In the validation study of liver and plasma samples from MASH and healthy control mice, DIATAGeR annotated all 9 TG standards at an FDR below 0.1 in both datasets: [evidence span withheld — anchor status: not_found]
