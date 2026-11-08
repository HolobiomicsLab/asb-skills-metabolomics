---
name: nmr-match-ratio-scoring
description: Use when candidate metabolites have been matched to one or more user-defined
  NMR spectral ROIs and you must filter out metabolites whose SRRMs mostly lie outside
  the analyzed regions before categorization.
when_to_use_negative:
- ROI boundaries have not yet been defined or matched against the database — run the
  per-ROI chemical-shift matching step first
- The goal is absolute quantification of metabolites — the article states absolute
  quantification is not the program's primary aim
- The user wants automated ROI identification or validation of user-defined ROI accuracy
  — both are explicitly beyond the program's scope
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: ROIAL-NMR
  role: Implements the SRRM match-ratio scoring and >=50% retention filter across
    all ROIs to avoid over-identification
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Loads and joins the per-ROI candidate table and the HMDB chemical-shift/SRRM
    library for ratio computation
- name: Human Metabolome Database (HMDB)
  role: Reference platform supplying the 891 metabolites' chemical shift peaks and
    SRRMs used in ratio calculation
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:d25cbef85ea7779441e3efaba29471d365f309132f11f2804cab3d2e00669a57
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Human Metabolome Database (HMDB)
  - Python
  - ROIAL-NMR
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# nmr-match-ratio-scoring

## Summary

Scores candidate metabolites retrieved from HMDB-derived chemical shift data against user-defined proton NMR ROIs by computing a match ratio (matched SRRMs / total SRRMs per metabolite) and retaining only metabolites with ratio >= 0.5, preventing over-identification in crowded spectra.

## When to use

Use when candidate metabolites have been matched to one or more user-defined NMR spectral ROIs and you must filter out metabolites whose SRRMs mostly lie outside the analyzed regions before categorization.

## When NOT to use

- ROI boundaries have not yet been defined or matched against the database — run the per-ROI chemical-shift matching step first
- The goal is absolute quantification of metabolites — the article states absolute quantification is not the program's primary aim
- The user wants automated ROI identification or validation of user-defined ROI accuracy — both are explicitly beyond the program's scope

## Inputs

- Per-ROI candidate metabolite table from the ROI-matching step (all_metabolites_matches.csv)
- HMDB-derived metabolite library of chemical shift peaks and metabolite-specific SRRMs for 891 metabolites
- User-defined ROI boundary table (e.g., 18 ROIs spanning the 0.5-4.5 ppm up-field segment)

## Outputs

- Scored metabolite table with abbreviation, match ratio, matched regions, and HMDB concentration range per metabolite
- Filtered metabolite list (match ratio >= 0.5) carried into the six-category trend/significance categorization step

## How to apply

Load the per-ROI candidate table and the HMDB-derived database of chemical shift peaks and SRRMs into pandas. First exclude peaks/SRRMs outside the analyzed spectral region (0.5-4.5 ppm for biofluids); if more than 50% of a metabolite's peaks lie beyond the analyzed region, exclude those peaks from the ratio calculation. Count a metabolite-specific SRRM as matched to a ROI when at least one of its chemical shift peaks (e.g., lactate peaks at 1.324 and 4.111 ppm) falls within the experimentally determined ROI boundaries. Compute the match ratio as matched SRRMs divided by total SRRMs, and keep only metabolites with a match ratio >= 0.5 (>= 50% of their SRRMs within the analyzed ROIs) for downstream categorization into the six trend/significance categories. Writing the scored table with match ratios and matched regions preserves the audit trail into the 'All Metabolites' view.

## Related tools

- **ROIAL-NMR** (Implements the SRRM match-ratio scoring and >=50% retention filter across all ROIs to avoid over-identification) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Loads and joins the per-ROI candidate table and the HMDB chemical-shift/SRRM library for ratio computation)
- **Human Metabolome Database (HMDB)** (Reference platform supplying the 891 metabolites' chemical shift peaks and SRRMs used in ratio calculation)

## Examples

```
python main.py  # open 'All Metabolites' window; inspect match_ratio = matched_SRRMs / total_SRRMs and keep metabolites with match_ratio >= 0.5
```

## Evaluation signals

- Lactate worked example reproduces exactly: HMDB SRRMs [1.30-1.33] and [4.08-4.13] ppm match ROIs [1.31-1.34] and [4.10-4.12] ppm, giving match ratio 2/2 = 1.0
- Every retained metabolite has match ratio >= 0.5; no metabolite with <50% of its SRRMs inside the analyzed ROIs survives the filter
- Peaks outside the 0.5-4.5 ppm analyzed region are absent from the ratio calculation; metabolites with >50% of peaks beyond the region have those peaks excluded
- Number of matched SRRMs per metabolite equals the count of SRRMs with at least one peak inside a ROI boundary
- Output table schema includes abbreviation, match ratio, matched regions, and concentration range, matching the 'All Metabolites' window

## Limitations

- Match-ratio scoring depends on user consistency in creating ROIs; variability between users is acknowledged
- The program was not built to test the accuracy of user-defined ROIs
- Filter parameters (e.g., the 50% threshold) are defaults that users can customize, so results may vary with settings
- Both chemical shift values and metabolite-specific SRRMs are required for accurate identification; either alone is insufficient

## Evidence
