---
name: 2d-nmr-cross-peak-integral-based-concentration-determination
description: Use when you have a DEEP Picker/Voigt Fitter peak table (one row per
  fitted cross-peak with integral and resonance assignment) plus a reference compound
  of known concentration, and need absolute compound concentration from overlapping
  2D spectra.
when_to_use_negative:
- Input is a raw Bruker FID (fid/ser) rather than an already peak-picked and Voigt-fitted
  peak table — run DEEP Picker/Voigt Fitter processing first.
- No reference compound of known concentration is present in the spectrum, so the
  ratio-based formula cannot be evaluated.
- The compound of interest has no non-overlapping cross-peaks with reliable fitted
  integrals, making the normalized integral sum untrustworthy.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3418
tools:
- name: DEEP Picker
  role: AI-driven neural-network peak deconvolution that produces the per-cross-peak
    integrals used in the concentration sums, even for overlapping peaks
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: 2D Voigt/Gaussian line-shape fitting that reconstructs the experimental spectrum
    and quantifies each cross-peak's integral
  repo: https://github.com/lidawei1975/colmarvista
- name: COLMARvista
  role: Web-based platform integrating DEEP Picker and Voigt Fitter; supports automated
    assignment transfer (Hungarian matching) to label compound vs. reference peaks
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:249b3b16c026935c5473d725b405fa5dbd157f126b6c49907bb644a71c77ffb4
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - DEEP Picker
  - Voigt Fitter
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# 2d-nmr-cross-peak-integral-based-concentration-determination

## Summary

Determines compound concentration from a fitted 2D NMR spectrum by summing Voigt-Fitter cross-peak integrals for the compound and a reference, normalizing by equivalent nuclei counts, and applying the standard 2D NMR concentration ratio formula.

## When to use

Use when you have a DEEP Picker/Voigt Fitter peak table (one row per fitted cross-peak with integral and resonance assignment) plus a reference compound of known concentration, and need absolute compound concentration from overlapping 2D spectra.

## When NOT to use

- Input is a raw Bruker FID (fid/ser) rather than an already peak-picked and Voigt-fitted peak table — run DEEP Picker/Voigt Fitter processing first.
- No reference compound of known concentration is present in the spectrum, so the ratio-based formula cannot be evaluated.
- The compound of interest has no non-overlapping cross-peaks with reliable fitted integrals, making the normalized integral sum untrustworthy.

## Inputs

- Fitted 2D NMR peak table from DEEP Picker/Voigt Fitter (one row per cross-peak: integral, assigned 1H/15N or 1H/13C resonance)
- Resonance assignment list distinguishing compound-of-interest rows from reference-compound rows (formats such as .list, .tab, .txt, .csv supported by COLMARvista assignment transfer)
- Known concentration of the reference compound

## Outputs

- concentration_result.csv with a header row naming the compound and the concentration unit, and a single computed numeric concentration value

## How to apply

Load the fitted 2D peak table produced by DEEP Picker/Voigt Fitter inside COLMARvista, which contains one row per fitted cross-peak with its integral and the assigned resonance (amide 1H/15N for a protein, or methyl/aromatic cross-peaks for a small molecule). Identify the rows belonging to the compound of interest and to the reference compound using the resonance assignments. Sum the fitted integrals of non-overlapping cross-peaks for each compound and normalize each sum by the number of equivalent nuclei contributing to it (e.g. divide the protein amide sum by the number of observable backbone amides). Compute concentration as (compound normalized integral / reference normalized integral) × reference concentration — the standard 2D NMR concentration-determination formula. This works even when peaks significantly overlap because the Voigt deconvolution yields peak-specific integrals rather than naive peak heights.

## Related tools

- **DEEP Picker** (AI-driven neural-network peak deconvolution that produces the per-cross-peak integrals used in the concentration sums, even for overlapping peaks) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (2D Voigt/Gaussian line-shape fitting that reconstructs the experimental spectrum and quantifies each cross-peak's integral) — https://github.com/lidawei1975/colmarvista
- **COLMARvista** (Web-based platform integrating DEEP Picker and Voigt Fitter; supports automated assignment transfer (Hungarian matching) to label compound vs. reference peaks) — https://github.com/lidawei1975/colmarvista

## Examples

```
Visit https://lidawei1975.github.io/colmarvista/ , load the fitted 2D peak table, run DEEP Picker + Voigt Fitter, then export concentration_result.csv
```

## Evaluation signals

- concentration_result.csv exists, is valid CSV, and contains a numeric concentration field
- The computed concentration falls in a physically plausible range given the reference concentration (e.g. same order of magnitude, not negative).
- The compound and reference integral sums were normalized by their respective equivalent-nuclei counts before the ratio was taken.
- Only non-overlapping cross-peaks were included in the sums; check that excluded rows correspond to genuinely overlapped resonances.
- Peak assignments used to group rows were confirmed via the assignment transfer (bipartite Hungarian matching with 1H/heteronucleus distance cutoffs) rather than ad-hoc matching.

## Limitations

- Accuracy depends on the quality of the DEEP Picker/Voigt Fitter deconvolution; severely overlapped or missed peaks bias the integral sums.
- Normalization requires knowing the number of observable backbone amides (or equivalent nuclei), which may be fewer than expected due to exchange or water suppression.
- The web server keeps all data client-side (no uploads); local offline use requires modifying browser settings for WebWorker/WebAssembly (e.g. Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false), which the README warns poses a security risk.

## Evidence

- [abstract] Integration with DEEP Picker and Voigt Fitter enables automated deconvolution and reconstruction for quantitative analysis, including concentration determination and cross-peak specific relaxation parameters for overlapping peaks: "allows the automated deconvolution and reconstruction of the experimental spectra for highly quantitative analysis, from compound concentration determination to the extraction of cross-peak specific"
- [readme] README describes peak picking and line-shape fitting capabilities: "**Peak Picking & 2D Line-Shape Fitting**: AI-driven DEEP Picker neural network, local maxima Simple Picker, and Voigt/Gaussian deconvolution."
- [readme] Assignment transfer supports assignment lists in common formats: "Globally optimal bipartite Hungarian matching ($O(N^3)$) from reference lists (`.list`, `.tab`, `.txt`, `.csv`), customizable $^1$H / heteronucleus distance cutoffs"
- [intro] Web server retains all data locally without uploads: "COLMAR Viewer is a static web server, which means all your data stays on your computer; the server does not take any input or uploads from you."
