---
name: spectral-library-decoy-preparation
description: Use when building or extending a spectral library (e.g. NIST human HCD,
  Selevsek yeast, MassIVE-KB) for ANN-SoLo library searching and target-decoy FDR
  control at 1% is needed.
when_to_use_negative:
- The input library already contains decoy spectra — re-adding shuffle-and-reposition
  decoys would corrupt the 1:1 target:decoy ratio; instead remove existing decoy-protein
  matches first
- Running a sequence database search (e.g. MSFragger against neXtProt + cRAP) rather
  than spectral library searching — decoy preparation here applies to spectral library
  entries, not protein FASTA sequences
- Only unmodified-peptide identification is needed and no FDR-controlled open search
  will be performed
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ANN-SoLo
  role: Search engine consuming the target-decoy library; uses decoy matches in the
    target-decoy strategy to compute FDRs after both cascade levels
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Concatenates spectral libraries while removing duplicate peptide-ion spectra
    before decoy appending
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:836af7e3f9e4cfd22668be67d543c59bffc9d98176f6f4e46f99ccc1fe48fca2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  - msconvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-library-decoy-preparation

## Summary

Prepare a target spectral library for open modification searching by appending shuffled decoy spectra in a 1:1 target:decoy ratio using the shuffle-and-reposition method, enabling target–decoy FDR estimation of spectrum-spectrum matches in ANN-SoLo.

## When to use

Use when building or extending a spectral library (e.g. NIST human HCD, Selevsek yeast, MassIVE-KB) for ANN-SoLo library searching and target-decoy FDR control at 1% is needed.

## When NOT to use

- The input library already contains decoy spectra — re-adding shuffle-and-reposition decoys would corrupt the 1:1 target:decoy ratio; instead remove existing decoy-protein matches first
- Running a sequence database search (e.g. MSFragger against neXtProt + cRAP) rather than spectral library searching — decoy preparation here applies to spectral library entries, not protein FASTA sequences
- Only unmodified-peptide identification is needed and no FDR-controlled open search will be performed

## Inputs

- Target spectral library file (e.g. NIST human HCD v2016/09/12, Selevsek TripleTOF yeast library, MassIVE-KB v2017/11/27)
- Query spectra in MGF format converted from raw files with msconvert (e.g. PRIDE PXD001468 HEK293 raw files)
- SpectraST (Trans-Proteomic Pipeline 5.1.0, v5.0) for duplicate removal and library concatenation

## Outputs

- Combined target-decoy spectral library in 1:1 ratio with shuffle-and-reposition decoys (e.g. 3,009,902 spectra for the MassIVE-KB human HCD library)
- List of confident SSMs at 1% FDR after target-decoy filtering
- CSV of per-search identifications (e.g. iprg2012_open_search_identifications.csv)

## How to apply

First clean the source library: remove matches to decoy proteins (e.g. from the Selevsek yeast library) and remove duplicate peptide-ion spectra with SpectraST when concatenating libraries (e.g. NIST human HCD v2016/09/12 + yeast, or MassIVE-KB human HCD, keeping only the best replicate spectrum per peptide ion). Then append decoy spectra in a 1:1 ratio with the target spectra using the shuffle-and-reposition method, which shuffles peak intensities and repositions the highest-intensity peaks to preserve spectral character while breaking peptide identity. The resulting combined target-decoy library (e.g. 1,188,168 spectra for iPRG2012 or 3,009,902 spectra for HEK293) is preprocessed identically to query spectra — precursor peak removal, dropping noise peaks below 1% of the base peak, keeping the 50 most intense peaks, discarding spectra with fewer than 10 peaks or a mass range under 250 Da, and rank-transforming intensities. After the cascade search, decoy matches feed the target-decoy strategy to compute FDRs, including subgroup FDR estimation where level-2 SSMs are grouped by charge-normalized precursor mass difference in 0.1 Da bins and only subgroups with at least 20 SSMs are separately filtered.

## Related tools

- **ANN-SoLo** (Search engine consuming the target-decoy library; uses decoy matches in the target-decoy strategy to compute FDRs after both cascade levels) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Concatenates spectral libraries while removing duplicate peptide-ion spectra before decoy appending)

## Evaluation signals

- The final library contains decoy spectra in an exact 1:1 ratio with target spectra (e.g. 1,188,168 total spectra for iPRG2012; 3,009,902 for HEK293 MassIVE-KB)
- All duplicate peptide-ion spectra are removed when concatenating libraries — each peptide ion is represented by only its best replicate spectrum
- Every retained SSM passes the 1% FDR threshold from the target-decoy estimate, including subgroup FDRs computed only for groups with at least 20 SSMs (smaller groups pooled into a residual group)
- Preprocessing invariants hold on the final library: ≤50 peaks per spectrum, ≥10 peaks per spectrum, mass range ≥250 Da, precursor peak and sub-1% base-peak intensity peaks removed, rank-transformed intensities

## Limitations

- The actual FDR might be underestimated when too-small subgroups are used for subgroup FDR estimation
- Accurate FDR estimation for open searches remains an unresolved research question; alternative approaches may be needed
- Decoy spectra generated by shuffle-and-reposition should have decoy-protein matches removed from source libraries first to avoid decoy contamination

## Evidence

- [methods] Decoys are appended 1:1 with the shuffle-and-reposition method: "decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method [41]"
- [methods] FDRs are computed via target-decoy strategy after the cascade search: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [methods] Subgroup FDR filtering only for subgroups with enough SSMs: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [methods] Duplicates removed with SpectraST during library concatenation: "both spectral libraries were concatenated using SpectraST [42] while removing duplicates"
- [readme] ANN-SoLo combines cascade search with strict FDR control: "This is combined with a cascade search strategy to maximize the number of identified unmodified and modified spectra while strictly controlling the false discovery rate"
