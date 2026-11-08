---
name: p-value-verification-against-publication-text
description: Use when extracting reported model significance values (e.g. OPLS-DA
  p-values) from a metabolomics paper's text/figures and validating each against its
  verbatim source span before writing to a results table.
when_to_use_negative:
- Source text is a data file or figure image with no accompanying reported statistics
  — nothing to verify against
- Values are being computed from raw GCxGC-TOFMS data rather than extracted from published
  text
- The goal is reproducing the OPLS-DA models themselves (sample classification / ROC
  analysis), not verifying reported statistics
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: GCxGC-TOFMS
  role: Source of the plasma metabolite data whose supervised pattern-recognition
    model statistics (OPLS-DA p-values) are being extracted and verified
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:ad9e04cb84d391353f9a004e0907878eb13981fe2a6301a5fe9531cb757e7c73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# p-value-verification-against-publication-text

## Summary

Verifies extracted OPLS-DA model significance p-values (e.g. 2.8*10−6, 0.0031, 0.07) from the typhoid carriage metabolomics study by cross-checking each number against its verbatim text span in the article, Fig 1/S1 Fig, and Table 1, and normalizing asterisk scientific notation to floats.

## When to use

Use when extracting reported model significance values (e.g. OPLS-DA p-values) from a metabolomics paper's text/figures and validating each against its verbatim source span before writing to a results table.

## When NOT to use

- Source text is a data file or figure image with no accompanying reported statistics — nothing to verify against
- Values are being computed from raw GCxGC-TOFMS data rather than extracted from published text
- The goal is reproducing the OPLS-DA models themselves (sample classification / ROC analysis), not verifying reported statistics

## Inputs

- PMC full-text article (DOI 10.1371/journal.pntd.0006215) including Results text, Fig 1, S1 Fig caption, and Table 1
- Extracted p-value strings with model description, class comparison, and source location

## Outputs

- oplsda_pvalues.csv with columns model, comparison, p_value, source_location
- Normalized float p-values (e.g. 2.8*10−6 → 2.8e-06) each cross-checked against verbatim article spans

## How to apply

Parse the PMC-deposited full text (DOI 10.1371/journal.pntd.0006215), focusing on the Results section, Fig 1, the S1 Fig caption, and Table 1 where OPLS-DA statistics appear. Extract each p-value with its model description and class comparison — three-class S. Typhi vs S. Paratyphi A separation along the second component (p = 0.0031), two-class combined-carriage vs controls (p = 2.8*10−6), pairwise S. Typhi vs controls (p = 3.1*10−5), S. Paratyphi A vs controls (p = 3.7*10−4), and direct S. Typhi vs S. Paratyphi A (p = 0.07). Normalize asterisk notation such as 2.8*10−6 to float form (2.8e-06). Write records to oplsda_pvalues.csv with columns model, comparison, p_value, source_location; validate by matching every value to its verbatim parenthetical span like '(p = 2.8*10−6)' and confirming consistency with Table 1 and Fig 1/S1 Fig descriptions.

## Related tools

- **GCxGC-TOFMS** (Source of the plasma metabolite data whose supervised pattern-recognition model statistics (OPLS-DA p-values) are being extracted and verified)

## Examples

```
import re; p='2.8*10−6'; float_p=re.sub(r'\*10([−-]?)','e\\1',p).replace('−','-'); print(float(p))
```

## Evaluation signals

- All five extracted p-values (0.0031, 2.8e-06, 3.1e-05, 3.7e-04, 0.07) match their verbatim parenthetical spans in the article
- Every p_value row in oplsda_pvalues.csv carries a non-empty source_location traceable to Results text, Fig 1, S1 Fig, or Table 1
- Asterisk scientific notation is normalized to float form (2.8*10−6 → 2.8e-06) without altering significant digits
- Cross-check confirms consistency between Results-paragraph values, Fig 1/S1 Fig descriptions, and Table 1 values
- model and comparison fields match the article's own class-comparison terminology (e.g. 'non-carriage controls vs Salmonella carriage samples')

## Limitations

- Small sample size makes the reported model statistics fragile — p-values were obtained in a study whose major limitation was small sample size with no independent validation cohort
- Potential confounding from underlying gallbladder conditions means the significance of carriage separations may not generalize to healthy populations
- OCR/formatting artifacts in asterisk notation (e.g. 10−6 with unicode minus) must be handled when normalizing to floats

## Evidence

- [supplementary] the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031): "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
- [full_text] The metabolite profiles from the Salmonella carriage samples were significantly divergent from the profiles of the non-carriage samples in the two-class OPLS-DA model (p = 2.8*10−6).: [evidence span withheld — anchor status: not_found]
- [discussion] The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella"
- [abstract] We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples.: "We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples."
