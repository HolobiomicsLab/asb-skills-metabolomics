---
name: fragment-ion-type-matching
description: 'Use when evaluating de novo peptide sequencing inputs on antibody MGF
  spectra: you need per-cleavage-site ion coverage, missing-fragment statistics, or
  peptide/noise peak labels for spectral quality assessment.'
when_to_use_negative:
- Input spectra lack validated PSMs / ground truth peptide assignments, so theoretical
  ion types per cleavage site cannot be computed
- The goal is peptide identification itself (this skill only annotates spectral quality;
  run a de novo tool or database search instead)
- Data are in a non-MGF vendor raw format without prior ProteoWizard conversion
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Pyteomics
  role: Framework used to read MGF spectra and match theoretical fragment ion m/z
    values to observed peaks within the 0.5 Da tolerance
- name: MS-GF+
  role: Database search engine supplying the pseudo-ground truth PSMs at 1% FDR
- name: X!Tandem
  role: Database search engine supplying the pseudo-ground truth PSMs at 1% FDR
- name: ProteoWizard
  role: Used to convert raw antibody MS/MS data into MGF format before annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:5c99d5e232a31e70f25424d6acf9812403ec2bd090b2b42b213d4b1bd5b8678d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-GF+
  - ProteoWizard
  - Pyteomics
  - X!Tandem
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# fragment-ion-type-matching

## Summary

Annotates peptide fragmentation completeness in MS/MS spectra by matching eight theoretical ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) to observed peaks with Pyteomics, and labels peaks as peptide vs. noise to derive per-dataset noise factors.

## When to use

Use when evaluating de novo peptide sequencing inputs on antibody MGF spectra: you need per-cleavage-site ion coverage, missing-fragment statistics, or peptide/noise peak labels for spectral quality assessment.

## When NOT to use

- Input spectra lack validated PSMs / ground truth peptide assignments, so theoretical ion types per cleavage site cannot be computed
- The goal is peptide identification itself (this skill only annotates spectral quality; run a de novo tool or database search instead)
- Data are in a non-MGF vendor raw format without prior ProteoWizard conversion

## Inputs

- MGF-format MS/MS spectra of antibody digests (converted with ProteoWizard)
- Pseudo-ground truth peptide-spectrum matches from MS-GF+ / X!Tandem database search filtered at 1% FDR
- Ground truth peptide sequences for cleavage-site-level theoretical ion computation

## Outputs

- Per-spectrum annotation table (spectrum_quality_annotations.csv) with number of missing cleavage sites and peptide/noise peak labels
- Per-dataset noise factors (ratio of high-intensity peaks to fragment ion peaks)

## How to apply

Load MGF-format antibody spectra (converted with ProteoWizard) together with pseudo-ground truth PSMs from an MS-GF+ / X!Tandem database search filtered at 1% FDR. For each cleavage site of each ground truth peptide, compute theoretical m/z values for the eight ion types and match them to observed peaks within a 0.5 Da tolerance using the Pyteomics framework; declare a cleavage site missing when no ion type matches. Label every peak in each spectrum as a peptide peak or a noise peak, retaining only noise peaks whose intensity exceeds the median noise intensity of the corresponding dataset. Compute the per-dataset noise factor as the ratio of high-intensity noise peaks to fragment ion peaks, and write per-spectrum annotations (missing cleavage sites, peak labels) and noise factors to spectrum_quality_annotations.csv.

## Related tools

- **Pyteomics** (Framework used to read MGF spectra and match theoretical fragment ion m/z values to observed peaks within the 0.5 Da tolerance)
- **MS-GF+** (Database search engine supplying the pseudo-ground truth PSMs at 1% FDR)
- **X!Tandem** (Database search engine supplying the pseudo-ground truth PSMs at 1% FDR)
- **ProteoWizard** (Used to convert raw antibody MS/MS data into MGF format before annotation)

## Examples

```
from pyteomics import mgf; spectrum = next(mgf.read('antibody_spectra.mgf')); peaks = spectrum['m/z array']
```

## Evaluation signals

- The annotation reproduces the paper-reported statistics: 90.51% of the 23,227 validated spectra are missing at least one fragment ion
- 84.32% of all peaks in those spectra are classified as noise peaks
- Every peak in each spectrum receives exactly one label (peptide peak or noise peak) and each noise peak retained exceeds the median noise intensity of its dataset
- A cleavage site is declared missing only when none of the eight ion types matches an observed peak within 0.5 Da
- Noise factor per dataset equals the count of high-intensity noise peaks divided by the count of fragment ion peaks

## Limitations

- A 0.5 Da match tolerance means low-resolution or high-precision spectra may yield over- or under-annotation of missing cleavage sites
- The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools
- Only b- and y-type ions (including 2+ and neutral-loss variants) are considered; other ion series are not annotated
- Isoleucine/leucine and other isobaric residues cannot be resolved via MS/MS, so cleavage-site completeness is blind to such ambiguities

## Evidence

- [methods] the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) were matched to spectral peaks using Pyteomics: [evidence span withheld — anchor status: not_found]
- [methods] 90.51% of the 23,227 validated spectra missing at least one fragment ion and 84.32% of all peaks in those spectra classified as noise peaks: [evidence span withheld — anchor status: not_found]
- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
- [intro] we investigate common error types, the impact of noisy spectra and missing fragmentation ions: "Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions."
