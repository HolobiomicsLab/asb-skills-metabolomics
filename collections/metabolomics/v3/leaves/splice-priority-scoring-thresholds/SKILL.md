---
name: splice-priority-scoring-thresholds
description: Decision support for assigning the PP3 evidence rule during ACMG 2015
  classification when SpliceAI and/or CADD scores are available for variants, and
  for handling its suppression by PVS1.
when_to_use_negative:
- The variant already qualifies for PVS1 via LoF coding impact — PP3 must be disabled
  rather than scored.
- No CADD or SpliceAI scores are available for the variants, so the >25.3 / >0.5 thresholds
  cannot be evaluated.
- The analysis targets ACMG rules other than splice-priority evidence (e.g., PS4,
  PM3, PP4, BP2, BP5, which the GDK engine excludes).
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3574
tools:
- name: ANNOVAR
  role: Generates predicted coding impacts and LoF calls used to gate PVS1 and evaluate
    PP3 thresholds
- name: SnpEff
  role: Alternative annotator producing predicted coding impacts and LoF calls feeding
    the PP3/PVS1 logic
- name: Variant Effect Predictor (VEP)
  role: Alternative annotator producing predicted coding impacts and LoF calls feeding
    the PP3/PVS1 logic
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:6aee614fc4ae3ac91d2814a793b1994286a36a50cd27be2a054c223c67584797
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3574
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# splice-priority-scoring-thresholds

## Summary

Implements the ACMG PP3 (splice-priority) rule with concrete scoring thresholds—CADD > 25.3 or SpliceAI > 0.5—as used by the in-house GDK platform's ACMG rule engine, including the interplay rule that PP3 is disabled whenever PVS1 is triggered by a loss-of-function coding impact.

## When to use

Decision support for assigning the PP3 evidence rule during ACMG 2015 classification when SpliceAI and/or CADD scores are available for variants, and for handling its suppression by PVS1.

## When NOT to use

- The variant already qualifies for PVS1 via LoF coding impact — PP3 must be disabled rather than scored.
- No CADD or SpliceAI scores are available for the variants, so the >25.3 / >0.5 thresholds cannot be evaluated.
- The analysis targets ACMG rules other than splice-priority evidence (e.g., PS4, PM3, PP4, BP2, BP5, which the GDK engine excludes).

## Inputs

- Variant set with ANNOVAR/SnpEff/VEP predicted coding impacts (e.g., 164,549 two-star ClinVar variants)
- Per-variant CADD scores
- Per-variant SpliceAI scores
- Loss-of-function (LoF) coding-impact calls from the annotation tools

## Outputs

- Per-variant PP3 assignment (triggered/disabled) with the qualifying score and threshold
- Per-variant PVS1 status used to gate PP3
- classification_changes.csv with per-variant rule assignments and ACMG classification-change summary

## How to apply

Annotate each variant with predicted coding impacts from tools such as ANNOVAR, SnpEff, and VEP, then evaluate splice-priority evidence using the GDK platform's thresholds: trigger PP3 when CADD > 25.3 OR SpliceAI > 0.5. Before assigning PP3, first determine whether the variant has a loss-of-function (LoF) coding impact that qualifies for PVS1; if PVS1 is assigned, PP3 must be disabled for that variant to avoid redundant pathogenic evidence stacking. Run the threshold logic across the full variant set (e.g., the 164,549 two-star ClinVar variants) and record per-variant rule assignments so downstream ACMG combination and classification changes can be audited. The rationale is that annotation-tool disagreement in predicted coding impacts—especially in the Loss-of-Function category—directly changes whether PVS1 fires, which in turn changes PP3 application and the final class.

## Related tools

- **ANNOVAR** (Generates predicted coding impacts and LoF calls used to gate PVS1 and evaluate PP3 thresholds)
- **SnpEff** (Alternative annotator producing predicted coding impacts and LoF calls feeding the PP3/PVS1 logic)
- **Variant Effect Predictor (VEP)** (Alternative annotator producing predicted coding impacts and LoF calls feeding the PP3/PVS1 logic)

## Examples

```
python acmg_engine.py --variants clinvar_2star_annotated.csv --pp3-cadd 25.3 --pp3-spliceai 0.5 --disable-pp3-on-pvs1 --out classification_changes.csv
```

## Evaluation signals

- PP3 is triggered exactly when CADD > 25.3 OR SpliceAI > 0.5, and never on variants with lower scores.
- No variant receives both PVS1 and PP3 — PP3 must be disabled wherever PVS1 is assigned.
- Reproducing the paper's PLP downgrade rates (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%) driven by incorrect PVS1 interpretations serves as an end-to-end validation of the rule engine.
- Per-variant rule assignments are written to classification_changes.csv and reconcile with the classification-change summary counts.

## Limitations

- Substantial discrepancies in the LoF category across ANNOVAR, SnpEff, and VEP mean PVS1—and therefore PP3 gating—flips between tools, downgrading PLP variants and risking false negatives.
- Concordance of predicted coding impacts across tools is only ~85.58% (58.52% for HGVSc, 84.04% for HGVSp), so PP3/PVS1 outcomes depend heavily on transcript and annotator choice.
- The threshold set (CADD > 25.3, SpliceAI > 0.5, PP3-off-under-PVS1) reflects the GDK platform's in-house 2015-guideline implementation, not a community-validated standard.

## Evidence

- [other] GDK PP3 thresholds and PVS1 gating: [evidence span withheld — anchor status: not_found]
- [abstract] PLP downgrade rates from incorrect PVS1: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)"
- [abstract] LoF category discrepancies: "The substantial discrepancies were noted in the Loss-of-Function (LoF) category."
- [abstract] coding impact concordance: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
