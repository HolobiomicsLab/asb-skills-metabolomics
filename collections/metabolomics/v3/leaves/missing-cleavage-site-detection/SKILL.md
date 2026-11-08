---
name: missing-cleavage-site-detection
description: Use when annotating validated peptide-spectrum matches (e.g. MGF spectra
  from PRIDE/MassIVE with pseudo-ground-truth PSMs filtered at 1% FDR) to quantify
  missing fragment ions before evaluating de novo sequencing recall.
when_to_use_negative:
- Input is raw vendor raw files not yet converted to MGF — convert with ProteoWizard
  first.
- PSMs have not been validated (no database-search result filtered at 1% FDR) — no
  ground-truth peptide sequence is available to compute theoretical fragment ions.
- 'Goal is to evaluate noise peaks rather than missing fragmentation: use the noise-peak
  labeling skill instead.'
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: Pyteomics
  role: Framework for computing theoretical fragment ion m/z values and matching them
    to observed MS/MS peaks within 0.5 Da tolerance
- name: ProteoWizard
  role: Converts raw vendor MS files to the MGF format consumed by the annotation
    workflow
- name: MS-GF+
  role: Database search tool providing the pseudo-ground-truth PSMs used to define
    expected fragment ions
- name: X!Tandem
  role: Database search tool providing the pseudo-ground-truth PSMs used to define
    expected fragment ions
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:f49d8be0c160fc712b549419476aa7aaa083b0b15874990945f360d3cb370bb8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - DeepNovo
  - MS-GF+
  - Novor
  - PointNovo
  - ProteoWizard
  - Pyteomics
  - SMSNet
  - X!Tandem
  - pNovo 3
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# missing-cleavage-site-detection

## Summary

Detects which peptide bond cleavage sites in a peptide-spectrum match lack any supporting fragment ion, using theoretical b/y ion-type m/z matching against the observed MS/MS peaks via Pyteomics.

## When to use

Use when annotating validated peptide-spectrum matches (e.g. MGF spectra from PRIDE/MassIVE with pseudo-ground-truth PSMs filtered at 1% FDR) to quantify missing fragment ions before evaluating de novo sequencing recall.

## When NOT to use

- Input is raw vendor raw files not yet converted to MGF — convert with ProteoWizard first.
- PSMs have not been validated (no database-search result filtered at 1% FDR) — no ground-truth peptide sequence is available to compute theoretical fragment ions.
- Goal is to evaluate noise peaks rather than missing fragmentation: use the noise-peak labeling skill instead.

## Inputs

- MGF-format MS/MS antibody spectra (converted with ProteoWizard)
- Validated peptide-spectrum matches from MS-GF+ / X!Tandem database search filtered at 1% FDR
- Peptide sequences (pseudo-ground truth) per PSM

## Outputs

- Per-spectrum annotation of missing cleavage sites (per cleavage site: matched ion types or missing flag)
- Per-spectrum count of missing cleavage sites written to spectrum_statistics.csv
- Peaked/noise peak labels per spectrum feeding the noise-factor computation

## How to apply

Load the MGF-format antibody spectra (converted with ProteoWizard) together with the validated peptide-spectrum matches from the MS-GF+ / X!Tandem database search filtered at 1% FDR. For each cleavage site of each ground-truth peptide, compute theoretical m/z values for eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) and match them to observed peaks using the Pyteomics framework within a 0.5 Da tolerance. Declare a cleavage site as missing when none of the eight ion types finds a matching peak. Aggregate per-spectrum counts of missing cleavage sites, verify against the paper's expected aggregates (90.51% of 23,227 validated spectra missing at least one fragment ion; 84.32% of peaks in those spectra being noise), and write the annotations to spectrum_statistics.csv. These per-spectrum missing-cleavage counts then serve as the binning variable for peptide-recall curves (Figure 3A) and the length x missing-cleavage heatmap (Figure 4).

## Related tools

- **Pyteomics** (Framework for computing theoretical fragment ion m/z values and matching them to observed MS/MS peaks within 0.5 Da tolerance)
- **ProteoWizard** (Converts raw vendor MS files to the MGF format consumed by the annotation workflow)
- **MS-GF+** (Database search tool providing the pseudo-ground-truth PSMs used to define expected fragment ions)
- **X!Tandem** (Database search tool providing the pseudo-ground-truth PSMs used to define expected fragment ions)

## Examples

```
from pyteomics import mgf, mass; spectra = mgf.read('antibody_spectra.mgf'); missing = [(s['params']['title'], [i for i in cleavage_sites(psm) if not any(match_ion_type(s, i, t, tol=0.5) for t in ION_TYPES)]) for s, psm in zip(spectra, psms_at_1pct_fdr)]
```

## Evaluation signals

- The aggregate statistic should reproduce the paper-reported value: 90.51% of the 23,227 validated spectra are missing at least one fragment ion.
- In the same annotated spectra, 84.32% of all peaks should classify as noise peaks.
- Every cleavage site should yield one of the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) or be explicitly flagged missing — no silent gaps.
- Match tolerance must be exactly 0.5 Da; tightening or loosening it should change the missing-site counts and is a sensitivity check.
- Output schema: one row per spectrum in spectrum_statistics.csv with a per-spectrum missing-cleavage-site count column.

## Limitations

- The 0.5 Da tolerance is coarse; near-isobaric ion types (e.g. b-NH3 vs y-NH3) may be conflated, and the skill cannot discriminate isoleucine/leucine since MS/MS alone cannot distinguish them.
- Only validated PSMs are analyzed — spectra that failed the 1% FDR database-search filter are not annotated.
- This skill detects missing fragmentation only; noisy-spectra impact is assessed by a separate noise-factor computation.

## Evidence

- [methods] ion type matching: [evidence span withheld — anchor status: not_found]
- [results] reported aggregate statistics: [evidence span withheld — anchor status: not_found]
- [abstract] tool comparison framing: "we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo)"
- [intro] proteomics data provenance: "The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36]."
