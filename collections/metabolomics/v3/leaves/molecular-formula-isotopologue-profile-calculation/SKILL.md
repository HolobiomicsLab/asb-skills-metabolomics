---
name: molecular-formula-isotopologue-profile-calculation
description: Use when you need to build or extend an isotopic profile database (IPDB)
  from a chemical list — e.g. a PubChem/CompTox/RefMet-style formula source — so that
  LC/HRMS MS1 peaks can be annotated with molecular formulas via theoretical-vs-experimental
  isotopic profile matching.
when_to_use_negative:
- The input is already an IDSL.IPA peaklist or an aligned peak table — at that point
  the IPDB is a pre-computed input (PARAM0004) and the skill that applies is formula
  matching/ranking, not IPDB construction.
- The study requires MS2/MS/MS structural elucidation rather than MS1-only formula
  annotation — IDSL.UFA's IPDB approach explicitly operates only on MS1 isotopic profiles.
- The chemical space needs exhaustive enumeration of novel elemental compositions
  rather than annotation of a known formula list — IDSL.UFAx provides brute-force
  enumeration of <10^27 molecular formulas for the 15 supported elements (C, B, Br,
  Cl, K, S, Si, N, H, As, F, I, Na, O, P) instead of building an IPDB from a curated
  list.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.UFA
  role: R package whose `UFA_workflow` consumes the parameter spreadsheet and the
    pre-built IPDB to match theoretical isotopic profiles against IDSL.IPA peaklists
    and rank candidate molecular formulas
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: upstream peak-picking tool that generates the individual and aligned peaklists
    (m/z-RT chromatographic information) that the IPDB-based annotation is applied
    to
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFAx
  role: companion exhaustive-enumeration tool used when the chemical space must be
    brute-forced rather than annotated against a pre-built IPDB
  repo: https://github.com/idslme/IDSL.UFAx
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:79876e2609c7a25789ebf78825b214e9aab2c5622ccd7b3f2e4a40464c7f461f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - R
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# molecular-formula-isotopologue-profile-calculation

## Summary

Computing the centroid-level theoretical isotopic profile database (IPDB) for a list of molecular formulas — enumerating isotopologue m/z and relative probabilities, merging adjacent peaks within a mass-accuracy window, and storing profiles in a .Rdata IPDB for formula annotation with IDSL.UFA.

## When to use

Use when you need to build or extend an isotopic profile database (IPDB) from a chemical list — e.g. a PubChem/CompTox/RefMet-style formula source — so that LC/HRMS MS1 peaks can be annotated with molecular formulas via theoretical-vs-experimental isotopic profile matching.

## When NOT to use

- The input is already an IDSL.IPA peaklist or an aligned peak table — at that point the IPDB is a pre-computed input (PARAM0004) and the skill that applies is formula matching/ranking, not IPDB construction.
- The study requires MS2/MS/MS structural elucidation rather than MS1-only formula annotation — IDSL.UFA's IPDB approach explicitly operates only on MS1 isotopic profiles.
- The chemical space needs exhaustive enumeration of novel elemental compositions rather than annotation of a known formula list — IDSL.UFAx provides brute-force enumeration of <10^27 molecular formulas for the 15 supported elements (C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P) instead of building an IPDB from a curated list.

## Inputs

- molecular formula list (e.g. SDF file, CompTox/RefMet/ExposomeDB export, compound database CSV)
- elemental isotope masses and natural abundances (IUPAC atomic mass tables / PubChem Periodic Table and Element Pages)
- instrument mass-accuracy window in ppm (e.g. 5 ppm for HRMS)

## Outputs

- isotopic profile database (IPDB) as a .Rdata file containing per-formula centroid m/z, relative intensity, and formula rows
- optional CSV/SQLite export of the same IPDB table keyed by molecular formula
- annotated molecular formula assignments for LC/HRMS MS1 peaks when the IPDB is consumed by IDSL.UFA

## How to apply

Load the compound list (SDF, CompTox/Dashboard export, RefMet or ExposomeDB formula list) with pandas/R and extract the unique molecular formulas. For each formula, enumerate all isotopologue combinations using the natural isotopic masses and abundances from the IUPAC atomic mass tables that IDSL.UFA draws on, calculating the monoisotopic and shifted m/z values and their relative probabilities. Merge adjacent isotopologue m/z entries that fall within the instrument mass-accuracy window (e.g. 5 ppm for HRMS) into single centroid peaks with summed relative intensities, then normalize the relative intensities so they sum to 1 for each formula. Assemble the per-formula centroid profiles into a single IPDB table (m/z, relative intensity, formula) and store it as a .Rdata file (per the IDSL.UFA Quick Batch example, e.g. IPDB_v1.8 from the RefMetDB folder) that the `UFA_workflow` consumes via PARAM0004. Validate that each formula's centroid profile contains the monoisotopic peak as its most intense entry and that row counts match the number of unique input formulas.

## Related tools

- **IDSL.UFA** (R package whose `UFA_workflow` consumes the parameter spreadsheet and the pre-built IPDB to match theoretical isotopic profiles against IDSL.IPA peaklists and rank candidate molecular formulas) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (upstream peak-picking tool that generates the individual and aligned peaklists (m/z-RT chromatographic information) that the IPDB-based annotation is applied to) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFAx** (companion exhaustive-enumeration tool used when the chemical space must be brute-forced rather than annotated against a pre-built IPDB) — https://github.com/idslme/IDSL.UFAx

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Each formula's centroid isotopic profile contains the monoisotopic peak as its most intense entry and the relative intensities sum to 1 within the profile.
- The IPDB row count equals the number of unique molecular formulas in the input compound list (e.g. IDSL.ExposomeDB gave 209,592 and 129,122 ion formulas in positive and negative modes).
- Adjacent isotopologue m/z entries separated by less than the mass-accuracy window (e.g. 5 ppm) are merged into a single centroid peak — no residual sub-window peaks remain in the IPDB.
- Consuming the IPDB in `UFA_workflow` (PARAM0004) yields formula annotations for IDSL.IPA peaks that reproduce the study's sensitivity: 54.31-85.51% true positives as the top hit and 90.58-100% within the top five hits.

## Limitations

- The IPDB approach is database-dependent: it can miss exposure compounds poorly represented in biochemical databases, compounds without transformation products, and compounds filtered out by detection-frequency and intensity thresholds.
- Formula annotation uses only MS1 isotopic profiles; structural identity requires complementary MS2 or library-based methods.
- IDSL.UFA targets 15 elements (C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P); formulas containing other elements cannot be enumerated with this workflow.
- Pre-calculated IPDBs must match the ionization mode and formula source of the study — the Quick Batch example uses positive or negative mode IPDBs from the RefMetDB folder depending on the IDSL.IPA results.

## Evidence

- [readme] IPDB generation method: [evidence span withheld — anchor status: not_found]
- [readme] IPDB usage parameter: [evidence span withheld — anchor status: not_found]
- [readme] IUPAC mass tables: [evidence span withheld — anchor status: not_found]
- [readme] MS1-only scope: [evidence span withheld — anchor status: external_doc]
