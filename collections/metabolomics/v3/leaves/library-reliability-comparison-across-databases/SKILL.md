---
name: library-reliability-comparison-across-databases
description: Use when a task asks to reproduce or verify library-level spectral-rating
  reliability statistics — e.g. the fraction of library spectra with average user
  match ratings below 2.5 stars — across multiple spectral libraries.
when_to_use_negative:
- The analysis concerns identification counts or library coverage (e.g. the 83.6%
  of GNPS-Community identifications using reference spectra not uploaded by the submitter)
  rather than per-spectrum user star ratings.
- Ratings are raw per-match values and the goal is match-level statistics, not per-spectrum
  averages aggregated across matches.
- The comparison targets spectral match scoring (normalized dot product between library
  and query spectra), not curation-quality ratings.
edam_topics: []
tools:
- name: GNPS
  role: Source of the GNPS-Collection and GNPS-Community library spectra and their
    crowdsourced user match ratings (4-star correct to 1-star incorrect scale) used
    as the reliability data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:518a2f6a0e478827f08b2b98554fc6afb54cf77a73a66e090857fbf1a4942025
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Library reliability comparison across databases

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Quantifies per-library perceived annotation reliability by computing the fraction of rated library spectra whose average user match rating falls below a 2.5-star threshold, then compares recomputed fractions against article-reported values for GNPS-Collection/GNPS-Community, NIST, and MassBank libraries.

## When to use

Use when a task asks to reproduce or verify library-level spectral-rating reliability statistics — e.g. the fraction of library spectra with average user match ratings below 2.5 stars — across multiple spectral libraries.

## When NOT to use

- The analysis concerns identification counts or library coverage (e.g. the 83.6% of GNPS-Community identifications using reference spectra not uploaded by the submitter) rather than per-spectrum user star ratings.
- Ratings are raw per-match values and the goal is match-level statistics, not per-spectrum averages aggregated across matches.
- The comparison targets spectral match scoring (normalized dot product between library and query spectra), not curation-quality ratings.

## Inputs

- User match-rating records (3,608 rated matches over 2,041 library spectra, Supplementary Table 6/7), each with a library spectrum identifier, source library label (GNPS-Collection/GNPS-Community, NIST, MassBank), and a 1-4 star user rating
- Article-reported library reliability fractions for validation (1.7% GNPS, 10.5% NIST, 20.1% MassBank)

## Outputs

- library_rating_fractions.csv with columns: library, rated_spectra_count, below_threshold_count, fraction

## How to apply

Load the user match-rating records (3,608 rated matches associated with 2,041 library spectra, per Supplementary Table 6/7 in the SI) into a pandas DataFrame and group ratings by library spectrum on the 4-star (correct) to 1-star (incorrect) scale. Compute the average star rating per spectrum, then split spectra by source library (GNPS-Collection/GNPS-Community: 1,746 rated spectra; NIST; MassBank) and count spectra whose average rating falls below the 2.5-star threshold. Divide below-threshold counts by the number of rated spectra per library to obtain fractions, expecting 29/1,746 = 1.7% for GNPS, 10.5% for NIST, and 20.1% for MassBank. The rationale is that crowd user ratings (correct vs. incorrect at the 4- vs 1-star ends) are the article's direct measure of perceived annotation reliability for crowdsourced versus curated commercial libraries. Write the final table with library, rated_spectra_count, below_threshold_count, and fraction columns for direct comparison against reported values.

## Related tools

- **GNPS** (Source of the GNPS-Collection and GNPS-Community library spectra and their crowdsourced user match ratings (4-star correct to 1-star incorrect scale) used as the reliability data)

## Examples

```
df.groupby(['library','spectrum_id'])['rating'].mean().reset_index().pipe(lambda d: d.assign(below=d.avg<2.5)).groupby('library').below.agg(['size','sum'])
```

## Evaluation signals

- The recomputed GNPS fraction equals 29/1,746 = 1.7%, matching the article-reported value within rounding tolerance.
- The recomputed NIST fraction equals 10.5% and the MassBank fraction equals 20.1%, within rounding tolerance of reported values.
- The GNPS rated-spectra count equals exactly 1,746 and the below-threshold count equals exactly 29.
- Every rated spectrum's average rating falls within the 1-4 star scale, and the total distinct rated spectra across libraries is consistent with 2,041 library spectra from 3,608 rated matches.
- Output CSV schema contains exactly the columns library, rated_spectra_count, below_threshold_count, fraction.

## Limitations

- The rating scale is subjective crowd feedback (4-star correct to 1-star incorrect), so the fraction below 2.5 stars measures perceived annotation reliability rather than ground-truth correctness.
- Results depend on the exact rating records in Supplementary Table 6/7 (3,608 matches over 2,041 spectra); any mismatch in joining ratings to library spectra or in the library source label changes the counts.
- Only rated spectra are included — unrated library spectra are excluded from the denominator, so fractions refer to rated spectra only.

## Evidence

- [abstract] Crowdsourced curation of community-wide reference MS libraries underpins GNPS annotation reliability: "In GNPS, crowdsourced curation of freely available community-wide reference MS libraries will underpin improved annotations."
- [supplementary] Ratings are per-match user judgments on a 4-star correct to 1-star incorrect scale associated with individual library spectra: [evidence span withheld — anchor status: not_found]
- [intro] MassBank and other third-party MS/MS spectral libraries provide the comparison set for cross-library reliability evaluation: "mass spectrometry databases including Massbank5, Metlin6, mzCloud7, and ReSpect8 host MS/MS spectra"
