---
name: user-rating-quality-assessment
description: Use when evaluating spectral-library reliability from user match-rating
  records, e.g. comparing fraction of rated library spectra below 2.5 average stars
  across GNPS-Collection, GNPS-Community, NIST, and MassBank.
when_to_use_negative:
- Input is raw MS/MS query spectra or spectral search scores rather than existing
  user match-rating records — there are no star ratings to aggregate.
- The goal is spectral-match scoring itself (normalized dot product / bipartite peak
  matching) rather than post-hoc quality assessment of library spectra.
- Ratings come from a non-star scale or lack a defined correct/incorrect polarity,
  making a 2.5-star threshold meaningless.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3361
tools:
- name: GNPS
  role: Source of the community-curated spectral libraries and the user match-rating
    records being assessed
- name: pandas
  role: Used to load the rating records, group by library spectrum, compute per-spectrum
    average star ratings, and threshold at 2.5 stars
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:e91fd5e24ab7f2b75a86b15f10414eb0ed924708bd88467994412f7ce85e23d8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# user-rating-quality-assessment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Quantifies perceived spectral-library annotation quality by aggregating crowd-sourced user match ratings (4-star correct to 1-star incorrect) per library spectrum and computing the fraction of rated spectra with average ratings below a 2.5-star threshold, benchmarked across GNPS, NIST, and MassBank libraries.

## When to use

Use when evaluating spectral-library reliability from user match-rating records, e.g. comparing fraction of rated library spectra below 2.5 average stars across GNPS-Collection, GNPS-Community, NIST, and MassBank.

## When NOT to use

- Input is raw MS/MS query spectra or spectral search scores rather than existing user match-rating records — there are no star ratings to aggregate.
- The goal is spectral-match scoring itself (normalized dot product / bipartite peak matching) rather than post-hoc quality assessment of library spectra.
- Ratings come from a non-star scale or lack a defined correct/incorrect polarity, making a 2.5-star threshold meaningless.

## Inputs

- User match-rating records table (3,608 rated matches linked to 2,041 library spectra, from Supplementary Table 6/7)
- Library-spectrum to source-library mapping (GNPS-Collection/GNPS-Community, NIST, MassBank) with star-rating scale 1–4

## Outputs

- library_rating_fractions.csv with columns: library, rated_spectra_count, below_threshold_count, fraction

## How to apply

Load the user match-rating records (3,608 rated matches associated with 2,041 library spectra, per Supplementary Table 6/7 in the SI) into a pandas DataFrame. Group ratings by library spectrum and compute the average star rating per spectrum on the 4-star (correct) to 1-star (incorrect) scale. Split spectra by source library (GNPS-Collection/GNPS-Community: 1,746 rated spectra; NIST; MassBank) and count those with average ratings below 2.5 stars. Divide by the number of rated spectra per library to obtain fractions, expecting 29/1,746 = 1.7% for GNPS, 10.5% for NIST, and 20.1% for MassBank. The rationale is that crowdsourced curation of community-wide reference MS libraries underpins improved annotations in GNPS, so per-spectrum consensus ratings serve as a direct quality proxy.

## Related tools

- **GNPS** (Source of the community-curated spectral libraries and the user match-rating records being assessed)
- **pandas** (Used to load the rating records, group by library spectrum, compute per-spectrum average star ratings, and threshold at 2.5 stars)

## Examples

```
avg = df.groupby(['library','spectrum_id'])['rating'].mean(); frac = avg.lt(2.5).groupby(level=0).mean()
```

## Evaluation signals

- Recomputed fraction for GNPS libraries must equal 29/1,746 = 1.7% within rounding tolerance.
- Recomputed fractions for NIST and MassBank must match the reported 10.5% and 20.1% respectively within rounding tolerance.
- GNPS rated-spectra count must equal exactly 1,746 and the below-threshold count must equal exactly 29.
- Total rated matches processed must equal 3,608 across 2,041 library spectra, matching Supplementary Table 6/7.
- Output CSV must contain exactly the columns library, rated_spectra_count, below_threshold_count, fraction with one row per source library.

## Limitations

- The 1.7%/10.5%/20.1% benchmark values depend on the exact Supplementary Table 6/7 rating extract; a different rating snapshot will not reproduce them.
- Average ratings are crowd-sourced and may not be evenly distributed across spectra, so per-library fractions are sensitive to which spectra received any ratings at all.
- The 2.5-star threshold is a binary cut on a 4-star scale; spectra with ratings near the threshold will flip the counts under small rounding differences.

## Evidence

- [supplementary] Only 29 of 1,746 rated library spectra (1.7%) from the GNPS-Collection and GNPS-Community libraries had average ratings below 2.5 stars, indicating high perceived annotation reliability.: [evidence span withheld — anchor status: not_found]
