---
name: ipdb-theoretical-isotope-profile-computation
description: Use when you need theoretical isotopic-profile libraries (.Rdata IPDB
  files) for MS1-only molecular formula annotation of untargeted LC/HRMS peaks — before
  running formula matching or candidate ranking.
when_to_use_negative:
- Only MS2 (tandem) spectra are available and MS1 precursor isotope envelopes are
  not needed for the annotation goal
- Input is an aligned/annotated peak table with no formula database; use exhaustive
  enumeration (IDSL.UFAx) instead when no formula source is available
- Peaks whose formulas lie outside the 15-element space (C, B, Br, Cl, K, S, Si, N,
  H, As, F, I, Na, O, P) that IDSL.UFAx supports
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: IDSL.UFA
  role: 'Generates and consumes the IPDB: computes theoretical isotopic profiles from
    natural isotope distributions and matches them to experimental MS1 peaks'
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Produces the individual peak lists and aligned peak table whose peaks get
    queried against the IPDB theoretical profiles
  repo: github.com/idslme/IDSL.IPA
- name: IDSL.UFAx
  role: Alternative to IPDB-based matching when formulas must be enumerated exhaustively
    rather than drawn from a database
  repo: https://cran.r-project.org/package=IDSL.UFAx
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:815bec832af15ffa881dec502eb62ece51f176a258ae2bc1c56fd80f0e7bd232
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ipdb-theoretical-isotope-profile-computation

## Summary

Compute pre-calculated theoretical isotopic profiles for molecular formulas (IPDB) so IDSL.UFA can match an LC/HRMS MS1 experimental isotopic envelope against theory, using IUPAC isotopic distribution profiles and elemental composition from a formula database.

## When to use

Use when you need theoretical isotopic-profile libraries (.Rdata IPDB files) for MS1-only molecular formula annotation of untargeted LC/HRMS peaks — before running formula matching or candidate ranking.

## When NOT to use

- Only MS2 (tandem) spectra are available and MS1 precursor isotope envelopes are not needed for the annotation goal
- Input is an aligned/annotated peak table with no formula database; use exhaustive enumeration (IDSL.UFAx) instead when no formula source is available
- Peaks whose formulas lie outside the 15-element space (C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P) that IDSL.UFAx supports

## Inputs

- List of molecular formulas (ion formulas) from a formula database such as IDSL.ExposomeDB IPDB or RefMetDB
- IUPAC atomic mass and natural isotope distribution tables
- Polarity selection (positive or negative ionization mode) determined by the IDSL.IPA workflow output

## Outputs

- IPDB file (.Rdata) containing theoretical isotopic profiles for each ion formula
- Per-formula theoretical m/z and intensity pairs for monoisotopic and isotopologue peaks

## How to apply

Build the IPDB from a formula database (e.g. the IDSL.ExposomeDB IPDB with 209,592 positive-mode and 129,122 negative-mode ion formulas) by computing a theoretical isotopic profile per formula using natural isotope atomic mass tables from IUPAC. The combinatorial isotope-prediction rules are applied to carbon-containing ionized formulas, producing per-hit m/z values and intensities for monoisotopic and M+1/M+2 isotopologues. Store the profiles as an IPDB (.Rdata) object whose address is supplied as PARAM0004 to the IDSL.UFA workflow; choose the positive- or negative-mode IPDB matching the IDSL.IPA polarity used for peak extraction. At runtime, IDSL.UFA queries the theoretical profile of each surviving candidate formula against the experimental isotopic pattern from the individual IPA peak list, feeding Profile Cosine Similarity (PCS) and normalized Euclidean mass error (NEME) scoring, which combine into equation (6) for ranking candidate formulas.

## Related tools

- **IDSL.UFA** (Generates and consumes the IPDB: computes theoretical isotopic profiles from natural isotope distributions and matches them to experimental MS1 peaks) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Produces the individual peak lists and aligned peak table whose peaks get queried against the IPDB theoretical profiles) — github.com/idslme/IDSL.IPA
- **IDSL.UFAx** (Alternative to IPDB-based matching when formulas must be enumerated exhaustively rather than drawn from a database) — https://cran.r-project.org/package=IDSL.UFAx

## Examples

```
library(IDSL.UFA); UFA_workflow('path/to/UFA_parameters.xlsx')
```

## Evaluation signals

- IPDB output loads as an .Rdata object in R and its number of ion formulas matches the input formula count (e.g. 209,592 positive-mode entries from IDSL.ExposomeDB)
- Each entry contains a monoisotopic mass and at least one isotopologue intensity consistent with IUPAC natural abundance ratios for carbon-containing formulas
- When supplied as PARAM0004 to IDSL.UFA, downstream candidate ranking produces rank and score columns in the annotation table
- Top-ranked hits satisfy the paper's reporting thresholds: median rank ≤ 5, top rank ≤ 30, and ≤ 30 hits per peak

## Limitations

- IPDB computation is 'database dependent': formulas absent from the source database will not have theoretical profiles, risking missed exposure compounds poorly represented in biochemical databases
- Isotopic profile accuracy depends on IUPAC atomic mass tables and combinatorial isotope rules; errors propagate to PCS and NEME scoring
- IPDB covers only carbon-containing compounds; profiles for formulas without carbon (and hence no informative C isotope pattern) may be less discriminative

## Evidence

- [readme] Generating comprehensive in-silico theoretical libraries (known as IPDB) using natural isotopic distribution profiles: [evidence span withheld — anchor status: external_doc]
- [readme] IUPAC isotopic pattern rules for theoretical profiles: [evidence span withheld — anchor status: external_doc]
- [results] Score combines PCS, RCS, NEME and R13C for ranking: [evidence span withheld — anchor status: not_found]
- [readme] PARAM0004 supplies IPDB address to the workflow: [evidence span withheld — anchor status: external_doc]
