---
name: reference-spectrum-mplus-peak-extraction
description: Use when an analyte's GC/EI-MS spectrum matched no reference library
  entry and enumerated derivative candidates must be narrowed by molecular mass before
  spectrum/fingerprint similarity ranking.
when_to_use_negative:
- The analyte's structure is already known and matches an established reference spectrum
  in NIST/Wiley/SWGDRUG — PS2MS's enumeration-and-filter pipeline is only invoked
  when no reference match is found.
- The analyte's reference spectrum lacks a determinable M+ peak, so no molecular-weight
  reference can be extracted.
- You need fine-grained structural similarity ranking (SMSF, Jaccard fingerprint similarity)
  rather than coarse candidate narrowing by mass.
edam_operation: http://edamontology.org/operation_3695
tools:
- name: PS2MS
  role: System whose mass-weight filtering step uses the M+ peak as a reference with
    a tolerance range to filter enumerated compounds
  repo: https://github.com/jhhung/PS2MS
- name: rdkit
  role: Used to compute exact masses / molecular properties of enumerated derivatives
    that are compared against the M+ reference mass
  repo: https://www.rdkit.org/docs/Install.html
- name: python 3.6.9
  role: Runtime for running the deep learning tools and the CSV-based mass filtering
    workflow
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:9dbaf8112ae05f58124de18812a11062cde70ab2ed46306f621ac537a29f521f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - PS2MS
  - python
  - python 3.6.9
  - rdkit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reference-spectrum-mplus-peak-extraction

## Summary

Extracts the molecular ion (M+) peak mass from an analyte's SWGDRUG reference spectrum to serve as the reference mass for filtering enumerated NPS derivative candidates within a tolerance range (e.g., mwf ±1 or ±5 Da).

## When to use

Use when an analyte's GC/EI-MS spectrum matched no reference library entry and enumerated derivative candidates must be narrowed by molecular mass before spectrum/fingerprint similarity ranking.

## When NOT to use

- The analyte's structure is already known and matches an established reference spectrum in NIST/Wiley/SWGDRUG — PS2MS's enumeration-and-filter pipeline is only invoked when no reference match is found.
- The analyte's reference spectrum lacks a determinable M+ peak, so no molecular-weight reference can be extracted.
- You need fine-grained structural similarity ranking (SMSF, Jaccard fingerprint similarity) rather than coarse candidate narrowing by mass.

## Inputs

- Analyte reference spectrum from the SWGDRUG MS library (to read the M+ peak mass)
- Enumerated derivative database with precomputed exact masses (CSV, from the enumeration step)
- Tolerance setting (mwf ±1 Da or mwf ±5 Da)

## Outputs

- CSV candidate list of derivatives within ±1 Da of the reference M+ mass
- CSV candidate list of derivatives within ±5 Da of the reference M+ mass
- Summary recording candidate counts per tolerance setting

## How to apply

First retrieve the analyte's reference spectrum from the SWGDRUG MS library (obtained for this study from the Scientific Working Group for the Analysis of Seized Drugs) and read the molecular-ion M+ peak as the mass reference. Then scan the enumerated derivative database, which carries precomputed exact masses from the enumeration step, and retain only candidates whose exact mass falls within the chosen tolerance window of the M+ mass. PS2MS uses two settings: a tight mwf ±1 Da window and a broader mwf ±5 Da window, producing two candidate subsets. Each subset is written to a CSV candidate list, and candidate counts per tolerance setting are recorded in a summary for later ranking. The rationale is that the unknown analyte and its true derivative must be near-isobaric, so the M+ peak anchors a physically meaningful mass constraint before expensive NEIMS/DeepEI similarity calculations.

## Related tools

- **PS2MS** (System whose mass-weight filtering step uses the M+ peak as a reference with a tolerance range to filter enumerated compounds) — https://github.com/jhhung/PS2MS
- **rdkit** (Used to compute exact masses / molecular properties of enumerated derivatives that are compared against the M+ reference mass) — https://www.rdkit.org/docs/Install.html
- **python 3.6.9** (Runtime for running the deep learning tools and the CSV-based mass filtering workflow) — https://www.python.org/downloads/

## Examples

```
python -c "from rdkit import Chem; from rdkit.Chem import Descriptors; keep = abs(Descriptors.ExactMass(Chem.MolFromSmiles(smi)) - mplus) <= 1"
```

## Evaluation signals

- Every candidate retained in a subset has an exact mass within the corresponding window (±1 Da or ±5 Da) of the reference M+ mass.
- Reported candidate counts per tolerance setting equal the row counts of the corresponding output CSV files.
- The reference mass equals the M+ peak extracted from the analyte's SWGDRUG reference spectrum, not an arbitrary or predicted mass.
- The ±5 Da subset is a superset of (or equal to) the ±1 Da subset, and both subsets are drawn only from the enumerated derivative database.

## Limitations

- The tolerance range is a fixed setting (±1 or ±5 Da); candidates outside the window are excluded even if they are true structural analogues with isobaric-but-different molecular formulas.
- The article does not specify version numbers or exact parameter values for PS2MS components, and no explicit counts per tolerance window are reported; validation relies on the window-consistency invariant.
- The method presumes the M+ peak is identifiable in the GC/EI-MS reference spectrum; spectra without a clear molecular ion cannot anchor the filter.

## Evidence

- [discussion] M+ peak as reference mass for tolerance filtering: "utilizes the M+ peak as a reference for the mass weight filtering of enumerated compounds within a certain tolerance range"
- [methods] SWGDRUG as source of reference mass spectra: "we obtained mass spectrum data from the Scientific Working Group for the Analysis of Seized Drugs (SWGDRUG)"
- [methods] Enumeration and rule-based filtering of derivatives precedes spectral comparison: "## Methods > Enumeration And Rule-Based Filtering Of Derivatives"
- [intro] PS2MS invoked only when no reference match is found: "When the mass spectrum of an analyte, retrieved via GC/EI-MS, does not match any established reference spectra during the conventional screening procedure, PS2MS builds a supplementary synthetic NPS"
