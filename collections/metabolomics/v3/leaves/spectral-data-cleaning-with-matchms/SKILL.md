---
name: spectral-data-cleaning-with-matchms
description: Use when raw MGF/mzML spectra are about to be matched against a spectral
  library (e.g. with MS2Query or modified cosine), and peak lists and metadata still
  need harmonization and noise filtering.
when_to_use_negative:
- Spectra have already been peak-picked, clustered, and reduced per feature (e.g.
  FBMN output from MZMine) and only need searching, not re-cleaning
- The task is raw LC-MS feature detection/peak picking on profile data — MS2Query/matchms
  cleaning does not do peak picking or clustering of similar MS2 spectra
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Performs metadata cleaning, intensity normalization, and peak filtering (>1000
    Da cutoff, 0.1% relative-intensity cutoff, <3-peak spectra removal) on library
    and query spectra
  repo: https://github.com/iomega/ms2query
- name: MS2Query
  role: Downstream consumer of the cleaned spectra; runs MS2Deepscore-based analogue/exact-match
    search on the preprocessed peak lists
  repo: https://github.com/iomega/ms2query
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:ad002f10f53fdf1f2e1ee68ecd68171e3e2e3215102cc52cb9cf559ff24c2c7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Query
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spectral-data-cleaning-with-matchms

## Summary

Standardized preprocessing of MS2 mass spectra (library and query) with matchms before analogue/exact-match searching, including metadata harmonization, intensity normalization, and peak filtering. Clean spectra are required so downstream tools like MS2Query operate on consistent, low-noise peak lists.

## When to use

Use when raw MGF/mzML spectra are about to be matched against a spectral library (e.g. with MS2Query or modified cosine), and peak lists and metadata still need harmonization and noise filtering.

## When NOT to use

- Spectra have already been peak-picked, clustered, and reduced per feature (e.g. FBMN output from MZMine) and only need searching, not re-cleaning
- The task is raw LC-MS feature detection/peak picking on profile data — MS2Query/matchms cleaning does not do peak picking or clustering of similar MS2 spectra

## Inputs

- Raw MS2 spectra files (MGF, mzML, mzXML, msp, json, USI, or pickled matchms Spectra objects)
- GNPS library spectra (e.g. ALL_GNPS_NO_PROPOGATED.mgf)
- Query spectra from experimental case studies (e.g. NIST 1950 plasma, 139 positive-mode spectra)

## Outputs

- Cleaned, intensity-normalized, ionmode-split Spectra objects ready for library matching
- Preprocessed library spectra and metadata (uniform format) as consumed by MS2Query
- Cleaned peak lists with peaks >1000 Da and <0.1% relative intensity removed, max 500 peaks per spectrum

## How to apply

Load the spectra (mgf, mzML, json, msp, mzxml, usi, or pickled matchms objects) and clean them with matchms before any library search. Remove peaks above 1000 Da and peaks with intensity below 0.1% of the highest peak; for spectra with more than 500 peaks, remove the lowest-intensity peaks to cap peak count. Remove spectra with fewer than 3 peaks entirely, normalize peak intensities, and clean the metadata to a uniform format (correcting or removing misplaced metadata). Split the cleaned dataset by ionization mode (positive/negative) so only matching-mode spectra are searched. The rationale is that MS2Query itself does no peak picking or clustering, so clean, normalized input is a prerequisite for reliable MS2Deepscore embeddings and re-ranking.

## Related tools

- **matchms** (Performs metadata cleaning, intensity normalization, and peak filtering (>1000 Da cutoff, 0.1% relative-intensity cutoff, <3-peak spectra removal) on library and query spectra) — https://github.com/iomega/ms2query
- **MS2Query** (Downstream consumer of the cleaned spectra; runs MS2Deepscore-based analogue/exact-match search on the preprocessed peak lists) — https://github.com/iomega/ms2query

## Evaluation signals

- No spectrum in the cleaned set has fewer than 3 peaks
- No peak exceeds 1000 Da and no retained peak has intensity below 0.1% of the spectrum's highest peak
- Spectra with more than 500 peaks have been trimmed to their highest-intensity peaks
- Metadata fields are in a uniform format with misplaced metadata corrected or removed, and spectra are correctly split by ionization mode
- Downstream sanity check: running MS2Query on the cleaned NIST 1950 plasma spectra reproduces 75 matches from 139 spectra, 72 of which are lipid-class matches

## Limitations

- Thresholds (1000 Da peak cutoff, 0.1% relative intensity, 3-peak minimum, 500-peak cap) reflect this paper's benchmarking settings and may need adjustment for other instruments or compound classes
- MS2Query does no peak picking or clustering, so heavily redundant MS2 spectra per feature should be reduced upstream before cleaning/searching
- Software versions for matchms were not stated in the article, so exact filter behavior may vary between matchms releases

## Evidence

- [discussion] Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed.: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [discussion] Spectra with less than 3 peaks were completely removed from the library.: "Spectra with less than 3 peaks were completely removed from the library."
- [discussion] For spectra with more than 500 peaks, the peaks with the lowest intensities were removed.: "For spectra with more than 500 peaks, the peaks with the lowest intensities were removed."
- [intro] filtering out spectra with less than three peaks and normalizing intensities: "filtering out spectra with less than three peaks and normalizing intensities"
- [readme] MS2Query does not do any peak picking or clustering of similar MS2 spectra.: [evidence span withheld — anchor status: external_doc]
