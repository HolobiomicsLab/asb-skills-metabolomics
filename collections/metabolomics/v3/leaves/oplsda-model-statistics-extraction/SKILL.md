---
name: oplsda-model-statistics-extraction
description: Use when an article reports OPLS-DA (or PCA/OPLS supervised modeling)
  separation statistics as inline p-values (e.g. 'p = 2.8*10−6', 'p = 0.0031') scattered
  across Results text, figure captions, and tables that must be compiled into a single
  validated significance table.
when_to_use_negative:
- The article reports only PCA (unsupervised) distributions without any supervised-model
  p-values — there is nothing to extract.
- Model statistics are already published as a machine-readable table or CSV supplement;
  re-extraction risks transcription errors.
- The task is to recompute OPLS-DA p-values from raw GCxGC-TOFMS metabolite data (permutation
  testing) rather than extract reported values.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: GCxGC-TOFMS
  role: Instrument platform whose plasma metabolite profiles (195 metabolites) underlie
    the OPLS-DA models whose statistics are extracted; the skill documents provenance
    of the derived significance values
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:15462687e49b3150465324b4b1c2abddd7a600650c1ad0f7b987045463976cb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# oplsda-model-statistics-extraction

## Summary

Extracts and normalizes OPLS-DA model significance statistics (p-values for component-wise class separations) from metabolomics articles — here, GCxGC-TOFMS plasma profiles of S. Typhi / S. Paratyphi A carriage vs non-carriage controls — into a structured table (oplsda_pvalues.csv) with model, comparison, p_value, and source location.

## When to use

Use when an article reports OPLS-DA (or PCA/OPLS supervised modeling) separation statistics as inline p-values (e.g. 'p = 2.8*10−6', 'p = 0.0031') scattered across Results text, figure captions, and tables that must be compiled into a single validated significance table.

## When NOT to use

- The article reports only PCA (unsupervised) distributions without any supervised-model p-values — there is nothing to extract.
- Model statistics are already published as a machine-readable table or CSV supplement; re-extraction risks transcription errors.
- The task is to recompute OPLS-DA p-values from raw GCxGC-TOFMS metabolite data (permutation testing) rather than extract reported values.

## Inputs

- Full-text article (PMC XML or printable PDF) of a metabolomics paper reporting OPLS-DA statistics, e.g. DOI 10.1371/journal.pntd.0006215
- Figure and supplementary figure captions describing OPLS-DA/PCA scores plots (Fig 1, S1 Fig)
- Model statistics table (Table 1) listing per-model p-values

## Outputs

- oplsda_pvalues.csv with columns: model, comparison, p_value (normalized float), source_location (Results paragraph / Fig 1 / Table 1)

## How to apply

Parse the article's full text (PMC-deposited or printable PDF, e.g. DOI 10.1371/journal.pntd.0006215), focusing on the Results section, the main figures (e.g. Fig 1 OPLS-DA scores plot), supplementary figure captions (S1 Fig PCA/OPLS-DA panels), and any model-statistics table (Table 1). Extract each reported p-value with its model description and class comparison: here, the three-class OPLS-DA separation of S. Typhi carriage from S. Paratyphi A carriage along the second component (p = 0.0031), the two-class combined-carriage vs non-carriage model (p = 2.8*10−6), pairwise carriage vs control models (p = 3.1*10−5 and p = 3.7*10−4), and the direct S. Typhi vs S. Paratyphi A model (p = 0.07). Normalize asterisk scientific notation (2.8*10−6 → 2.8e-06) to standard float representation. Write one record per model to oplsda_pvalues.csv with columns model, comparison, p_value, and source_location. Validate by cross-checking every value against its verbatim text span and confirming all values match Table 1 and the Fig 1 / S1 Fig descriptions.

## Related tools

- **GCxGC-TOFMS** (Instrument platform whose plasma metabolite profiles (195 metabolites) underlie the OPLS-DA models whose statistics are extracted; the skill documents provenance of the derived significance values)

## Examples

```
python extract_oplsda_stats.py --input pntd.0006215.xml --output oplsda_pvalues.csv
```

## Evaluation signals

- All five extracted p-values (0.0031, 2.8e-06, 3.1e-05, 3.7e-04, 0.07) match Table 1 and the Fig 1 / S1 Fig descriptions verbatim.
- Every p_value is stored as a standard float (e.g. 2.8e-06), not as an asterisk-notation string like '2.8*10−6'.
- Each record's source_location points to a specific Results paragraph, Fig 1, S1 Fig, or Table 1 entry.
- Each record's comparison string identifies the exact class contrast (e.g. 'S. Typhi carriage vs S. Paratyphi A carriage, second component').
- CSV parses cleanly with exactly the columns model, comparison, p_value, source_location and one row per reported model.

## Limitations

- Reported p-values are those of the original CV-ANOVA / model testing and cannot be recomputed or audited from the extraction alone; the original small sample size limits their generalizability.
- Scientific-notation variants (asterisk form '2.8*10−6', Unicode minus signs) must be normalized; unhandled variants will break float parsing.
- Model statistics may be split across Results text, figure captions, and tables — omitting any location yields an incomplete table (the direct S. Typhi vs S. Paratyphi A p = 0.07 appears separately from the carriage-vs-control values).

## Evidence

- [supplementary] three-class second-component separation: "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
- [abstract] abstract discrimination claim: "We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples."
- [supplementary] instrument and data provenance: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [discussion] sample size limitation: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues"
