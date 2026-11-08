---
name: nmr-peak-pairing-between-spectra
description: Use when experimental mixture NMR spectra and pure-component reference
  spectra are both available and you need to measure how far the same compound's peaks
  shift between mixtures, e.g. to derive or validate a chemical shift tolerance interval
  in ppm.
when_to_use_negative:
- Input spectra are simulated or augmented synthetic superpositions rather than experimental
  mixture spectra — the derived ppm variation interval is an empirical property of
  real mixture/reference pairs
- Solvent is not DMSO-d6/HDO or the compounds lack active-hydrogen signals — the exclusion
  filter steps would remove nothing and other solvent artifacts may need different
  treatment
- Goal is compound identification itself; use the trained pSCNN model prediction pipeline
  (pSCNN.py) instead of measuring shift deviation
tools:
- name: Python 3.8.13
  role: Runtime for loading spectra, pairing peaks along the ppm axis, and aggregating
    deviation statistics with numpy/pandas
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: pSCNN
  role: Deposited model and NMR data package whose example spectra serve as the inputs
    for peak pairing and shift-deviation measurement
  repo: https://doi.org/10.5281/zenodo.6504814
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:8d48a4bb596b93ebd613d23d96deccca84513e391c2ed5b79b224396a7b9b67a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN.git
  tools:
  - Python
  - Python 3.8.13
  - pSCNN
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# nmr-peak-pairing-between-spectra

## Summary

Quantifies chemical shift variation between experimental 1H NMR spectra of mixtures and their pure-component reference spectra by detecting and pairing peaks across the ppm axis and excluding active-hydrogen and DMSO-d6/HDO solvent signals. This establishes the tolerance interval (mean ± 1.5 × SD) that motivates chemical-shift-robust compound identification methods such as pSCNN.

## When to use

Use when experimental mixture NMR spectra and pure-component reference spectra are both available and you need to measure how far the same compound's peaks shift between mixtures, e.g. to derive or validate a chemical shift tolerance interval in ppm.

## When NOT to use

- Input spectra are simulated or augmented synthetic superpositions rather than experimental mixture spectra — the derived ppm variation interval is an empirical property of real mixture/reference pairs
- Solvent is not DMSO-d6/HDO or the compounds lack active-hydrogen signals — the exclusion filter steps would remove nothing and other solvent artifacts may need different treatment
- Goal is compound identification itself; use the trained pSCNN model prediction pipeline (pSCNN.py) instead of measuring shift deviation

## Inputs

- Experimental 1H NMR spectra of mixtures (e.g. flavor mixtures dataset, DMSO-d6, from Zenodo DOI 10.5281/zenodo.6504814)
- Pure-component reference NMR spectra corresponding to compounds in the mixtures
- Spectral data loaded in Python 3.8.13 (numpy/pandas arrays indexed along the ppm axis)

## Outputs

- chemical_shift_deviation_stats.csv containing mean, standard deviation, and mean ± 1.5 × SD interval of chemical shift deviations in ppm
- Paired peak list (mixture peak ↔ reference peak) with ppm deviations, excluding active-hydrogen and DMSO-d6/HDO signals

## How to apply

Download the deposited NMR model and example data from Zenodo (DOI 10.5281/zenodo.6504814) as referenced in the methods, and load the experimental mixture spectra with their pure-component references in Python 3.8.13 (numpy/pandas). Detect and pair peaks between each mixture spectrum and its component reference spectrum across the ppm axis. Exclude active-hydrogen signals and the residual solvent signals of DMSO-d6/HDO from the peak pairs before computing deviations, since these signals shift systematically and would inflate the variation. Compute the chemical shift deviation in ppm for each retained pair, then aggregate the mean, standard deviation, and mean ± 1.5 × SD interval; write the statistics table to chemical_shift_deviation_stats.csv. In this paper's flavor-mixture data the reproduced mean was −0.0016 ppm with SD 0.0077 ppm, giving the interval (−0.013, 0.010) ppm — which is why the translational invariance of CNNs can absorb chemical shift variation in pSCNN.

## Related tools

- **Python 3.8.13** (Runtime for loading spectra, pairing peaks along the ppm axis, and aggregating deviation statistics with numpy/pandas) — https://github.com/yuxuanliao/pSCNN.git
- **pSCNN** (Deposited model and NMR data package whose example spectra serve as the inputs for peak pairing and shift-deviation measurement) — https://doi.org/10.5281/zenodo.6504814

## Examples

```
git clone https://github.com/yuxuanliao/pSCNN.git && conda create --name pSCNN python=3.8.13 && conda activate pSCNN && python pSCNN.py
```

## Evaluation signals

- Reproduced mean chemical shift deviation equals −0.0016 ppm within numerical tolerance
- Reproduced standard deviation equals 0.0077 ppm within numerical tolerance
- Mean ± 1.5 × SD interval matches the reported (−0.013, 0.010) ppm
- All active-hydrogen and DMSO-d6/HDO peak pairs are excluded before aggregation (no residual solvent signals present in the deviation table)
- chemical_shift_deviation_stats.csv is produced with mean, SD, and interval columns

## Limitations

- The interval is derived from two experimental NMR datasets (flavor mixtures and additional flavor mixture); it may not transfer to other solvents, field strengths, or compound classes
- Peak overlap among molecules and systematic shifts of active-hydrogen/solvent signals must be handled manually via exclusion before pairing, otherwise deviations are inflated
- No discussion-section content was available in the indexed document, so reproducibility gaps and limitations could not be fully extracted

## Evidence

- [full_text] chemical shift variation finding: [evidence span withheld — anchor status: not_found]
- [methods] workflow steps: [evidence span withheld — anchor status: not_found]
- [intro] motivation for shift variation: "is difficult to identify compounds in mixtures because of chemical shift variation of the same compound in different mixtures and peak overlapping among molecules"
- [readme] data source: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
