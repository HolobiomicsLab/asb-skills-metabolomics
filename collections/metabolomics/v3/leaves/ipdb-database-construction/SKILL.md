---
name: ipdb-database-construction
description: Use when you have a chemical-space list (e.g. RefMet, LIPID MAPS, CompTox)
  and need a formula-to-theoretical-isotope-profile library for MS1-only formula annotation.
when_to_use_negative:
- When raw LC/HRMS data (mzXML, mzML, netCDF) still need peak picking — first run
  the IDSL.IPA workflow to generate peaklists and aligned peak tables; IPDB construction
  happens before, not after, peak annotation.
- When the analysis goal is exhaustive elemental composition de novo enumeration (e.g.
  <10^27 formula space) — use IDSL.UFAx instead of building an IPDB from a finite
  formula source.
- When molecular formula assignment requires MS2 fragmentation spectra (DDA/DIA/MS^E)
  — IPDB matching is MS1-only.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: IDSL.UFA
  role: R package whose UFA_workflow function consumes the IPDB (.Rdata) via PARAM0004
    to annotate IDSL.IPA peaklists by isotope-profile matching
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.IPA
  role: Upstream peak-picking package that produces the mzXML/mzML/netCDF-derived
    peaklists and aligned peak tables the IPDB profiles are matched against
  repo: https://github.com/idslme/IDSL.IPA
- name: RefMet
  role: Reference metabolite-name source used as a chemical space from which pre-calculated
    IPDB libraries (RefMetDB folder) are generated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:cbb74a4bce89706d39cfea8110c98833b24d9ecbbefa5f7cdc0884ab19698edb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.UFA
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ipdb-database-construction

## Summary

Constructs the IDSL.UFA Isotopic Profile DataBase (IPDB) — a pre-computed library of in-silico theoretical isotopic distribution profiles for molecular formula sources — so that MS1-only LC/HRMS peak lists can later be annotated with molecular formulas by isotope-profile matching.

## When to use

Use when you have a chemical-space list (e.g. RefMet, LIPID MAPS, CompTox) and need a formula-to-theoretical-isotope-profile library for MS1-only formula annotation.

## When NOT to use

- When raw LC/HRMS data (mzXML, mzML, netCDF) still need peak picking — first run the IDSL.IPA workflow to generate peaklists and aligned peak tables; IPDB construction happens before, not after, peak annotation.
- When the analysis goal is exhaustive elemental composition de novo enumeration (e.g. <10^27 formula space) — use IDSL.UFAx instead of building an IPDB from a finite formula source.
- When molecular formula assignment requires MS2 fragmentation spectra (DDA/DIA/MS^E) — IPDB matching is MS1-only.

## Inputs

- Chemical-space source list of molecular formulas (e.g. RefMet reference metabolite names, LIPID MAPS, CompTox Chemistry Dashboard)
- IUPAC atomic mass tables and natural isotope abundance data
- Target instrument mass-accuracy window / resolving-power specification (e.g. Thermo Q Exactive HF Orbitrap)
- Ionization mode designation (positive or negative ESI)

## Outputs

- IPDB (.Rdata) isotopic profile database keyed by molecular formula
- Pre-calculated IPDB folders split by ionization mode (positive/negative) for distribution (e.g. Zenodo record 7512923)

## How to apply

Enumerate molecular formulas from the prioritized chemical-space source (e.g. RefMetDB) and for each formula compute the theoretical isotopic profile using natural isotope abundances and atomic masses from the IUPAC atomic mass tables, following the combinatorial rules described for IDSL.UFA. Merge adjacent isotopologue m/z entries that fall within the instrument mass-accuracy window into centroid-level peaks so the profile reflects the resolving power of the HRMS instrument (e.g. Thermo Q Exactive HF Orbitrap). Store the per-formula centroid profiles as an IPDB object (.Rdata), which IDSL.UFA loads via the UFA parameter spreadsheet PARAM0004 'Address of the IPDB (.Rdata)'. Select either the positive or negative mode IPDB according to the IDSL.IPA ionization mode of the target data. Validate that each formula's profile retains the monoisotopic peak as the most intense entry and that all relative intensities are normalized within the profile before distributing the library.

## Related tools

- **IDSL.UFA** (R package whose UFA_workflow function consumes the IPDB (.Rdata) via PARAM0004 to annotate IDSL.IPA peaklists by isotope-profile matching) — https://github.com/idslme/IDSL.UFA
- **IDSL.IPA** (Upstream peak-picking package that produces the mzXML/mzML/netCDF-derived peaklists and aligned peak tables the IPDB profiles are matched against) — https://github.com/idslme/IDSL.IPA
- **RefMet** (Reference metabolite-name source used as a chemical space from which pre-calculated IPDB libraries (RefMetDB folder) are generated)

## Examples

```
Rscript -e 'library(IDSL.UFA); UFA_workflow("UFA_parameters.xlsx")'
```

## Evaluation signals

- Each molecular formula's stored profile retains the monoisotopic peak as its most intense entry.
- Relative isotopic intensities within each formula's profile are normalized (sum to 1).
- The IPDB loads successfully into IDSL.UFA via the PARAM0004 parameter field and matching proceeds without missing-formula errors.
- Positive-mode and negative-mode IPDB variants correspond one-to-one with the ionization mode used in the downstream IDSL.IPA run.
- Row counts of theoretical profiles match the unique formula count from the source chemical-space list (e.g. RefMetDB folder contents).

## Limitations

- The IPDB only encodes MS1 isotopic profiles; it cannot resolve formulas with degenerate isotope patterns, and formula annotation is limited to carbon-containing compounds whose theoretical isotopic profile can be queried against MS1 data.
- IPDB quality is bounded by the accuracy of the natural isotope abundance tables and the chosen mass-accuracy window; profiles must be regenerated when switching to an instrument with substantially different resolving power.
- IPDBs are static, pre-calculated libraries — new chemical spaces require re-running the library-generation step rather than incremental updates.

## Evidence

- [readme] Generating comprehensive in-silico theoretical libraries (known as IPDB) using natural isotopic distribution profiles: [evidence span withheld — anchor status: external_doc]
- [readme] The isotopic pattern for a chemical structure can be accurately predicted using a set of combinatorial rules that uses atomic mass tables provided by the IUPAC: [evidence span withheld — anchor status: external_doc]
- [readme] download these pre-calculated IPDBs and use positive or negative mode IPDB from RefMetDB folder according to the IDSL.IPA folder results: [evidence span withheld — anchor status: external_doc]
- [readme] PARAM0004 for the Address of the IPDB (.Rdata): [evidence span withheld — anchor status: external_doc]
- [readme] UFA_workflow("Address of the UFA parameter spreadsheet"): [evidence span withheld — anchor status: external_doc]
