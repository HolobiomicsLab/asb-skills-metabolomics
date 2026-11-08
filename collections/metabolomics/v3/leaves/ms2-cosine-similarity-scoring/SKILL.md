---
name: ms2-cosine-similarity-scoring
description: Use when a user-provided tissue MS2 spectrum (e.g. exported from mzML)
  must be quantitatively compared against Met-ID's MS2 database of FMP-10-derivatized
  chemical standards and scored with cosine similarity.
when_to_use_negative:
- The task is MS1 mass matching or recalibration against HMDB/LIPID MAPS entries rather
  than MS2 spectral comparison
- The data were acquired without a derivatizing matrix such as FMP-10 and no matching
  standard spectra exist in the database
- Input is already a scored or annotated feature table rather than raw MS2 spectra
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Met-ID
  role: Provides the MS2 spectral database of FMP-10-derivatized standards and defines
    the cosine-similarity comparison against user-provided tissue MS2 spectra
  repo: https://github.com/pbjarterot/Met-ID
- name: Python
  role: Implements the binned-intensity cosine-similarity computation used to reconstruct
    the MS2 matching algorithm
- name: pandas
  role: Tabulates and writes the pairwise, bin-size-annotated similarity scores to
    CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:7bfee208d8eb3886d43529e4e59482fd25d890a86974ae7b347c1433958dc5ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - Met-ID
  - Python
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ms2-cosine-similarity-scoring

## Summary

Quantifies similarity between user-provided tissue MS2 spectra and Met-ID's built-in MS2 spectral database of FMP-10-derivatized standards using bin-size-parameterized cosine similarity, the same metric Met-ID applies for spectral confirmation of derivatized metabolite annotations.

## When to use

Use when a user-provided tissue MS2 spectrum (e.g. exported from mzML) must be quantitatively compared against Met-ID's MS2 database of FMP-10-derivatized chemical standards and scored with cosine similarity.

## When NOT to use

- The task is MS1 mass matching or recalibration against HMDB/LIPID MAPS entries rather than MS2 spectral comparison
- The data were acquired without a derivatizing matrix such as FMP-10 and no matching standard spectra exist in the database
- Input is already a scored or annotated feature table rather than raw MS2 spectra

## Inputs

- User-provided tissue MS2 spectra from mzML files (read with a fine m/z tolerance, e.g. 0.01)
- Met-ID built-in MS2 spectral database of FMP-10-derivatized neurotransmitter and metabolite standards
- User-specified m/z bin size (e.g. 0.01 Da and 1 Da)

## Outputs

- Pairwise cosine-similarity scores between tissue MS2 spectra and database standards, annotated with bin size, written to ms2_similarity_scores.csv

## How to apply

Import the tissue MS2 spectra from mzML files and bin each spectrum's intensities onto a common m/z grid at a user-specified bin size (run both 0.01 Da and 1 Da to expose bin-size sensitivity). Represent each binned spectrum as an intensity vector and compute cosine similarity of the tissue spectrum against every FMP-10-derivatized standard entry in Met-ID's MS2 spectral database, aligning intensities at matching m/z values within the bin size. The bin size is the key parameter: finer bins preserve fragment resolution while coarse bins (1 Da) collapse peaks. Validate the metric by checking that identical spectra yield exactly 1.0 and dissimilar pairs score near 0. Record every pairwise score annotated with its bin size to ms2_similarity_scores.csv.

## Related tools

- **Met-ID** (Provides the MS2 spectral database of FMP-10-derivatized standards and defines the cosine-similarity comparison against user-provided tissue MS2 spectra) — https://github.com/pbjarterot/Met-ID
- **Python** (Implements the binned-intensity cosine-similarity computation used to reconstruct the MS2 matching algorithm)
- **pandas** (Tabulates and writes the pairwise, bin-size-annotated similarity scores to CSV)

## Examples

```
from scipy.spatial.distance import cosine; sim = 1 - cosine(binned_tissue_ms2, binned_standard_ms2)  # bin_size=0.01 or 1.0
```

## Evaluation signals

- Cosine similarity of a spectrum against itself must equal exactly 1.0
- Dissimilar spectrum pairs must score near 0
- Output CSV must contain scores for both the 0.01 Da and 1 Da bin settings
- All scores must lie within the cosine-similarity range [0, 1]
- Scores must differ between the 0.01 Da and 1 Da binnings, demonstrating bin-size sensitivity of the metric

## Limitations

- Met-ID ships a preconfigured, finite number of MS2 spectra, limiting the database to its bundled FMP-10-derivatized standards
- No automated MS2 collection for MSI is available; MS2 spectra of standards were collected with a 1 Da isolation window over 0-50 eV collision energies
- Changes to the MS2 database only affect the local Met-ID install and not any other installation

## Evidence

- [full_text] Met-ID computes MS2 similarity via cosine similarity, calculated by aligning intensities at matching m/z values between spectra within the user-specified bin size.: [evidence span withheld — anchor status: not_found]
- [abstract] enables comparison of these spectra with user-provided tissue MS2 spectra for similarity assessment.: "enables comparison of these spectra with user-provided tissue MS2 spectra for similarity assessment."
- [abstract] Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10: "Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10"
- [readme] Met-ID also includes tools to study MS2 spectra, as a critical part of metabolite identification is to study MS2 spectra of unknown compounds and compare them to those generated by standard.: [evidence span withheld — anchor status: external_doc]
