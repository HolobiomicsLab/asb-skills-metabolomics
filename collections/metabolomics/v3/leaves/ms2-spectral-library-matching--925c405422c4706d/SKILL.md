---
name: ms2-spectral-library-matching
description: Use when tissue MS2 spectra (e.g. from mzML files) need to be compared
  against reference MS2 spectra of chemical standards — such as FMP-10-derivatized
  neurotransmitters — to annotate on-tissue metabolites or quantify bin-size sensitivity
  of similarity scores.
when_to_use_negative:
- The task is MS1-level mass matching or m/z feature annotation without tandem MS
  data — use Met-ID's identify() mass matching instead
- The spectra come from an underivatized workflow where reference MS2 entries do not
  exist in Met-ID's preconfigured database of FMP-10-derivatized standards
- Input is already a feature/annotation table with no raw MS2 peak lists to match
tools:
- name: Met-ID
  role: Provides the MS2 spectral database of FMP-10-derivatized standards, the local-server
    mzML/imzML reader, and the target of the cosine-similarity comparison workflow
  repo: https://github.com/pbjarterot/Met-ID/releases
- name: Python
  role: Implements binning of spectra onto a common m/z grid and cosine-similarity
    computation across spectrum pairs
- name: pandas
  role: Writes the pairwise similarity scores annotated with bin size to CSV output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:b2e9b55f375aac1214865dd8520ece8901292e9c5dd9261f3b4e8e596ed06041
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  - Python
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ms2-spectral-library-matching

## Summary

Quantify similarity between user-provided tissue MS2 spectra and Met-ID's built-in MS2 spectral database of FMP-10-derivatized standards using bin-size-parameterized cosine similarity on binned intensity vectors, exposing how binning resolution (e.g. 0.01 Da vs 1 Da) affects the scores.

## When to use

Use when tissue MS2 spectra (e.g. from mzML files) need to be compared against reference MS2 spectra of chemical standards — such as FMP-10-derivatized neurotransmitters — to annotate on-tissue metabolites or quantify bin-size sensitivity of similarity scores.

## When NOT to use

- The task is MS1-level mass matching or m/z feature annotation without tandem MS data — use Met-ID's identify() mass matching instead
- The spectra come from an underivatized workflow where reference MS2 entries do not exist in Met-ID's preconfigured database of FMP-10-derivatized standards
- Input is already a feature/annotation table with no raw MS2 peak lists to match

## Inputs

- User-provided tissue MS2 spectra from mzML files
- Met-ID built-in MS2 spectral database of FMP-10-derivatized neurotransmitter and metabolite standards
- User-specified m/z bin size (e.g. 0.01 Da and 1 Da)

## Outputs

- Pairwise MS2 cosine similarity scores annotated with bin size (e.g. ms2_similarity_scores.csv)
- Binned intensity vectors at each tested bin size

## How to apply

Import the tissue MS2 spectra from mzML files using Met-ID's local-server imzML/mzML reading convention (MetIDServer context manager) with a fine m/z tolerance such as 0.01. Bin each spectrum's intensities onto a common m/z grid at the user-specified bin size — computing binned spectra at both 0.01 Da and 1 Da exposes bin-size sensitivity. Represent each binned spectrum as an intensity vector and compute cosine similarity for every spectrum pair, using the FMP-10-derivatized standards in Met-ID's MS2 spectral database as reference entries; identical spectra must score exactly 1.0 and dissimilar pairs near 0. Write the pairwise similarity scores annotated with bin size (e.g. to ms2_similarity_scores.csv) so 0.01 Da and 1 Da results can be compared side by side.

## Related tools

- **Met-ID** (Provides the MS2 spectral database of FMP-10-derivatized standards, the local-server mzML/imzML reader, and the target of the cosine-similarity comparison workflow) — https://github.com/pbjarterot/Met-ID/releases
- **Python** (Implements binning of spectra onto a common m/z grid and cosine-similarity computation across spectrum pairs)
- **pandas** (Writes the pairwise similarity scores annotated with bin size to CSV output)

## Examples

```
import numpy as np; sim = float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))
```

## Evaluation signals

- Cosine similarity of a spectrum against itself must equal exactly 1.0
- Dissimilar spectrum pairs must score near 0
- The output CSV must contain scores for both the 0.01 Da and 1 Da bin settings
- Scores must change across the two bin sizes, quantifying how binning resolution affects the comparison
- Every score must be traceable to a matched reference entry from the FMP-10-derivatized standards database

## Limitations

- Met-ID ships a preconfigured finite number of MS2 spectra; matching is restricted to those reference entries unless the user adds spectra locally
- Changes to the MS2 database only affect the local Met-ID install, not any other installation
- Results are sensitive to the bin size chosen, so a single bin setting may misrepresent similarity

## Evidence

- [other] Met-ID computes MS2 similarity via cosine similarity, calculated by aligning intensities at matching m/z values between spectra within the user-specified bin size.: [evidence span withheld — anchor status: not_found]
- [abstract] Comparison of user spectra to database entries: "enables comparison of these spectra with user-provided tissue MS2 spectra for similarity assessment."
- [readme] README on user MS2 input and standards comparison: [evidence span withheld — anchor status: external_doc]
- [discussion] Limitation on preconfigured MS2 spectra: "some limitations exist, particularly regarding the software’s preconfigured finite number of MS2 spectra"
