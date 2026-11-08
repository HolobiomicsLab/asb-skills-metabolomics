---
name: lcms-replicate-retention-time-analysis
description: Use when you have three or more replicate mzXML/mzML injections of a
  chemical-standard mixture (e.g. from comparing HILIC LC-MS mobile phases) and need
  per-peak quality metrics, validated peaks, and replicate-consistent retention times.
when_to_use_negative:
- Input is already a processed peak/feature table rather than raw mzXML/mzML replicate
  files — the EIC generation, IRLS baseline correction and Savitzky-Golay smoothing
  steps assume raw chromatographic data
- Fewer than three replicate injections are available, since RT RSD and replicate-consistent
  retention time are computed across three replicates
- The goal is untargeted metabolite feature detection rather than evaluation of known
  chemical standards — MetaMOPE is designed for batch-evaluating peaks of standard
  analytes
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaMOPE
  role: Containerized web tool that generates EICs, applies IRLS baseline correction
    and Savitzky-Golay smoothing, computes the eight per-peak metrics, validates peaks
    against thresholds, and scores mobile phases (Sp + Saf)
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Deployment container for MetaMOPE; the service is launched with docker-compose
    and accessed at localhost:13002
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Language of the source-code functions implementing the peak-quality metrics
    (MCQ, asymmetry factor, etc.)
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:5a354a6a6847785eaf20b2d183a65a7ade625b421fa6639afd1e1b97244a3c84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lcms-replicate-retention-time-analysis

## Summary

Batch-evaluates chromatography peaks across LC-MS replicate acquisitions (mzXML/mzML) to determine analyte retention times and validate peaks, using eight per-peak metrics (MCQ, max intensity, RT RSD, Std/Blk ratio, jaggedness, asymmetry factor, FWHM, modality) computed by MetaMOPE after IRLS baseline correction and Savitzky-Golay smoothing.

## When to use

Use when you have three or more replicate mzXML/mzML injections of a chemical-standard mixture (e.g. from comparing HILIC LC-MS mobile phases) and need per-peak quality metrics, validated peaks, and replicate-consistent retention times.

## When NOT to use

- Input is already a processed peak/feature table rather than raw mzXML/mzML replicate files — the EIC generation, IRLS baseline correction and Savitzky-Golay smoothing steps assume raw chromatographic data
- Fewer than three replicate injections are available, since RT RSD and replicate-consistent retention time are computed across three replicates
- The goal is untargeted metabolite feature detection rather than evaluation of known chemical standards — MetaMOPE is designed for batch-evaluating peaks of standard analytes

## Inputs

- Three (or more) replicate mzXML/mzML files per condition (e.g. mobile phase) for a chemical-standard mixture
- List of standard analyte masses/mz targets for EIC extraction
- Blank injections for Std/Blk intensity ratio computation
- User-specified thresholds: Std/Blk ratio r, RT RSD c, and intensity I

## Outputs

- Per-peak table of eight metrics: MCQ, maximum intensity, RT RSD, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, modality
- Validation flag per metabolite/peak (pass/fail against thresholds r, c, I)
- Retention time per reference analyte (from EICs across three replicates)
- Count of validated, resolved peaks per condition and peak quality score (Sp + Saf)
- mobile_phase_peak_quality_scores.csv (peak quality score table per mobile phase)

## How to apply

Read each replicate's mzXML/mzML file and generate extracted ion chromatograms (EICs) for the standard analytes. Preprocess every EIC with IRLS (iteratively reweighted least squares) baseline correction followed by Savitzky-Golay smoothing, then compute the eight MetaMOPE metrics per peak: MCQ, maximum intensity, retention-time RSD across the three replicates, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM and modality. A peak is validated only when its Std/Blk ratio exceeds user threshold r, its RT RSD across replicates is below user threshold c, and its intensity exceeds user threshold I; the validation column flags which metabolites pass. Retention time for each reference analyte is generated from its EIC across the three replicates, and the count of validated, resolved peaks per condition feeds the detected-peaks score (Sp). Use the resulting peak quality score table to rank conditions (e.g. mobile phases) — the best is the one with the highest Sp + Saf sum, as in the 20-standard example where Mobile Phase B (10 peaks, score 3) beat Phase A (4 peaks, score 2).

## Related tools

- **MetaMOPE** (Containerized web tool that generates EICs, applies IRLS baseline correction and Savitzky-Golay smoothing, computes the eight per-peak metrics, validates peaks against thresholds, and scores mobile phases (Sp + Saf)) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment container for MetaMOPE; the service is launched with docker-compose and accessed at localhost:13002) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Language of the source-code functions implementing the peak-quality metrics (MCQ, asymmetry factor, etc.)) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git; docker-compose up --build; # then open http://localhost:13002
```

## Evaluation signals

- Each per-peak record contains all eight metrics: MCQ, maximum intensity, RT RSD, Std/Blk ratio, jaggedness, asymmetry factor, FWHM and modality
- Validation flags are internally consistent: a peak passes only when Std/Blk > r, RT RSD < c, and intensity > I
- Retention times for each reference analyte agree across the three replicate injections (RSD below threshold c)
- Peak quality score table reproduces the article's benchmark pattern — e.g. Phase A: 4 resolved peaks, Sp = 1, Saf = 1; Phase B: 10 resolved peaks, Sp = 2, Saf = 1; totals 2 vs 3
- Output CSV (mobile_phase_peak_quality_scores.csv) is produced and the selected condition is the one with the highest Sp + Saf sum

## Limitations

- Peak validation depends entirely on user-specified thresholds (r, c, I); different settings change which peaks count as validated and thus the final score
- The scoring workflow assumes each condition is evaluated with three replicate injections; the RT RSD metric is computed across those three replicates
- No changelog was found in the document, so limitations, missing artifacts, or reproducibility gaps could not be extracted from a discussion section

## Evidence

- [abstract] EIC preprocessing then eight metrics: "We calculated the maximum intensity, mass chromatographic quality index (MCQ) and asymmetry factor after IRLS (Iteratively reweighted least squares) baseline correction and Savitzky-Golay smoothing"
- [abstract] Eight per-peak metrics enumerated: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity, and"
- [abstract] Peak validation criteria thresholds: "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r, the relative standard deviation"
- [results] Optimal phase selection rule: "The optimal mobile phase can be quickly determined by selecting the mobile phase that produces the largest number of resolved peaks"
- [results] Deployment via Docker: "Run `docker-compose up --build`"
