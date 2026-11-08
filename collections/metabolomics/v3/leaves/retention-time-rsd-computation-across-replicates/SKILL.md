---
name: retention-time-rsd-computation-across-replicates
description: Use when validating LC-MS reference analytes from >=3 replicate runs
  (e.g. HILIC standard mixtures in mzML/mzXML) and you must quantify retention-time
  reproducibility per analyte against a user-specified threshold c (default 15%).
when_to_use_negative:
- Data consist of a single LC-MS injection per analyte — RSD across replicates is
  undefined without repeated runs
- Goal is mobile-phase optimization scoring (peak quality score Sp + Saf comparison
  across candidate mobile phases) rather than replicate-based analyte validation
- Retention times are already curated from a trusted reference library and only peak-shape
  metrics (MCQ, asymmetry factor) need computing
edam_operation: http://edamontology.org/operation_2945
tools:
- name: MetaMOPE
  role: Containerized web tool that generates EICs, computes the RT RSD metric after
    IRLS baseline correction and Savitzky-Golay smoothing, and applies the three-threshold
    validation criterion
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Language of the source-code functions MetaMOPE provides for computing the
    multiple peak metrics including RSD of retention time
- name: Docker
  role: Container runtime used to build and deploy the MetaMOPE web service via docker-compose
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:4ab2013a23418581933ae1da144ae38fcc92d857db6b8675d5e4b665aabf21ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# retention-time-rsd-computation-across-replicates

## Summary

Computes the relative standard deviation (RSD) of chromatographic peak retention times across replicate LC-MS injections, used in the MetaMOPE three-threshold validation criterion (with Std/Blk ratio r=6 and intensity I=1000) to decide which standard analytes enter a reference library.

## When to use

Use when validating LC-MS reference analytes from >=3 replicate runs (e.g. HILIC standard mixtures in mzML/mzXML) and you must quantify retention-time reproducibility per analyte against a user-specified threshold c (default 15%).

## When NOT to use

- Data consist of a single LC-MS injection per analyte — RSD across replicates is undefined without repeated runs
- Goal is mobile-phase optimization scoring (peak quality score Sp + Saf comparison across candidate mobile phases) rather than replicate-based analyte validation
- Retention times are already curated from a trusted reference library and only peak-shape metrics (MCQ, asymmetry factor) need computing

## Inputs

- Three replicate LC-MS runs of a standard mixture in mzML or mzXML format
- List of standard analyte m/z values for extracted ion chromatogram (EIC) generation
- User-specified threshold c for maximum tolerated RT RSD (default 15% in the reference evaluation)

## Outputs

- Per-metabolite RSD of retention time across the three replicates (one of MetaMOPE's eight peak metrics)
- Per-metabolite validation flag reflecting the three-threshold criterion (RSD < c, Std/Blk > r, intensity > I)

## How to apply

Run each standard mixture three times on the LC-MS method, then extract the EIC of each analyte from the replicate mzML/mzXML files. Preprocess each EIC with IRLS baseline correction and Savitzky-Golay smoothing, and determine the peak retention time per replicate from the local maximum intensity. Compute the RSD of retention time as the relative standard deviation of the three replicate RTs and report it as one of the eight MetaMOPE peak metrics (alongside MCQ, maximum intensity, Std/Blk ratio, jaggedness, asymmetry factor, FWHM, modality). Mark the analyte's validation flag as true only if RSD < c (user-specified, 15% in the reference case) AND Std/Blk > r (6) AND peak intensity > I (1000). In the published run this correctly validated taurocholate, l-aspartate, and sphingosine, while estradiol (Std/Blk 4.719 < 6) and l-cystine (intensity 915.368 < 1000) were excluded.

## Related tools

- **MetaMOPE** (Containerized web tool that generates EICs, computes the RT RSD metric after IRLS baseline correction and Savitzky-Golay smoothing, and applies the three-threshold validation criterion) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Language of the source-code functions MetaMOPE provides for computing the multiple peak metrics including RSD of retention time)
- **Docker** (Container runtime used to build and deploy the MetaMOPE web service via docker-compose)

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && docker-compose up --build
```

## Evaluation signals

- RSD values are unitless percentages, and every analyte flagged validated must have RT RSD below the chosen threshold c (15% in the reference case)
- Reproduced validation flags match the reported decisions: taurocholate, l-aspartate, and sphingosine validated; estradiol excluded (Std/Blk 4.719 < 6); l-cystine excluded (intensity 915.368 < 1000)
- Retention times per replicate derive from the local maximum intensity of the EIC after IRLS baseline correction and Savitzky-Golay smoothing, not from raw uncorrected traces
- The output table includes the validation column that decides whether metabolites in the sample mixture are validated by the user-specified thresholds
- Only metabolites satisfying all three criteria simultaneously receive validated flags; failure of any single criterion excludes the metabolite

## Limitations

- The RSD threshold c is user-specified, so validation decisions are sensitive to its choice; the article's documented discussion section contained no changelog, so no additional limitations or reproducibility gaps were reported
- An analyte is excluded if ANY one criterion fails even marginally (e.g. l-cystine at intensity 915.368 just below I=1000), so borderline metabolites may be dropped from the reference library
- RSD quality depends on correct peak detection and RT assignment from EIC local maxima; poor baseline correction or smoothing can distort replicate RTs

## Evidence

- [abstract] Only the peaks that satisfy the following criteria: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r, the relative standard deviation: [evidence span withheld — anchor status: not_found]
- [abstract] MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity, and: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity, and"
- [abstract] The LC/MS method is used to evaluate each standard mixture three times, and the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates.: "The LC/MS method is used to evaluate each standard mixture three times, and the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates."
- [readme] Run `docker-compose up --build`: "Run `docker-compose up --build`"
