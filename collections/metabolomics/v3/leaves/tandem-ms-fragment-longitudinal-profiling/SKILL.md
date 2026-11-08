---
name: tandem-ms-fragment-longitudinal-profiling
description: Use when identifying isomeric metabolites in untargeted LC-MS/MS data
  acquired at multiple distinct collision energies (NCE 1–100% or stepped 20/40/60%),
  where single-CE cross-sectional spectral similarity misranks isomers with similar
  predominant fragments.
when_to_use_negative:
- MS/MS spectra were acquired at only a single collision energy, so no longitudinal
  fragment profile (fragment intensity trajectory across CEs) can be constructed
- The goal is raw peak detection or feature quantification rather than isomer-level
  metabolite identification — this skill does not replace MS1 full-scan feature processing
- No reference library with multi-CE spectra of candidate isomers is available (the
  CQMD/CQMU databases are in-house and no public accession is stated)
tools:
- name: 001-007.ipynb
  role: 'Notebook series executing the pipeline: MS1 and MS/MS spectrum extraction,
    spectrum filtering, invalid fragment removal, fragment rationality assessment,
    CQMU database construction, isomer similarity calculations, and metabolite identification'
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Python library of mass spectrometry data-processing methods (file operations,
    data validation/generation/calculation/extraction) used alongside the notebooks
    for CQMD loading and similarity computation
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Jaccard similarity algorithm
  role: Optimized scoring function that compares longitudinal fragment profiles between
    experimental and reference spectra in place of dot-product or optimal-entropy
    cross-sectional algorithms
- name: conda
  role: Environment manager creating the python=3.8.13 'longprof' environment with
    dependencies from requirements.txt
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:af50cf76001f5d17f9dff2024089da8566e89b2c6a7d684018ee8b50de70c009
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - conda
  - mzbatch.py
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# tandem-ms-fragment-longitudinal-profiling

## Summary

A metabolite-identification skill that matches experimental MS/MS spectra to a reference library using longitudinal fragment profiles (LFPs) — each fragment's relative-intensity trajectory across 13 collision energies (e.g., NCE 1–100% or stepped 20/40/60%) — scored with an optimized Jaccard similarity and calibrated by ROC-derived I/N-ratio cutoffs, improving isomer accuracy by 8.7–25.9% over cross-sectional profiling.

## When to use

Use when identifying isomeric metabolites in untargeted LC-MS/MS data acquired at multiple distinct collision energies (NCE 1–100% or stepped 20/40/60%), where single-CE cross-sectional spectral similarity misranks isomers with similar predominant fragments.

## When NOT to use

- MS/MS spectra were acquired at only a single collision energy, so no longitudinal fragment profile (fragment intensity trajectory across CEs) can be constructed
- The goal is raw peak detection or feature quantification rather than isomer-level metabolite identification — this skill does not replace MS1 full-scan feature processing
- No reference library with multi-CE spectra of candidate isomers is available (the CQMD/CQMU databases are in-house and no public accession is stated)

## Inputs

- Experimental MS/MS spectra acquired at 13 distinct collision energies (NCE 1–100%) or stepped NCE 20/40/60%, in CP, CN, AP, and AN ionization modes
- In-house CQMD spectral database of ~180,000 MS/MS spectra of over 4500 metabolites (isomer pair similarity data loaded via 001-007.ipynb and mzbatch.py)
- Candidate metabolite lists from KEGG and HMDB
- NIST MS/MS database spectra for validation

## Outputs

- CQMU database (constructed after spectrum filtering, invalid fragment removal, and fragment rationality assessment)
- roc_analysis_results.csv with per-LFP-number-bin AUC values, optimal cutoffs, and per-mode sensitivity/specificity
- Metabolite identifications with Jaccard-based longitudinal-profile similarity scores

## How to apply

Preprocess spectra with the 001-007.ipynb notebooks: extract MS1 and MS/MS spectra, filter peaks with relative intensity below 1%, remove invalid fragments, and assess fragment rationality. Build longitudinal fragment profiles from relative ion intensities of each fragment across the 13 distinct collision energies so predominant and minute fragments are weighed independently, then compute the I/N ratio (intensity-based over number-based Jaccard similarity) for identical isomer pairs versus constitutional isomer pairs in the in-house CQMD database (~180,000 MS/MS spectra of >4500 metabolites). Stratify isomer pairs into bins by LFP number and run ROC analysis per bin to obtain AUC values (expected 0.874–0.991) and the optimal similarity-score cutoff per bin. Apply the derived cutoffs to score candidates retrieved from KEGG/HMDB against the CQMU database, evaluating sensitivity and specificity per ionization mode (CP, CN, AP, AN) at NCE 1–100%. The rationale is that cross-sectional scoring diminishes minute fragments that carry subtle structural differences, whereas longitudinal profiles make the false discovery rate 28.6–41.7% lower than the optimal entropy algorithm.

## Related tools

- **001-007.ipynb** (Notebook series executing the pipeline: MS1 and MS/MS spectrum extraction, spectrum filtering, invalid fragment removal, fragment rationality assessment, CQMU database construction, isomer similarity calculations, and metabolite identification) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Python library of mass spectrometry data-processing methods (file operations, data validation/generation/calculation/extraction) used alongside the notebooks for CQMD loading and similarity computation) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Jaccard similarity algorithm** (Optimized scoring function that compares longitudinal fragment profiles between experimental and reference spectra in place of dot-product or optimal-entropy cross-sectional algorithms)
- **conda** (Environment manager creating the python=3.8.13 'longprof' environment with dependencies from requirements.txt) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt && python -m jupyter nbconvert --to notebook --execute 001-007.ipynb
```

## Evaluation signals

- ROC analysis within each LFP-number bin yields AUC values in the expected 0.874–0.991 range
- Per-mode sensitivity at NCE 1–100% matches 100, 98.35, 99.56, 99.69% and specificity matches 92.98, 91.67, 94.17, 91.91% (CP, CN, AP, AN)
- Average sensitivity ≈ 98.89% and average specificity ≈ 92.66% across the four ionization modes
- roc_analysis_results.csv values reproduce Figures 5e–h, S6, and Excel S9–S13
- Annotation accuracy on human plasma samples improves 8.7–25.9% and FDR drops 28.6–41.7% relative to the optimal entropy cross-sectional algorithm

## Limitations

- The CQMD/CQMU databases are in-house; no accession or public availability is stated, so full replication requires access to the authors' spectral library
- Results and data are provided 'as is', intended for experimental reference only and not to guide human medicine (README disclaimer)
- The method depends on MS/MS acquisition at multiple distinct collision energies (13 CEs or stepped NCE 20/40/60%); availability of the referenced supplementary artifacts (Figures 5e–h, S6, Excel S9–S13) is not confirmed in the discussion section

## Evidence

- [methods] Peak filtering threshold: [evidence span withheld — anchor status: not_found]
