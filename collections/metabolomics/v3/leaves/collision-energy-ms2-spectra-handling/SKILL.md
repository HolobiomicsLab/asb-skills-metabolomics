---
name: collision-energy-ms2-spectra-handling
description: Use when comparing user-acquired tissue MS2 spectra from mzML files against
  Met-ID's FMP-10-derivatized standards MS2 database, and you need quantitative cosine-similarity
  scores under explicit bin-size choices.
when_to_use_negative:
- When the analysis goal is MS1-level mass matching or adduct annotation (e.g., identify()
  on m/z values) rather than MS2 spectral similarity scoring.
- When the tissue MS2 spectra were not acquired with a derivatizing-matrix workflow
  compatible with the bundled FMP-10-derivatized standards, since Met-ID's database
  is preconfigured with a finite number of those spectra.
- When spectra are already peak-picked and aligned feature lists rather than raw profile/centroid
  MS2 spectra needing binning onto a common m/z grid.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Met-ID
  role: Provides the bundled MS2 spectral database of FMP-10-derivatized chemical
    standards and the local-server mechanism for reading imzML/mzML data; supports
    uploading user spectra and adding user-generated standard spectra to the local
    database install
  repo: https://github.com/pbjarterot/Met-ID
- name: Python
  role: Implements the binning of MS2 intensities onto a common m/z grid and the cosine-similarity
    computation over spectrum pairs
- name: pandas
  role: Writes and annotates the pairwise similarity scores (per bin size) to ms2_similarity_scores.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:f5f08d01c0fc001c8885ad02fbb48ca722e972b43ddcde47ff9fd53673783574
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - Met-ID
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# collision-energy-ms2-spectra-handling

## Summary

Scores similarity between user-provided tissue MS2 spectra and Met-ID's bundled MS2 spectral database of FMP-10-derivatized chemical standards (collected on an FT-ICR at 0–50 eV collision energies, 1 Da isolation window) using bin-size-parameterized cosine similarity.

## When to use

Use when comparing user-acquired tissue MS2 spectra from mzML files against Met-ID's FMP-10-derivatized standards MS2 database, and you need quantitative cosine-similarity scores under explicit bin-size choices.

## When NOT to use

- When the analysis goal is MS1-level mass matching or adduct annotation (e.g., identify() on m/z values) rather than MS2 spectral similarity scoring.
- When the tissue MS2 spectra were not acquired with a derivatizing-matrix workflow compatible with the bundled FMP-10-derivatized standards, since Met-ID's database is preconfigured with a finite number of those spectra.
- When spectra are already peak-picked and aligned feature lists rather than raw profile/centroid MS2 spectra needing binning onto a common m/z grid.

## Inputs

- User-provided tissue MS2 spectra in mzML files (e.g., from MALDI-MSI of derivatized tissue sections)
- Met-ID's built-in MS2 spectral database of FMP-10-derivatized neurotransmitter and metabolite standards (FT-ICR, 0–50 eV collision energies, 1 Da isolation window)
- User-specified m/z bin size (e.g., 0.01 Da and/or 1 Da)

## Outputs

- ms2_similarity_scores.csv containing pairwise cosine similarity scores between tissue MS2 spectra and database standards, annotated with bin size (0.01 Da and 1 Da)

## How to apply

Import the tissue MS2 spectra from mzML files using Met-ID's local-server reading convention (MetIDServer context manager) with a fine m/z tolerance such as 0.01. Bin each spectrum's intensities onto a common m/z grid at a user-specified bin size — run at both 0.01 Da and 1 Da to expose binning sensitivity — and treat each binned spectrum as an intensity vector. Compute cosine similarity for every tissue-vs-reference pair, using the FMP-10-derivatized standards spectra in Met-ID's MS2 database as reference entries; the reference spectra were acquired at collision energies 0–50 eV with a 1 Da isolation window, so pick the closest-matching collision energy when available. Validate that identical spectra yield a cosine similarity of exactly 1.0 and dissimilar pairs score near 0, then write the pairwise scores annotated with bin size to ms2_similarity_scores.csv. Rationale: cosine similarity on matched m/z bins is Met-ID's mechanism for MS2-based metabolite confirmation, and bin size materially changes which fragment peaks align.

## Related tools

- **Met-ID** (Provides the bundled MS2 spectral database of FMP-10-derivatized chemical standards and the local-server mechanism for reading imzML/mzML data; supports uploading user spectra and adding user-generated standard spectra to the local database install) — https://github.com/pbjarterot/Met-ID
- **Python** (Implements the binning of MS2 intensities onto a common m/z grid and the cosine-similarity computation over spectrum pairs)
- **pandas** (Writes and annotates the pairwise similarity scores (per bin size) to ms2_similarity_scores.csv)

## Examples

```
sim = np.dot(binned_tissue, binned_ref) / (np.linalg.norm(binned_tissue) * np.linalg.norm(binned_ref))
```

## Evaluation signals

- Cosine similarity of an identical spectrum pair must equal exactly 1.0.
- Dissimilar spectrum pairs must score near 0.
- ms2_similarity_scores.csv must contain scores for both the 0.01 Da and 1 Da bin settings, with each score annotated by bin size.
- All scores fall in the [0, 1] range for non-negative intensity vectors.
- Comparing the 0.01 Da vs 1 Da results should show sensitivity of scores to binning resolution — large divergence flags that coarse binning merged distinct fragments.

## Limitations

- Met-ID ships a preconfigured, finite number of MS2 spectra, limiting which metabolites can be scored without adding user-generated standard spectra.
- All changes to the MS2 database (e.g., adding user standard spectra) affect only the local install and not any other Met-ID installation.
- No automated MS2 collection for MSI is available; collecting the tissue MS2 spectra to compare remains a manual step (noted as future work by the authors).

## Evidence

- [methods] Met-ID computes MS2 similarity via cosine similarity, calculated by aligning intensities at matching m/z values between spectra within the user-specified bin size.: [evidence span withheld — anchor status: not_found]
- [discussion] The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV): "The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV)"
- [abstract] enables comparison of these spectra with user-provided tissue MS2 spectra for similarity assessment.: "enables comparison of these spectra with user-provided tissue MS2 spectra for similarity assessment."
- [readme] Met-ID allows the user to input MS2 spectra from unknown compounds to compare to standards and also ways to add user generated spectra from chemical standards to the database.: [evidence span withheld — anchor status: external_doc]
- [readme] The base version of Met-ID comes with a number of MS2 spectra collected from chemical standards with an FT-ICR using the FMP-10 chemical matrix.: [evidence span withheld — anchor status: external_doc]
