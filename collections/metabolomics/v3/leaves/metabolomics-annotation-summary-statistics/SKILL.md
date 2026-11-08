---
name: metabolomics-annotation-summary-statistics
description: Use when you have annotation counts per MSI confidence level plus a total
  count of acquired/merged MS/MS spectra, and need to report annotation coverage and
  the unannotated residual.
when_to_use_negative:
- The goal is to assign new annotations to unannotated spectra (use SIRIUS/CSI:FingerID
  and NIST hybrid search for level 3 annotation) rather than to summarize coverage
  of existing annotations
- Input is raw LC-MS/MS signal data or a feature table rather than counts of merged
  spectra and their MSI-level annotations
- The annotation set includes MSI level 3 (in-silico) assignments but you intend to
  report only high-confidence coverage; mixing levels inflates the numerator inconsistently
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: manual expert review
  role: Source of the annotated-spectrum counts (Table S2 merged-spectrum annotation
    counts) that serve as the numerator and denominator for the coverage calculation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:8f369c70929793ff48bd746dd28972c884de76c972a8dc3afdfd4e972c7788a1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - manual expert review
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolomics-annotation-summary-statistics

## Summary

Computes the fraction of acquired MS/MS spectra covered by MSI confidence-level annotations (e.g. 753 MSI level 1+2 annotated spectra out of 3,894 merged urinary MS/MS spectra = 19.3%) and quantifies the residual unannotated fraction, exposing the size of the uncharted metabolome.

## When to use

Use when you have annotation counts per MSI confidence level plus a total count of acquired/merged MS/MS spectra, and need to report annotation coverage and the unannotated residual.

## When NOT to use

- The goal is to assign new annotations to unannotated spectra (use SIRIUS/CSI:FingerID and NIST hybrid search for level 3 annotation) rather than to summarize coverage of existing annotations
- Input is raw LC-MS/MS signal data or a feature table rather than counts of merged spectra and their MSI-level annotations
- The annotation set includes MSI level 3 (in-silico) assignments but you intend to report only high-confidence coverage; mixing levels inflates the numerator inconsistently

## Inputs

- Annotation counts per MSI confidence level from a merged-spectrum table (e.g. Table S2: 753 MSI level 1+2 annotated spectra)
- Total number of acquired/merged MS/MS spectra (e.g. 3,894)
- Per-level compound identification counts (175 MSI level 1; 578 MSI level 2)

## Outputs

- coverage_summary.csv: a small summary table with numerator, denominator, coverage fraction (~0.193), and unannotated fraction (~0.807)
- A validation statement confirming the computed coverage reproduces the reported percentage

## How to apply

Load the merged-spectrum annotation table (e.g. Table S2 counts) recording the number of spectra with MSI level 1 and level 2 annotations — in the urine HILIC/CSH Q Exactive HF study, 175 level 1 compounds plus 578 level 2 compounds (including CarniBlast acylcarnitine annotations) gave 753 annotated spectra out of 3,894 total merged MS/MS spectra. Compute coverage as annotated/total (753/3894 ≈ 0.193) and the unannotated residual as (total − annotated)/total (≈ 0.807) using simple arithmetic (pandas). The decision point is which MSI levels to include: summing only level 1+2 measures the high-confidence coverage, deliberately excluding level 3 in-silico (CSI:FingerID/NIST hybrid search) annotations. Write the numerator, denominator, coverage fraction, and residual fraction to a summary CSV. Validate by checking that the computed coverage reproduces the published figure (0.193 within rounding tolerance); a mismatch signals an error in numerator or denominator bookkeeping.

## Related tools

- **manual expert review** (Source of the annotated-spectrum counts (Table S2 merged-spectrum annotation counts) that serve as the numerator and denominator for the coverage calculation)

## Examples

```
import pandas as pd; cov = pd.DataFrame({'msi_level':['1+2'],'annotated':[753],'total':[3894],'coverage':[753/3894],'unannotated':[(3894-753)/3894]}); cov.to_csv('coverage_summary.csv', index=False)
```

## Evaluation signals

- Computed coverage fraction 753/3894 ≈ 0.193 must match the paper's reported 19.3% within rounding tolerance
- Coverage fraction + unannotated fraction must sum to 1 (0.193 + 0.807 = 1.0)
- Numerator (753) must be ≤ denominator (3,894) and consistent with per-level counts (175 MSI level 1 + 578 MSI level 2)
- coverage_summary.csv must contain the four expected fields: numerator, denominator, coverage fraction, residual fraction

## Limitations

- Combining MSI level 1+2 left more than 80% of the 3,894 acquired urinary MS/MS spectra unannotated — coverage statistics reflect high-confidence levels only, not the full annotation landscape
- The field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments, so MSI-level bookkeeping is manual and not FDR-controlled
- Top-ranked CSI:FingerID annotations yielded only 40% accuracy against MSI level 1 compounds, so extending the summary to in-silico level 3 counts would overstate reliable coverage

## Evidence
