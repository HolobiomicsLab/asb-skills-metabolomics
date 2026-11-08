---
name: msms-spectrum-quality-metrics
description: Use when antibody MGF spectra plus a database-search pseudo-ground truth
  (MS-GF+/X!Tandem, 1% FDR) are available and you need to quantify missing fragment
  ions or noise-peak fractions to interpret de novo sequencing errors.
when_to_use_negative:
- Input spectra lack a validated database-search ground truth (e.g. purely de novo
  predictions with no MS-GF+/X!Tandem PSMs at 1% FDR), so fragment ion completeness
  cannot be assessed against known sequences.
- Spectra are already pre-annotated with ion assignments or the analysis goal is de
  novo prediction itself rather than spectrum quality characterization.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Pyteomics
  role: Framework used to compute theoretical m/z values for the eight ion types and
    match them to observed peaks within 0.5 Da tolerance for fragment ion completeness
    and noise annotation
- name: MS-GF+
  role: Database search tool supplying the pseudo-ground truth peptide-spectrum matches
    (filtered at 1% FDR) against which fragment ion completeness is evaluated
- name: X!Tandem
  role: Database search tool supplying the pseudo-ground truth peptide-spectrum matches
    (filtered at 1% FDR) against which fragment ion completeness is evaluated
- name: ProteoWizard
  role: Used to convert raw MS files to the MGF spectra consumed by this annotation
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:7b0067163d3ead6a346b06dfb5fdd34210fd3ce3fb3cf0096363c4ed24923626
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-GF+
  - ProteoWizard
  - Pyteomics
  - X!Tandem
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# msms-spectrum-quality-metrics

## Summary

Annotates antibody MS/MS spectra for fragment ion completeness and noise by matching the eight b/y ion types to observed peaks with Pyteomics at 0.5 Da tolerance, then computes per-dataset noise factors. This quantifies the spectral quality limitations (missing fragmentation sites and noisy peaks) that constrain de novo peptide sequencing tools.

## When to use

Use when antibody MGF spectra plus a database-search pseudo-ground truth (MS-GF+/X!Tandem, 1% FDR) are available and you need to quantify missing fragment ions or noise-peak fractions to interpret de novo sequencing errors.

## When NOT to use

- Input spectra lack a validated database-search ground truth (e.g. purely de novo predictions with no MS-GF+/X!Tandem PSMs at 1% FDR), so fragment ion completeness cannot be assessed against known sequences.
- Spectra are already pre-annotated with ion assignments or the analysis goal is de novo prediction itself rather than spectrum quality characterization.

## Inputs

- MGF-format antibody MS/MS spectra (converted with ProteoWizard)
- Pseudo-ground truth peptide-spectrum matches from MS-GF+ / X!Tandem database search filtered at 1% FDR

## Outputs

- spectrum_quality_annotations.csv with per-spectrum number of missing cleavage sites and peptide/noise peak labels
- Per-dataset noise factors (ratio of high-intensity peaks to fragment ion peaks)

## How to apply

Load the MGF-format antibody spectra (converted with ProteoWizard) together with the peptide-spectrum matches from the MS-GF+/X!Tandem database search filtered at 1% FDR. For each cleavage site of each ground truth peptide, compute theoretical m/z values for the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) and match them to observed peaks within a 0.5 Da tolerance using Pyteomics; declare a cleavage site as missing when no ion type matches. Label every peak in each spectrum as a peptide peak or a noise peak, retaining only noise peaks whose intensity exceeds the median noise intensity of the corresponding dataset. Compute the noise factor per dataset as the ratio of the number of high-intensity peaks to the number of fragment ion peaks. Write the per-spectrum annotations (number of missing cleavage sites, peptide/noise peak labels) and per-dataset noise factors to spectrum_quality_annotations.csv.

## Related tools

- **Pyteomics** (Framework used to compute theoretical m/z values for the eight ion types and match them to observed peaks within 0.5 Da tolerance for fragment ion completeness and noise annotation)
- **MS-GF+** (Database search tool supplying the pseudo-ground truth peptide-spectrum matches (filtered at 1% FDR) against which fragment ion completeness is evaluated)
- **X!Tandem** (Database search tool supplying the pseudo-ground truth peptide-spectrum matches (filtered at 1% FDR) against which fragment ion completeness is evaluated)
- **ProteoWizard** (Used to convert raw MS files to the MGF spectra consumed by this annotation workflow)

## Examples

```
import pyteomics.auxiliary as aux; matching = aux.naive_ions(spectrum_peaks, ion_types=['b','b2','b-NH3','b-H2O','y','y2','y-NH3','y-H2O'], tolerance=0.5)
```

## Evaluation signals

- Reproduce the paper-reported statistics: 90.51% of the 23,227 validated spectra are missing at least one fragment ion.
- Reproduce the reported 84.32% of all peaks in spectra missing at least one fragment ion being classified as noise peaks.
- Per-dataset noise factor should equal (number of high-intensity peaks) / (number of fragment ion peaks) using the median-intensity noise filter.
- Every cleavage site with no match among the eight ion types within 0.5 Da must be counted as missing; verify a site matched by any of b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O is not counted as missing.
- Output CSV contains per-spectrum missing-cleavage-site counts and peptide/noise peak labels without crashing on spectra with no matching ions.

## Limitations

- Missing fragmentation sites, noisy spectra and long peptide sequences pose a limit for all de novo sequencing tools — the annotations quantify, but do not overcome, these limits.
- The 0.5 Da tolerance matches the paper's evaluation setup; results under different instrument mass accuracy are not validated here.
- Only spectra with a validated database-search ground truth at 1% FDR can be annotated; unannotated spectra cannot be assessed.

## Evidence

- [methods] Fragment ion completeness: [evidence span withheld — anchor status: not_found]
- [methods] Noise factor definition: [evidence span withheld — anchor status: not_found]
- [methods] Noise peak filtering criterion: [evidence span withheld — anchor status: not_found]
- [results] Validation statistics: [evidence span withheld — anchor status: not_found]
- [discussion] Quality limitations on de novo tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
