---
name: spectral-intensity-binning
description: Use when user-provided tissue MS2 spectra (e.g., from mzML files) must
  be compared against reference standard spectra via cosine similarity, and the effect
  of m/z bin size on those scores must be quantified.
when_to_use_negative:
- The task is MS1 mass matching or m/z annotation of tissue features rather than MS2
  fragment-intensity pattern comparison
- Spectra are already binned on a shared m/z grid, making re-binning redundant
- The comparison targets adduct/mass lookup via Met-ID's identify() or predict_ccs()
  workflows instead of MS2 spectral similarity
tools:
- name: Met-ID
  role: Supplies the MS2 spectral database of FMP-10-derivatized standards that binned
    tissue spectra are compared against; supports user-uploaded spectra for similarity
    comparison
  repo: github.com/pbjarterot/Met-ID
- name: Python
  role: Implements the binning of intensities onto the common m/z grid and the cosine-similarity
    computation over spectrum pairs
- name: pandas
  role: Holds binned intensity vectors and writes the bin-size-annotated pairwise
    scores to ms2_similarity_scores.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:44244efc6ec2aecca6e70a08da446526db72bf04061888151dcd829b5ee4f763
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
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spectral-intensity-binning

## Summary

Bins MS2 spectrum intensities onto a common m/z grid at a user-specified bin size (e.g., 0.01 Da and 1 Da) and computes cosine similarity between tissue MS2 spectra and Met-ID's MS2 spectral database of FMP-10-derivatized standards, quantifying how binning resolution affects the similarity scores.

## When to use

Use when user-provided tissue MS2 spectra (e.g., from mzML files) must be compared against reference standard spectra via cosine similarity, and the effect of m/z bin size on those scores must be quantified.

## When NOT to use

- The task is MS1 mass matching or m/z annotation of tissue features rather than MS2 fragment-intensity pattern comparison
- Spectra are already binned on a shared m/z grid, making re-binning redundant
- The comparison targets adduct/mass lookup via Met-ID's identify() or predict_ccs() workflows instead of MS2 spectral similarity

## Inputs

- User-provided tissue MS2 spectra from mzML files (read with a fine m/z tolerance, e.g., 0.01 Da)
- Met-ID's built-in MS2 spectral database of FMP-10-derivatized neurotransmitter and metabolite chemical standards
- User-specified bin size(s), e.g., 0.01 Da and 1 Da

## Outputs

- ms2_similarity_scores.csv containing pairwise cosine similarity scores annotated with bin size
- Binned intensity vectors at each bin-size setting (0.01 Da and 1 Da)

## How to apply

Import the tissue MS2 spectra from mzML files following Met-ID's local-server imzML/mzML reading convention with a fine m/z tolerance (e.g., 0.01 Da). Bin each spectrum's intensities onto a common m/z grid at the user-specified bin size, generating binned spectra at both 0.01 Da and 1 Da to expose bin-size sensitivity. Represent each binned spectrum as an intensity vector and compute cosine similarity for every spectrum pair, using the FMP-10-derivatized standards in Met-ID's MS2 spectral database as reference entries. Verify the defining properties — identical spectra score exactly 1.0 and dissimilar spectra score near 0 — then compare scores across the two bin sizes, since coarser 1 Da bins merge fragment peaks and change the similarity values. Write the pairwise scores, annotated with bin size, to ms2_similarity_scores.csv.

## Related tools

- **Met-ID** (Supplies the MS2 spectral database of FMP-10-derivatized standards that binned tissue spectra are compared against; supports user-uploaded spectra for similarity comparison) — github.com/pbjarterot/Met-ID
- **Python** (Implements the binning of intensities onto the common m/z grid and the cosine-similarity computation over spectrum pairs)
- **pandas** (Holds binned intensity vectors and writes the bin-size-annotated pairwise scores to ms2_similarity_scores.csv)

## Examples

```
import numpy as np; V = binned.to_numpy().T; S = (V @ V.T) / (np.linalg.norm(V, axis=1)[:, None] * np.linalg.norm(V, axis=1))
```

## Evaluation signals

- Cosine similarity of an identical spectrum against itself equals exactly 1.0
- Dissimilar spectrum pairs score near 0
- The output CSV contains similarity scores for both the 0.01 Da and 1 Da bin settings
- Scores are symmetric across pairs and bounded in [0, 1] for non-negative binned intensities
- Scores differ between the 0.01 Da and 1 Da bin settings, demonstrating bin-size sensitivity

## Limitations

- Met-ID ships a preconfigured, finite number of MS2 spectra, so reference coverage is limited
- Changes to the MS2 database only affect that specific install of Met-ID and not any other
- Binning resolution materially affects scores — 1 Da bins merge fragment peaks that 0.01 Da bins resolve
- No automated MS2 collection for MALDI-MSI is available; this was noted as future work

## Evidence

- [abstract] Met-ID enables comparison of binned tissue MS2 spectra to standards: "enables comparison of these spectra with user-provided tissue MS2 spectra for similarity assessment."
- [abstract] Met-ID's built-in MS2 database of FMP-10-derivatized standards serves as the reference entries: "Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10"
- [other] Intensities are binned onto a common m/z grid at user-specified bin size, e.g., 0.01 Da and 1 Da: [evidence span withheld — anchor status: not_found]
- [readme] Met-ID provides MS2 comparison tooling for unknowns vs standards: [evidence span withheld — anchor status: external_doc]
- [readme] Base Met-ID ships MS2 spectra of FT-ICR standards with FMP-10 matrix: [evidence span withheld — anchor status: external_doc]
