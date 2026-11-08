---
name: metabolomics-phenotype-association-testing
description: Use when an aligned peak-intensity matrix from an untargeted LC/HRMS
  study (e.g. IDSL.IPA or MS-DIAL output for 499 MTBLS1684 samples) is joined to sample-level
  phenotype metadata and you need to rank peaks by monotonic association strength
  with a significance decision at adjusted p < 0.05.
when_to_use_negative:
- Input is a single-file peak list or raw centroided spectra rather than an aligned
  peak x sample intensity matrix — peak-to-phenotype correlation requires the aligned
  data matrix after IPA_workflow processing
- The phenotype is a binary or multi-class label with no monotonic ordering — Spearman
  rank correlation is for continuous or ordered phenotypes; use a group-difference
  test instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: IDSL.IPA
  role: Produces the aligned, gap-filled peak-height matrix that is statistically
    filtered against the phenotype in this skill
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: R
  role: Language in which the Spearman correlation, FDR adjustment, and per-peak ranked
    table are computed
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: MS-DIAL
  role: Alternative peak-detection pipeline whose aligned matrix is processed identically
    to benchmark the count of significant phenotype-associated peaks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:c9df2355df3ce6f5ceceaf182345670fc4f255c3f9c7ba2439e9997c08bdf39b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - MS-DIAL
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolomics-phenotype-association-testing

## Summary

Tests aligned untargeted LC/HRMS peak-table intensities against a continuous clinical or experimental phenotype (e.g. birth weight) using Spearman rank correlation with BH/FDR multiple-testing correction, and compares the resulting significant-peak sets across alternative peak-detection pipelines such as IDSL.IPA and MS-DIAL.

## When to use

Use when an aligned peak-intensity matrix from an untargeted LC/HRMS study (e.g. IDSL.IPA or MS-DIAL output for 499 MTBLS1684 samples) is joined to sample-level phenotype metadata and you need to rank peaks by monotonic association strength with a significance decision at adjusted p < 0.05.

## When NOT to use

- Input is a single-file peak list or raw centroided spectra rather than an aligned peak x sample intensity matrix — peak-to-phenotype correlation requires the aligned data matrix after IPA_workflow processing
- The phenotype is a binary or multi-class label with no monotonic ordering — Spearman rank correlation is for continuous or ordered phenotypes; use a group-difference test instead

## Inputs

- IDSL.IPA aligned and gap-filled peak-height matrix (peak x sample) from untargeted LC/HRMS data
- Comparable aligned peak table from an alternative pipeline (e.g. MS-DIAL) for the same samples
- Sample-level phenotype metadata table keyed by sample ID (e.g. birth weight for MTBLS1684 cord-blood samples)

## Outputs

- Ranked per-peak association table (peak ID, Spearman rho, raw p-value, BH-adjusted p-value, significance flag at adjusted p < 0.05)
- Significant-peak counts per pipeline (e.g. IDSL.IPA-only, common, MS-DIAL-only) for cross-pipeline comparison

## How to apply

Join the IDSL.IPA aligned and gap-filled peak-height matrix to the phenotype metadata (e.g. birth weight for the 499 cord-blood samples of MTBLS1684) on sample IDs, then for each peak compute a Spearman rank correlation of peak height against the phenotype. Adjust raw p-values for multiple testing (Benjamini-Hochberg/FDR) across all peaks and flag peaks with adjusted p < 0.05 as significant; sort the table by rho or p-value. Repeat the identical procedure on a comparison matrix (e.g. MS-DIAL aligned peaks) and count significant peaks from each pipeline to quantify enrichment. Rationale: the IDSL.IPA matrix yielded 2.9 times more significant birth-weight-associated peaks than MS-DIAL (213 IDSL.IPA-only, 401 common, 37 MS-DIAL-only), and the article stresses that IDSL.IPA 'generated aligned data matrices need to be further processed for identifying adducts and fragments, filtering for statistics and annotating molecular formulas', so this statistical filtering step is the downstream consumer of the peak matrix.

## Related tools

- **IDSL.IPA** (Produces the aligned, gap-filled peak-height matrix that is statistically filtered against the phenotype in this skill) — https://cran.r-project.org/package=IDSL.IPA
- **R** (Language in which the Spearman correlation, FDR adjustment, and per-peak ranked table are computed) — https://cran.r-project.org/package=IDSL.IPA
- **MS-DIAL** (Alternative peak-detection pipeline whose aligned matrix is processed identically to benchmark the count of significant phenotype-associated peaks)

## Examples

```
cor.test(x = peak_heights, y = birth_weight, method = 'spearman'); p_adj <- p.adjust(p_raw, method = 'BH')
```

## Evaluation signals

- All peaks retained in the final table have BH-adjusted p-value < 0.05
- Spearman rho values lie in [-1, 1] and every peak has a finite raw and adjusted p-value
- Significant-peak counts decompose cleanly into IDSL.IPA-only, common, and MS-DIAL-only sets that reproduce the reported ratio (IDSL.IPA significant peaks ≈ 2.9x MS-DIAL significant peaks; 213 vs 401 vs 37 for MTBLS1684 birth weight)
- Every row of the ranked table maps to a peak present in the source aligned matrix (peak IDs match)
- Sample size used in each correlation equals the number of successfully joined phenotype rows (499 for MTBLS1684)

## Limitations

- IDSL.IPA aligned data matrices require additional downstream processing for adducts, fragments, statistics filtering, and molecular formula annotation before biological interpretation
- A majority of low-abundance EICs that passed the S/N threshold were still rejected, so some biologically relevant low-abundance peaks may be absent from the aligned matrix and untestable against the phenotype
- Spearman correlation only captures monotonic associations; the article reports no direct validation of this correlation workflow against a reference standard

## Evidence

- [results] Spearman correlation between phenotype and peak intensity: [evidence span withheld — anchor status: not_found]
- [results] Low-abundance rejected EICs: [evidence span withheld — anchor status: not_found]
- [other] Reproduce 2.9x enrichment card: [evidence span withheld — anchor status: not_found]
