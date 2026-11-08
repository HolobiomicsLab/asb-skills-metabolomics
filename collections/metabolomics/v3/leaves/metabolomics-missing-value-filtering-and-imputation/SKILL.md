---
name: metabolomics-missing-value-filtering-and-imputation
description: Use when raw metabolomics peak-intensity tables (e.g., TEDDY ST001386/ST001636
  from Metabolomics Workbench) contain missing values and must be made complete before
  building DNEA partial correlation networks or comparing against marginal-correlation
  tools like MetaBox2.
when_to_use_negative:
- Data has already been normalized/log-transformed by another pipeline — re-imputing
  median values on transformed data is not the paper's recipe (imputation is done
  on raw peak intensities before DNEA's own log-transform and auto-scaling)
- The analysis is pathway enrichment (e.g., FELLA/KEGG mapping) rather than network
  construction — missing-value handling does not address the KEGG annotation bottleneck
  (only 19 of 174 TEDDY metabolites were mappable)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DNEA R package
  role: Consumes the filtered/imputed matrix; its createDNEAobject() performs per-group
    log-transformation and auto-scaling and checks eigenvalue/condition-number stability
    on the cleaned data
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: R
  role: Language in which the 30%-missingness filter and median imputation are implemented
    prior to DNEA network estimation
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:ec009ae9c78ff476132d299cf6b5404d6819db894d379ec97c20a25f621c2903
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - DNEA R package
  - Metabox2
  - R
  - glasso
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metabolomics-missing-value-filtering-and-imputation

## Summary

Pre-processing step in the DNEA metabolomics workflow that drops metabolites missing in more than 30% of samples and median-imputes the remaining missing values, yielding a complete peak-intensity matrix suitable for GLASSO-based partial correlation network estimation.

## When to use

Use when raw metabolomics peak-intensity tables (e.g., TEDDY ST001386/ST001636 from Metabolomics Workbench) contain missing values and must be made complete before building DNEA partial correlation networks or comparing against marginal-correlation tools like MetaBox2.

## When NOT to use

- Data has already been normalized/log-transformed by another pipeline — re-imputing median values on transformed data is not the paper's recipe (imputation is done on raw peak intensities before DNEA's own log-transform and auto-scaling)
- The analysis is pathway enrichment (e.g., FELLA/KEGG mapping) rather than network construction — missing-value handling does not address the KEGG annotation bottleneck (only 19 of 174 TEDDY metabolites were mappable)

## Inputs

- Raw (unnormalized) metabolomics peak-intensity matrix (samples x metabolites), e.g., TEDDY data from Metabolomics Workbench ST001386/ST001636 or PR000950
- Sample metadata / group labels (e.g., T1Dmeta with 'DM:control' vs 'DM:case') to restrict to the relevant cohort (134 metabolites, 322 samples)

## Outputs

- Complete (no-NA) peak-intensity matrix restricted to metabolites missing in <= 30% of samples, with median-imputed values
- Pre-processed input ready for createDNEAobject() and downstream BICtune/stabilitySelection/getNetworks steps

## How to apply

Apply the recipe described in the DNEA paper to the raw (unnormalized) peak intensity values: first filter columns to exclude any metabolite that is missing in more than 30% of the samples; then for each remaining metabolite, impute all missing values with the median value of that metabolite across samples. On the TEDDY T1D dataset this produced the 134-metabolite, 322-sample matrix used for the case-vs-control differential network. The order matters: imputation happens after exclusion, and only afterwards should DNEA's createDNEAobject() apply its own per-group log-transformation and auto-scaling, so do not transform or scale the data before imputing. Optionally precede this by dropping individuals who converted from control to case and by regressing out covariates (age, gender) via residuals, as the paper's workflow does.

## Related tools

- **DNEA R package** (Consumes the filtered/imputed matrix; its createDNEAobject() performs per-group log-transformation and auto-scaling and checks eigenvalue/condition-number stability on the cleaned data) — http://www.github.com/Karnovsky-Lab/DNEA/
- **R** (Language in which the 30%-missingness filter and median imputation are implemented prior to DNEA network estimation)

## Examples

```
keep <- colMeans(is.na(peak_int)) <= 0.30; dat <- peak_int[, keep]; dat <- apply(dat, 2, function(x) { x[is.na(x)] <- median(x, na.rm = TRUE); x })
```

## Evaluation signals

- Every retained metabolite column has missingness <= 30% across samples; metabolites exceeding that cutoff are removed, not imputed
- The output matrix contains zero NA values after median imputation (verify with sum(is.na(matrix)) == 0)
- On the TEDDY T1D subset, the recipe should reproduce the paper's 134 metabolites x 322 samples input to createDNEAobject()
- Downstream DNEA run passes createDNEAobject()'s eigenvalue/condition-number stability checks and yields the expected 290 total edges (70 control-specific, 112 case-specific, 108 shared)
- Imputation is per-metabolite median (column-wise), not per-sample or global median

## Limitations

- The 30% exclusion cutoff and median imputation were validated specifically on TEDDY metabolomics data in the DNEA paper; other datasets or omics types may need different thresholds
- DNEA's subsequent BIC tuning protocol requires large datasets (~500 or more samples per group); for smaller datasets the lambda approximation (sqrt(ln(features)/samples)) must be used instead, which affects downstream network density regardless of imputation quality

## Evidence

- [methods] Exclude metabolites missing in more than 30% of samples, then median imputation: "The data were filtered to exclude metabolites that were missing in more than 30% of the samples."
- [methods] Median imputation of remaining missing values: "For the remaining metabolites, all missing values were imputed with the median value of each metabolite."
- [other] Imputation precedes DNEA's per-group log-transform and auto-scaling in createDNEAobject(): [evidence span withheld — anchor status: not_found]
- [results] Raw peak intensities are the input to the DNEA analysis: "unnormalized peak intensity values adjusted for age and sex were used as input. DNEA analysis was performed on IA and T1D datasets"
- [methods] Data source for the pre-processing recipe: "The metabolomics data used in our analyses were obtained from the Metabolomics Workbench (ST001386 and ST001636)."
