---
name: supplementary-table-data-extraction
description: Use when a paper's main text defers quantitative results to supplementary
  tables (e.g., Supplementary Tables 6/7/8/11 in the GNPS SI) and the goal is to reproduce
  or verify a reported number from those tables.
when_to_use_negative:
- The quantity of interest is reported fully in the main text with no supporting SI
  table (e.g., the 83.6% community-library reuse finding), so re-extraction adds nothing.
- The article explicitly defers content to a supplementary table that is not included
  in the available SI (e.g., Supplementary Tables 3, 8, or 11 referenced but missing)
  — the skill cannot fabricate the missing data.
- The raw rating or measurement records themselves are needed rather than pre-aggregated
  SI tables; go to the primary source (e.g., GNPS library spectrum provenance files)
  instead.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GNPS
  role: Source of the library spectra and user match-rating data summarized in the
    supplementary tables; also hosts the underlying library spectrum provenance for
    verification
- name: pandas
  role: Loads the supplementary-table records and performs group-by aggregation of
    star ratings per library spectrum and threshold counting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:307b9c55efe7c6e88b6633164dea3510abc317da13d3480582bbde94bc43b276
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# supplementary-table-data-extraction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Extracts tabular records from a paper's supplementary information (e.g., SI Tables 6/7 user match-rating records of 3,608 rated matches over 2,041 library spectra) into a pandas DataFrame so article-reported statistics can be independently recomputed and validated.

## When to use

Use when a paper's main text defers quantitative results to supplementary tables (e.g., Supplementary Tables 6/7/8/11 in the GNPS SI) and the goal is to reproduce or verify a reported number from those tables.

## When NOT to use

- The quantity of interest is reported fully in the main text with no supporting SI table (e.g., the 83.6% community-library reuse finding), so re-extraction adds nothing.
- The article explicitly defers content to a supplementary table that is not included in the available SI (e.g., Supplementary Tables 3, 8, or 11 referenced but missing) — the skill cannot fabricate the missing data.
- The raw rating or measurement records themselves are needed rather than pre-aggregated SI tables; go to the primary source (e.g., GNPS library spectrum provenance files) instead.

## Inputs

- Supplementary Table 6/7 user match-rating records (3,608 rated matches linked to 2,041 library spectra, star-rating scale 1–4)
- Supplementary table metadata mapping table number to content (e.g., Table 3 = Gold/Silver/Bronze quality levels, Table 8 = exact spectral library search parameters)
- Article-reported target values and denominators (e.g., 29/1,746 = 1.7% below 2.5 stars for GNPS libraries)

## Outputs

- pandas DataFrame of per-entity aggregated metrics (average star rating per library spectrum)
- library_rating_fractions.csv with columns: library, rated_spectra_count, below_threshold_count, fraction

## How to apply

Locate the specific supplementary table(s) cited in the article text for the quantity of interest — here, Supplementary Tables 6/7 hold the 3,608 user match-rating records (4-star correct to 1-star incorrect scale) associated with 2,041 library spectra. Load the records into a pandas DataFrame, group by the entity key (library spectrum), and compute the derived metric (average star rating per spectrum). Split by the article's categorical condition (source library: GNPS-Collection/GNPS-Community = 1,746 rated spectra; NIST; MassBank) and apply the article's threshold (average rating < 2.5 stars) to count and divide for fractions. Finally, compare recomputed values against the article-reported numbers (1.7% GNPS, 10.5% NIST, 20.1% MassBank) within rounding tolerance before writing outputs.

## Related tools

- **GNPS** (Source of the library spectra and user match-rating data summarized in the supplementary tables; also hosts the underlying library spectrum provenance for verification)
- **pandas** (Loads the supplementary-table records and performs group-by aggregation of star ratings per library spectrum and threshold counting)

## Examples

```
df = pd.read_csv('supplementary_table_6_user_match_ratings.csv'); avg = df.groupby('library_spectrum_id')['star_rating'].mean(); below = (avg < 2.5).sum(); frac = below / avg.size
```

## Evaluation signals

- Recomputed fraction of GNPS-Collection/GNPS-Community rated spectra below 2.5 stars equals 29/1,746 = 1.7% within rounding tolerance.
- Recomputed NIST fraction is 10.5% and MassBank fraction is 20.1%, matching article-reported values.
- The GNPS rated-spectra count from the extracted table equals exactly 1,746 and the below-threshold count equals exactly 29.
- Total extracted rating records reconcile with the SI totals: 3,608 rated matches associated with 2,041 library spectra.
- Output CSV schema contains exactly the columns library, rated_spectra_count, below_threshold_count, fraction.

## Limitations

- Several SI resources referenced in the article (Supplementary Tables 3, 8, 11 and Supplementary Note 3) may not be included with the main text, blocking extraction of quality-level definitions, search parameters, or molecular networking parameters.
- Some provenance links in the article are unresolved placeholders (e.g., '(e.g. link)' for library spectrum downloads and '(link)' for the molecular network job), so cross-checking SI numbers against primary GNPS records may not be possible.
- Rounding in the published fractions (e.g., 1.7%) limits exact-match validation; comparisons must allow rounding tolerance.

## Evidence

- [supplementary] Load the user match-rating records (3,608 rated matches associated with 2,041 library spectra, per Supplementary Table 6/7 in the SI) into a pandas DataFrame.: [evidence span withheld — anchor status: not_found]
- [results] Only 29 of 1,746 rated library spectra (1.7%) from the GNPS-Collection and GNPS-Community libraries had average ratings below 2.5 stars: [evidence span withheld — anchor status: not_found]
