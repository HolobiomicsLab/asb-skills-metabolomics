---
name: cosine-similarity-spectral-comparison
description: 'Use when assigning or validating molecular formulas from untargeted
  LC/HRMS MS1 peaks: you have a candidate formula and need to rank or score it by
  how well its theoretical isotopic envelope matches the observed MS1 isotopic profile.'
when_to_use_negative:
- Input is MS2/MSn tandem spectra and the goal is structural or library spectral matching
  — this skill matches MS1 isotopic profiles only ('The IDSL.UFA pipeline only requires
  MS1'); use MS2-based tools instead for structure elucidation.
- Data are low-resolution MS or single-charge spectra lacking a resolvable isotopic
  envelope, so no meaningful theoretical-vs-experimental isotopic profile comparison
  is possible.
- Input is already a fully annotated, formula-assigned feature table and no isotopic-profile
  scoring or validation is needed.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.UFA
  role: Computes theoretical isotopic profiles for molecular formulas (IPDB) and matches
    them against experimental MS1 isotopic profiles via cosine-similarity-based ranking
    to assign formulas
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Upstream peak picking that produces the peaklists and aligned peak table whose
    peaks' isotopic profiles are compared
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Environment in which IDSL.UFA and the NEME/PCS validation computations are
    implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:abe224a6875b5f3d33b248ac3e6003f67363ecb6da4326832217e394e59e4547
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cosine-similarity-spectral-comparison

## Summary

Scores molecular formula candidates by comparing theoretical MS1 isotopic profiles (computed from IUPAC isotopic abundances, as in IDSL.UFA's IPDB) against experimentally integrated isotopic profile spectra across chromatography using pairwise cosine similarity (PCS), and validates the metric against authentic reference standards.

## When to use

Use when assigning or validating molecular formulas from untargeted LC/HRMS MS1 peaks: you have a candidate formula and need to rank or score it by how well its theoretical isotopic envelope matches the observed MS1 isotopic profile.

## When NOT to use

- Input is MS2/MSn tandem spectra and the goal is structural or library spectral matching — this skill matches MS1 isotopic profiles only ('The IDSL.UFA pipeline only requires MS1'); use MS2-based tools instead for structure elucidation.
- Data are low-resolution MS or single-charge spectra lacking a resolvable isotopic envelope, so no meaningful theoretical-vs-experimental isotopic profile comparison is possible.
- Input is already a fully annotated, formula-assigned feature table and no isotopic-profile scoring or validation is needed.

## Inputs

- MS1-level LC/HRMS data files (mzXML, mzML, or netCDF)
- Candidate molecular formula list or IPDB (.Rdata) theoretical isotopic-profile database (e.g., RefMetDB positive/negative mode IPDBs)
- IDSL.IPA-generated peaklists and peak_alignment directories (m/z-RT chromatographic peak information)
- UFA parameter spreadsheet (UFA_parameters.xlsx) with IPDB address, HRMS data location, and peaklist/alignment directories (PARAM0004, PARAM0009, PARAM0011, PARAM0012, PARAM0014)
- For validation: MSV000088661 authentic reference standards dataset and deposited IDSL.UFA output (Zenodo 5803968)

## Outputs

- Ranked candidate molecular formula hits per peak with isotopic-profile match scores
- Per-compound PCS (pairwise cosine similarity, %) and NEME (mDa) table for validation standards
- Aggregate validation statistics (mean NEME ~0.70 mDa, mean PCS ~99.968% over 367 standards)
- Aligned annotated peak table summarizing formula annotations by detection frequency and median rank

## How to apply

For each candidate molecular formula, generate its theoretical isotopic profile (IDSL.UFA builds these as IPDB .Rdata libraries using natural isotopic distributions from IUPAC atomic mass tables) and extract the experimental integrated isotopic profile spectra across chromatography from the MS1 data. Compute the pairwise cosine similarity (PCS) between the theoretical and experimental profiles as the match score; IDSL.UFA combines this isotopic-profile matching with mass-error criteria to rank formula hits for peaks in IDSL.IPA peaklists. To validate the pipeline, run it on the MSV000088661 authentic reference standards dataset and compute per-compound NEME (monoisotopic mass error in mDa) and PCS, then aggregate means across compounds. Successful application reproduces an average NEME close to 0.70 mDa and an average PCS close to 99.968% across the 367 standards, with reference theoretical/experimental spectra available at Zenodo record 5803968.

## Related tools

- **IDSL.UFA** (Computes theoretical isotopic profiles for molecular formulas (IPDB) and matches them against experimental MS1 isotopic profiles via cosine-similarity-based ranking to assign formulas) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Upstream peak picking that produces the peaklists and aligned peak table whose peaks' isotopic profiles are compared) — https://github.com/idslme/IDSL.IPA
- **R** (Environment in which IDSL.UFA and the NEME/PCS validation computations are implemented)

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Aggregate mean PCS across the 367 MSV000088661 authentic standard compounds is close to 99.968%.
- Aggregate mean NEME across the same standards is close to 0.70 mDa.
- Per-compound PCS values fall near 1.0 (perfect profile overlap) for correctly assigned formulas on high-quality Orbitrap MS1 data.
- Recomputed theoretical/experimental spectra match the deposited reference spectra at Zenodo record 5803968.
- Formula ranks reported in the aligned annotated peak table are consistent with a median rank ≤ 5 for reproducible annotations.

## Limitations

- Isotopic-profile matching uses only MS1, so the top-ranked formula is not guaranteed to be the true formula — in validation, IDSL.UFA assigned the correct formula as the top hit for only 54.31–85.51% of true positives (90.58–100% within the top five hits).
- Theoretical profiles for carbon-containing compounds depend on combinatorial isotope rules and IUPAC atomic mass tables; assignments rely on instrument mass accuracy (NEME reported in mDa), so low-resolution or poorly calibrated data degrade PCS/NEME.
- Versions of IDSL.UFA/IPA and reproducibility parameters were not explicitly stated in all sections of the article.

## Evidence

- [results] Across 367 authentic standard compounds, IDSL.UFA achieved an average NEME of 0.70 mDa and an average PCS of 99.968%: [evidence span withheld — anchor status: not_found]
- [abstract] molecular formulas may be assigned through matching theoretical and observed isotopic profiles: "molecular formulas may be assigned to LC/HRMS peaks through matching theoretical and observed isotopic profiles (MS1) of the underlying ionized compound"
- [results] IDSL.UFA computes theoretical isotopic profiles for molecular formulas, matches theoretical isotopic profiles against experimental LC/HRMS data: "IDSL.UFA computes theoretical isotopic profiles for molecular formulas, matches theoretical isotopic profiles against experimental LC/HRMS data in individual data file"
- [readme] annotate peaklists from the IDSL.IPA package with molecular formula using an isotopic profile matching approach: [evidence span withheld — anchor status: external_doc]
- [results] The theoretical and experimental integrated isotopic profile spectra across chromatography for these standards are provided at Zenodo repository accession: "The theoretical and experimental integrated isotopic profile spectra across chromatography for these standards are provided at Zenodo repository accession (https://zenodo.org/record/5803968)"
