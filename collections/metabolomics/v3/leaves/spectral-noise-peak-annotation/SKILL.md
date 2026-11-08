---
name: spectral-noise-peak-annotation
description: 'Use when assessing MS/MS spectrum quality for de novo sequencing studies:
  you need per-spectrum counts of missing fragment ions, peptide-vs-noise peak labels,
  or a noise factor per dataset.'
when_to_use_negative:
- Spectra lack validated PSMs (e.g. no 1%-FDR database-search results), so no ground-truth
  peptide exists to derive theoretical fragment ions from.
- The goal is de novo sequencing or assembly itself (e.g. running Casanovo/PointNovo
  or ALPS) rather than spectrum-quality characterization.
- Input is already a spectrum-quality table — re-annotation would duplicate the missing-cleavage-site
  and noise-peak labels.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Pyteomics
  role: Computes theoretical m/z values for the eight ion types per cleavage site
    and matches them to observed spectral peaks within 0.5 Da to label peptide vs
    noise peaks
- name: ProteoWizard
  role: Converts raw antibody MS data to MGF-format spectra prior to annotation
- name: MS-GF+
  role: Provides the pseudo-ground-truth PSMs (1% FDR) whose cleavage sites drive
    the fragment-ion completeness annotation
- name: X!Tandem
  role: Second database-search engine supplying pseudo-ground-truth PSMs at 1% FDR
    for the annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:082b13cc0289fcb97847c6252dc7fcdb3236b052a698f7db2b4267f972d0b416
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-GF+
  - ProteoWizard
  - Pyteomics
  - X!Tandem
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# spectral-noise-peak-annotation

## Summary

Annotates antibody MS/MS spectra to label every peak as a peptide fragment-ion peak or a noise peak and to quantify missing cleavage sites and per-dataset noise factors. This is the mechanism the paper used to evaluate the impact of missing fragmentation sites and noisy spectra on de novo peptide sequencing tools.

## When to use

Use when assessing MS/MS spectrum quality for de novo sequencing studies: you need per-spectrum counts of missing fragment ions, peptide-vs-noise peak labels, or a noise factor per dataset.

## When NOT to use

- Spectra lack validated PSMs (e.g. no 1%-FDR database-search results), so no ground-truth peptide exists to derive theoretical fragment ions from.
- The goal is de novo sequencing or assembly itself (e.g. running Casanovo/PointNovo or ALPS) rather than spectrum-quality characterization.
- Input is already a spectrum-quality table — re-annotation would duplicate the missing-cleavage-site and noise-peak labels.

## Inputs

- MGF-format antibody MS/MS spectra (converted with ProteoWizard)
- Pseudo-ground-truth peptide-spectrum matches from MS-GF+ / X!Tandem database search filtered at 1% FDR

## Outputs

- spectrum_quality_annotations.csv with per-spectrum annotations (number of missing cleavage sites, peptide/noise peak labels)
- Per-dataset noise factors (ratio of high-intensity noise peaks to fragment ion peaks)

## How to apply

Load MGF-format antibody spectra (converted with ProteoWizard) together with pseudo-ground-truth PSMs from an MS-GF+ / X!Tandem database search filtered at 1% FDR. For each cleavage site of each ground-truth peptide, compute theoretical m/z values for eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) using Pyteomics and match them to observed peaks within a 0.5 Da tolerance; declare the cleavage site missing when no ion type matches. Label every remaining peak in each spectrum as a peptide peak or a noise peak, retaining only noise peaks whose intensity exceeds the median noise intensity of the corresponding dataset. Compute the noise factor per dataset as the ratio of the number of high-intensity noise peaks to the number of fragment ion peaks, and write per-spectrum annotations plus per-dataset noise factors to spectrum_quality_annotations.csv.

## Related tools

- **Pyteomics** (Computes theoretical m/z values for the eight ion types per cleavage site and matches them to observed spectral peaks within 0.5 Da to label peptide vs noise peaks)
- **ProteoWizard** (Converts raw antibody MS data to MGF-format spectra prior to annotation)
- **MS-GF+** (Provides the pseudo-ground-truth PSMs (1% FDR) whose cleavage sites drive the fragment-ion completeness annotation)
- **X!Tandem** (Second database-search engine supplying pseudo-ground-truth PSMs at 1% FDR for the annotation)

## Examples

```
from pyteomics import mgf, mass; spectra = mgf.read('antibody_spectra.mgf'); tol = 0.5  # Da match tolerance for b/y ion types
```

## Evaluation signals

- Reproduce paper-reported statistics: 90.51% of the 23,227 validated spectra have at least one missing fragment ion.
- Reproduce the noise classification: 84.32% of all peaks in those spectra are labeled as noise peaks.
- Every peak in each spectrum carries exactly one label (peptide peak or noise peak), with only high-intensity noise (above the median noise intensity of the dataset) retained.
- A cleavage site is counted as missing only when none of the eight ion types matches an observed peak within 0.5 Da.
- Output spectrum_quality_annotations.csv contains both per-spectrum fields (missing cleavage sites, peak labels) and per-dataset noise factors.

## Limitations

- Missing fragmentation sites, noisy spectra and long peptide sequences pose a limit for all de novo sequencing tools evaluated.
- Discrimination of the isomeric residues isoleucine and leucine cannot be achieved via MS/MS and requires MS3 fragmentation, constraining ion-type annotation.
- Particular amino acids cannot be resolved even with higher-resolution spectra and tools (e.g. I & L; Q & AG; deamidated N & D).

## Evidence

- [methods] For each cleavage site, the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) were matched to spectral peaks: [evidence span withheld — anchor status: not_found]
- [results] 90.51% of the 23,227 validated spectra missing at least one fragment ion and 84.32% of all peaks in those spectra classified as noise peaks: [evidence span withheld — anchor status: not_found]
- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
