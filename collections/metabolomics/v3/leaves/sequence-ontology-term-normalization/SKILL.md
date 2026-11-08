---
name: sequence-ontology-term-normalization
description: Use when predicting coding impacts from multiple annotation tools (e.g.,
  ANNOVAR, SnpEff, VEP) whose native labels are not directly comparable, and you need
  concordance against a clinical benchmark like ClinVar.
when_to_use_negative:
- The analysis concerns HGVS nomenclature (HGVSc/HGVSp) strings rather than functional
  coding-impact classes — use a dedicated HGVS normalization/parsing approach instead
- Only a single annotation tool is used with no cross-tool or ClinVar comparison,
  so vocabulary mapping adds no comparability
- Input variants are not from a genomic/transcript annotation context (e.g., metabolomics
  or proteomics identifiers) that maps to Sequence Ontology terms
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ANNOVAR
  role: One of the three annotation tools whose predicted coding-impact terms are
    normalized and compared to ClinVar
- name: SnpEff
  role: One of the three annotation tools whose predicted coding-impact terms are
    normalized and compared to ClinVar
- name: Variant Effect Predictor (VEP)
  role: One of the three annotation tools whose predicted coding-impact terms are
    normalized and compared to ClinVar; can be run with --per_gene and --coding_only
    to streamline relevant transcripts
- name: ClinVar
  role: Clinical benchmark database providing two-star variants with reported impacts
    for concordance evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:787b0c1b91eda0304e518e01b038bf6ba857404d74ec701b434f8a32d84ef525
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - VEP
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# sequence-ontology-term-normalization

## Summary

Normalizes tool-specific predicted coding-impact terms from ANNOVAR, SnpEff, and VEP into a common Sequence Ontology vocabulary so their variant annotations can be compared against each other and against ClinVar-reported impacts.

## When to use

Use when predicting coding impacts from multiple annotation tools (e.g., ANNOVAR, SnpEff, VEP) whose native labels are not directly comparable, and you need concordance against a clinical benchmark like ClinVar.

## When NOT to use

- The analysis concerns HGVS nomenclature (HGVSc/HGVSp) strings rather than functional coding-impact classes — use a dedicated HGVS normalization/parsing approach instead
- Only a single annotation tool is used with no cross-tool or ClinVar comparison, so vocabulary mapping adds no comparability
- Input variants are not from a genomic/transcript annotation context (e.g., metabolomics or proteomics identifiers) that maps to Sequence Ontology terms

## Inputs

- Same VCF variant set annotated independently by ANNOVAR, SnpEff, and VEP
- ClinVar high-review-status (two-star) benchmark variants with reported coding impact
- Tool-specific predicted coding-impact term labels (e.g., stop-gained, splice-acceptor)

## Outputs

- Normalized coding-impact calls in a common Sequence Ontology vocabulary per tool per variant
- Per-tool agreement/concordance table (e.g., concordance_table.csv) with overall and category-specific (LoF) fractions

## How to apply

Annotate the same variant set (e.g., 164,549 two-star ClinVar variants) with ANNOVAR, SnpEff, and VEP using identical reference genomes and transcript annotations across all three tools. Map each tool's predicted coding-impact terms into a shared Sequence Ontology vocabulary so tool-specific labels become comparable. When resolving ambiguous term mappings, prioritize severe/high-impact variant classes, since they are often more clinically relevant. Then compare each tool's normalized call against the ClinVar-reported impact per variant and compute per-tool agreement rates (this study reported ~85.58% three-tool coding-impact concordance, with LoF category agreement of 0.983 ANNOVAR, 0.975 SnpEff, 0.972 VEP over 31,640 LoF variants). Write the per-tool agreement values to a table such as concordance_table.csv for validation.

## Related tools

- **ANNOVAR** (One of the three annotation tools whose predicted coding-impact terms are normalized and compared to ClinVar)
- **SnpEff** (One of the three annotation tools whose predicted coding-impact terms are normalized and compared to ClinVar)
- **Variant Effect Predictor (VEP)** (One of the three annotation tools whose predicted coding-impact terms are normalized and compared to ClinVar; can be run with --per_gene and --coding_only to streamline relevant transcripts)
- **ClinVar** (Clinical benchmark database providing two-star variants with reported impacts for concordance evaluation)

## Examples

```
import pandas as pd; pd.DataFrame(rows, columns=['tool','overall_agreement','lof_concordance']).to_csv('concordance_table.csv', index=False)
```

## Evaluation signals

- Per-tool overall coding-impact agreement should be near the reported 0.990 (ANNOVAR), 0.992 (SnpEff), and 0.994 (VEP) levels, consistent with the ~85.58% three-tool coding-impact concordance
- Restricted to the Loss-of-Function category, per-tool concordance should match 0.983 (ANNOVAR), 0.975 (SnpEff), and 0.972 (VEP) over 31,640 LoF variants within rounding tolerance
- The LoF category should emerge as the lowest coding-impact agreement category across all three tools
- Values written to concordance_table.csv must match reported rates within rounding tolerance
- Every mapped term must resolve to a valid Sequence Ontology concept; ambiguous mappings should be resolved toward severe/high-impact classes

## Limitations

- Discrepancies with ClinVar persisted even after restricting transcripts to those selected by ClinVar
- Some HGVS syntax in ClinVar does not adhere to HGVS recommendations, and there are inconsistencies between ClinVar web pages and its VCF, which can perturb benchmark comparisons
- Different MANE selections still impact the final annotation, so normalized calls depend on the transcript set chosen; no specific software versions of ANNOVAR, SnpEff, or VEP were reported

## Evidence

- [abstract] Three-tool coding-impact concordance level: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [discussion] Identical references across tools: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
- [intro] Tool annotation mechanism: "Annotation tools like ANNOVAR14, SnpEff15, and VEP16 utilize RNA transcripts to annotate variants by using overlapping genomic characteristics."
- [discussion] High-impact prioritization for ambiguous mappings: "Prioritize high-impact variants, as they are often more clinically relevant."
- [discussion] Transcript restriction does not eliminate discrepancies: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
