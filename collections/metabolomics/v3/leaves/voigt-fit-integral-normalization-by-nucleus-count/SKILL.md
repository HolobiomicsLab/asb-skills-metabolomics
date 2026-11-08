---
name: voigt-fit-integral-normalization-by-nucleus-count
description: Use when a fitted 2D NMR peak table (one row per cross-peak with integral
  and resonance assignment, e.g. amide 1H/15N or methyl/aromatic) is available and
  a compound concentration must be derived relative to a reference compound of known
  concentration.
when_to_use_negative:
- Input is raw Bruker time-domain FID data (fid, ser) rather than an already-fitted
  cross-peak table with integrals — run DEEP Picker/Voigt Fitter fitting first
- No reference compound of known concentration is present in the spectrum, so the
  ratio-based formula cannot be applied
- The compound has no non-overlapping cross-peaks to sum (the method explicitly relies
  on summing non-overlapping fitted peaks)
edam_operation: http://edamontology.org/operation_3799
tools:
- name: DEEP Picker
  role: Deep neural network that deconvolves the complex 2D spectrum and picks the
    cross-peaks whose rows populate the fitted peak table
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Voigt/Gaussian line-shape fitting inside COLMARvista that yields the per-cross-peak
    integrals used in the nucleus-count-normalized sums
  repo: https://github.com/lidawei1975/colmarvista
- name: COLMARvista
  role: Web-based environment integrating DEEP Picker and Voigt Fitter from which
    the fitted peak table is loaded for quantitative concentration determination
  repo: https://github.com/lidawei1975.github.io/colmarvista
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:6547144780b6b39f1375a096a45e54c315a2fe9f11069b175a8e8445f9999da1
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# voigt-fit-integral-normalization-by-nucleus-count

## Summary

Determines compound concentration from a fitted 2D NMR spectrum by summing Voigt-Fitter cross-peak integrals for the compound and a reference of known concentration, normalizing each sum by the number of equivalent nuclei, and applying the standard 2D NMR concentration formula.

## When to use

Use when a fitted 2D NMR peak table (one row per cross-peak with integral and resonance assignment, e.g. amide 1H/15N or methyl/aromatic) is available and a compound concentration must be derived relative to a reference compound of known concentration.

## When NOT to use

- Input is raw Bruker time-domain FID data (fid, ser) rather than an already-fitted cross-peak table with integrals — run DEEP Picker/Voigt Fitter fitting first
- No reference compound of known concentration is present in the spectrum, so the ratio-based formula cannot be applied
- The compound has no non-overlapping cross-peaks to sum (the method explicitly relies on summing non-overlapping fitted peaks)

## Inputs

- Fitted 2D NMR peak table from DEEP Picker/Voigt Fitter (one row per fitted cross-peak with integral and assigned resonance, e.g. amide 1H/15N for a protein or methyl/aromatic cross-peak for a small molecule)
- Reference compound of known concentration present in the same spectrum

## Outputs

- concentration_result.csv containing a single computed concentration value with a header row naming the compound and the concentration unit

## How to apply

Load the DEEP Picker/Voigt Fitter peak table produced inside COLMARvista and identify rows belonging to the compound of interest and the reference compound via their resonance assignments. Sum the fitted integrals of the non-overlapping cross-peaks for each compound — exclude overlapped peaks, since Voigt/Gaussian deconvolution is what makes the remaining integrals trustworthy. Normalize each sum by the number of equivalent nuclei contributing to it (e.g. divide the protein amide-sum by the number of observable backbone amides, which may be fewer than the sequence length). Compute concentration as (compound normalized integral / reference normalized integral) × reference concentration, the standard 2D NMR concentration-determination formula. Write the single computed value to concentration_result.csv with a header row naming the compound and concentration unit.

## Related tools

- **DEEP Picker** (Deep neural network that deconvolves the complex 2D spectrum and picks the cross-peaks whose rows populate the fitted peak table) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Voigt/Gaussian line-shape fitting inside COLMARvista that yields the per-cross-peak integrals used in the nucleus-count-normalized sums) — https://github.com/lidawei1975/colmarvista
- **COLMARvista** (Web-based environment integrating DEEP Picker and Voigt Fitter from which the fitted peak table is loaded for quantitative concentration determination) — https://github.com/lidawei1975.github.io/colmarvista

## Examples

```
python -c "c=(compound_norm_int/reference_norm_int)*reference_conc; open('concentration_result.csv','w').write('compound,concentration\n'+str(c))"
```

## Evaluation signals

- concentration_result.csv exists, parses as valid CSV, and has a header row naming the compound and the concentration unit
- The file contains a numeric concentration field whose value falls in a physically plausible range given the reference concentration
- Both integral sums exclude overlapping cross-peaks — only non-overlapping fitted peaks are summed per compound
- Each compound's integral sum is divided by the number of equivalent nuclei (e.g. observable backbone amides) before the ratio is taken
- Result scales linearly with the reference concentration: halving the reference concentration halves the computed value

## Limitations

- Concentration determination is only valid when identifiable non-overlapping cross-peaks exist for both the compound and the reference; significant overlap must be handled by the Voigt deconvolution itself
- The number of observable backbone amides may be smaller than the protein sequence length, so the normalization count must reflect actually observable residues
- Local offline use requires relaxing browser WebWorker/WebAssembly security settings (e.g. Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false), which the README warns poses a security risk

## Evidence
