---
name: qc-representativeness-hypothesis-testing
description: Use when normalizing LC-MS metabolomics data with QC-based methods and
  you must decide whether QC samples faithfully represent biological-sample signal
  drift, or whether normalization must fall back to biological samples.
when_to_use_negative:
- When QCs and biological samples are known to be identical in matrix and only a fixed
  drift correction is needed, the test and its type covariate add cost without changing
  the fit
- When using QC-RLSC with QConly = TRUE deliberately prescribed (as in the README
  example), the sample-based fallback triggered by this test is not what the protocol
  intends
- When the data contain no QC samples at all — the test compares QC vs biological
  samples and cannot run on sample-only matrices
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: metanorm
  role: Package function exposing QCcheck = TRUE; runs the representativeness hypothesis
    test and re-fits with biological samples (QConly = FALSE) when the test fails
  repo: github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: Fits the thin-plate-spline GAM by REML whose 'type' covariate (QC vs biological)
    supplies the hypothesis test for QC representativeness
- name: R
  role: Runtime environment (version >= 4.4.0) in which the metanorm package and its
    QCcheck test are executed
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:c1a00a8569d5bc6b853d53653435668114c80e01658194bf69c1d79c07db0124
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - calcWeights
  - changepoint
  - cowplot
  - fANCOVA
  - ggplot2
  - metanorm
  - mgcv
  - pbapply
  - plotPCA
  - rLOESS
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# QC-representativeness hypothesis testing

## Summary

A hypothesis test in the metanorm R package that checks whether QC samples are representative of biological samples before QC-based normalization; if QCs and biological samples differ significantly, the tGAM/QC-RLSC model is re-fitted using biological samples only.

## When to use

Use when normalizing LC-MS metabolomics data with QC-based methods and you must decide whether QC samples faithfully represent biological-sample signal drift, or whether normalization must fall back to biological samples.

## When NOT to use

- When QCs and biological samples are known to be identical in matrix and only a fixed drift correction is needed, the test and its type covariate add cost without changing the fit
- When using QC-RLSC with QConly = TRUE deliberately prescribed (as in the README example), the sample-based fallback triggered by this test is not what the protocol intends
- When the data contain no QC samples at all — the test compares QC vs biological samples and cannot run on sample-only matrices

## Inputs

- rawdata: numerical matrix of unnormalized metabolite intensities (rows = compounds, columns = samples), e.g. from example.RData
- metanorm.qc: per-run vector of sample types with 'QC' and non-QC labels
- batch: per-run batch vector so the type test can be fitted batchwise
- QCcheckp / changepoints options and a user-specified significance cut-off for the type-covariate test

## Outputs

- normdat: normalized data matrix produced under either QC-based or re-fitted sample-based normalization
- Decision outcome of the QC-representativeness test (pass -> keep QC-based fit; fail -> re-fit on biological samples)
- Diagnostic plots written to plotdir (pre- vs post-normalization intensity-vs-order plots per compound)

## How to apply

Pass the sample-type vector (e.g. metanorm.qc with 'QC' vs 'sample' labels) via the type argument and enable QCcheck = TRUE in the metanorm() call. The test adds a 'type' covariate to the tGAM model (mgcv thin-plate-spline GAM fitted by REML with gam.k and gam.frac settings) comparing QC vs biological samples; if a significant difference is detected at a user-specified cut-off, the model is re-fitted and normalization uses biological samples only (set QConly = FALSE so the sample-based fallback triggers automatically). Rationale: differences between QC and biological samples would otherwise bias drift estimation and may also invalidate QC-RSD filtering, so the test guards against unrepresentative QCs. The authors recommend using both QC and biological samples for normalization together with this discrepancy check as the default safe configuration.

## Related tools

- **metanorm** (Package function exposing QCcheck = TRUE; runs the representativeness hypothesis test and re-fits with biological samples (QConly = FALSE) when the test fails) — github.com/UGent-LIMET/Metanorm
- **mgcv** (Fits the thin-plate-spline GAM by REML whose 'type' covariate (QC vs biological) supplies the hypothesis test for QC representativeness)
- **R** (Runtime environment (version >= 4.4.0) in which the metanorm package and its QCcheck test are executed) — https://cloud.r-project.org/index.html

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Confirm QCcheck = TRUE was set and a type-covariate p-value / cut-off decision was recorded for each compound
- When the test is significant, verify the returned normdat was produced from the re-fitted model using biological samples only, not the QC-based fit
- Inspect per-compound pre- vs post-normalization intensity-vs-order plots in plotdir: signal drift in biological (non-QC) samples should be corrected after re-fitting
- plotPCA(normdat, type = batch) score plots should show diminished batch effects relative to plotPCA(rawdata, type = batch)
- Check QC-RSD-based filtering decisions were revisited, since QC/biological differences invalidate RSD filtering on QC samples

## Limitations

- Differences between QC and biological samples may invalidate downstream filtering on relative standard deviation (RSD) observed in QC samples, beyond the normalization itself
- The QCcheck outcome depends on a user-specified cut-off; an inappropriate threshold can either over-trigger sample-based re-fitting or miss unrepresentative QCs
- The README notes that apparent absence of batch effects in PCA score plots does not guarantee correct normalization — per-compound pre/post plots must be inspected, since drift in biological samples can be hidden (e.g. the QC-RLSC example where plot 4 reveals problems)

## Evidence

- [methods] QCcheck = TRUE,      # check whether QCs are representative: "QCcheck = TRUE,      # check whether QCs are representative"
- [discussion] differences between QC and biological samples may invalidate filtering on RSD in QC samples: "differences between QC and biological samples may invalidate filtering on e.g., relative standard deviation (RSD) observed in QC samples"
- [readme] recommend using both QC as well as biological samples and have metanorm check discrepancies: "We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples."
- [readme] PCA plot absence of batch effects does not guarantee correct normalization of biological-sample drift: "While no apparent batch effects remain in the PC score plot, you might want to check the pre versus post-normalization plot number 4, for example! What does the signal drift look like in the"
