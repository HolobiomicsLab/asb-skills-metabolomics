---
name: oncogene-condition-label-mapping
description: Use when text-extracted metabolite/enzyme findings from a metabolomics
  study must be assigned to specific oncogenic driver conditions (e.
when_to_use_negative:
- Input is raw mass spectrometry spectra or un-extracted metabolomics data — this
  skill maps already-extracted finding statements, it does not perform metabolite
  peak detection or quantitation
- The study lacks distinct oncogenic driver conditions or subgroup labels (no AKT1/MYC-type
  or equivalent condition structure to map findings onto)
- The goal is computing pathway enrichment itself (single-sample GSEA NES scores over
  KEGG sets) rather than labeling extracted findings with those pathway contexts
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0203
tools:
- name: KEGG
  role: Source of KEGG-annotated metabolite sets (>50 sets, Dataset S1) used to assign
    each finding row its pathway_context (glycolysis, glutaminolysis, glycine/choline/sarcosine
    pathway, etc.)
- name: pandas
  role: Parsing article Abstract/Results/Discussion text and assembling/validating
    the condition-labeled findings CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:c399cccb00a68c139a06c489a4e3cd8134af16adc4acf8c6ccf0b20405728690
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KEGG
  - pandas
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# oncogene-condition-label-mapping

## Summary

Maps metabolite/enzyme findings from oncogene-driver metabolomics studies onto condition labels exactly as reported in the source text (e.g., RWPE-AKT1 vs RWPE-EV, RWPE-MYC vs RWPE-EV, MPAKT vs Lo-MYC mice, phosphoAKT1-high/MYC-low vs phosphoAKT1-low/MYC-high human tumors), recording direction of change and KEGG pathway context per row. It exists because oncogenic driver effects are only interpretable when each finding is tied to the specific experimental condition and driver (AKT1 vs MYC) that produced it.

## When to use

Use when text-extracted metabolite/enzyme findings from a metabolomics study must be assigned to specific oncogenic driver conditions (e.g., RWPE-AKT1 vs RWPE-MYC cells, MPAKT vs Lo-MYC mice, phosphoAKT1-high/MYC-low vs phosphoAKT1-low/MYC-high human tumors) with direction of change and pathway context.

## When NOT to use

- Input is raw mass spectrometry spectra or un-extracted metabolomics data — this skill maps already-extracted finding statements, it does not perform metabolite peak detection or quantitation
- The study lacks distinct oncogenic driver conditions or subgroup labels (no AKT1/MYC-type or equivalent condition structure to map findings onto)
- The goal is computing pathway enrichment itself (single-sample GSEA NES scores over KEGG sets) rather than labeling extracted findings with those pathway contexts

## Inputs

- Full article text (Abstract, Results, Discussion) and figure/supplement captions (e.g., Fig. S1A, Fig. 1C, Fig. 4A)
- Extracted metabolite/enzyme finding statements with direction of change from mass spectrometry-based metabolite profiling of RWPE-1 engineered cells, transgenic mice, and human tumors
- KEGG metabolite set annotation (Dataset S1, >50 metabolite sets) used for pathway context

## Outputs

- CSV file with columns finding_type (metabolite|enzyme), entity, condition, model_system, direction_of_change, pathway_context, evidence_location
- Validation result: file parses as valid CSV with exactly 6 data rows covering the six enumerated findings and non-empty condition, direction_of_change, pathway_context fields

## How to apply

Parse the article's Abstract, Results, and Discussion (plus referenced figure/supplement captions such as Fig. S1A, Fig. 1C, Fig. 4A) with pandas to extract every metabolite or enzyme finding statement. Assign each finding the condition label exactly as reported — RWPE-AKT1 vs RWPE-EV, RWPE-MYC vs RWPE-EV, MPAKT mice, Lo-MYC mice, phosphoAKT1-high/MYC-low tumors, phosphoAKT1-low/MYC-high tumors, tumor vs normal prostate — never a paraphrase. Record the direction of change as increased/decreased/exclusive for each entity (e.g., lactate increased in RWPE-AKT1 and MPAKT; oleic, arachidonic and DHA acids higher in MYC-high tumors; sarcosine increased exclusively in Lo-MYC; glutaminase mRNA and protein induced by MYC; creatine increased in phosphoAKT1-high/MYC-low). Annotate each row with its KEGG-annotated pathway context (glycolysis, glycerophospholipid/glycerolipid metabolism, glutaminolysis, glycine/choline/sarcosine pathway, lysine metabolism), consistent with the >50 KEGG metabolite sets the paper tested by single-sample GSEA. Assemble rows into a CSV with columns finding_type (metabolite|enzyme), entity, condition, model_system, direction_of_change, pathway_context, evidence_location, and validate that the file parses as valid CSV with exactly 6 data rows and non-empty condition, direction_of_change, and pathway_context fields.

## Related tools

- **KEGG** (Source of KEGG-annotated metabolite sets (>50 sets, Dataset S1) used to assign each finding row its pathway_context (glycolysis, glutaminolysis, glycine/choline/sarcosine pathway, etc.))
- **pandas** (Parsing article Abstract/Results/Discussion text and assembling/validating the condition-labeled findings CSV)

## Examples

```
df.to_csv('oncogene_condition_findings.csv', columns=['finding_type','entity','condition','model_system','direction_of_change','pathway_context','evidence_location'])
```

## Evaluation signals

- Output file exists and parses as valid CSV (pd.read_csv succeeds) with exactly 6 data rows
- Every row has non-empty condition, direction_of_change, and pathway_context fields
- Condition strings match the article's own labels verbatim (RWPE-AKT1 vs RWPE-EV, RWPE-MYC vs RWPE-EV, MPAKT, Lo-MYC, phosphoAKT1-high/MYC-low, phosphoAKT1-low/MYC-high, tumor vs normal prostate)
- Direction-of-change values are consistent with the source text (e.g., lactate increased in RWPE-AKT1 and MPAKT; sarcosine exclusive to Lo-MYC; creatine increased in phosphoAKT1-high/MYC-low; oleic/arachidonic/DHA higher in MYC-high tumors)
- Each pathway_context corresponds to a KEGG-annotated metabolite set among the >50 sets used in the paper's single-sample GSEA

## Limitations

- Condition labels are limited to those explicitly reported in the article text; findings from conditions not enumerated (e.g., phosphoAKT1-high/MYC-high subgroup) cannot be unambiguously attributed to a single driver
- Pathway context granularity is bounded by the KEGG metabolite-set annotation (Dataset S1); metabolites outside these sets lack pathway_context support
- The validation contract is fixed at 6 rows for the six enumerated findings — additional or fewer findings from other sources will fail the row-count check unless the schema is re-scoped

## Evidence
