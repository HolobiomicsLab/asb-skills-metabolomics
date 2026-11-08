---
name: noise-factor-computation-across-datasets
description: Use when evaluating how noisy spectra and missing fragmentation ions
  affect de novo peptide sequencing, by annotating MGF spectra against validated PSMs
  and computing per-dataset noise factors.
when_to_use_negative:
- Spectra have no validated PSMs (pseudo ground truth) from a 1% FDR database search,
  so theoretical fragment ions cannot be matched to observed peaks to call missing
  cleavage sites.
- The goal is de novo sequencing prediction itself (e.g. running Novor, pNovo 3, DeepNovo,
  SMSNet, PointNovo or Casanovo) rather than characterizing the spectral input quality
  fed to those tools.
- The input is already quality-annotated (peak labels and missing-cleavage-site counts
  exist) and only the downstream noise-factor ratios need recombination.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Pyteomics
  role: Computes theoretical m/z for the eight fragment ion types per cleavage site
    and matches them to observed MGF peaks within 0.5 Da; supports peptide/noise peak
    labeling
- name: MS-GF+ / X!Tandem
  role: Provides the pseudo-ground truth peptide-spectrum matches (filtered at 1%
    FDR) used as reference peptide sequences for cleavage-site annotation
- name: ProteoWizard
  role: Converts vendor antibody MS/MS data into the MGF format ingested by the annotation
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:c67159ea79682e921c7755c93a99ab753f69227f41514dadeb20f83f6f9b963c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-GF+ / X!Tandem
  - ProteoWizard
  - Pyteomics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# noise-factor-computation-across-datasets

## Summary

Annotates antibody MS/MS spectra by labeling every peak as peptide or noise peak and by flagging missing fragment-ion cleavage sites, then computes a per-dataset noise factor (ratio of high-intensity noise peaks to fragment ion peaks) to quantify spectral quality for de novo sequencing evaluation.

## When to use

Use when evaluating how noisy spectra and missing fragmentation ions affect de novo peptide sequencing, by annotating MGF spectra against validated PSMs and computing per-dataset noise factors.

## When NOT to use

- Spectra have no validated PSMs (pseudo ground truth) from a 1% FDR database search, so theoretical fragment ions cannot be matched to observed peaks to call missing cleavage sites.
- The goal is de novo sequencing prediction itself (e.g. running Novor, pNovo 3, DeepNovo, SMSNet, PointNovo or Casanovo) rather than characterizing the spectral input quality fed to those tools.
- The input is already quality-annotated (peak labels and missing-cleavage-site counts exist) and only the downstream noise-factor ratios need recombination.

## Inputs

- MGF-format antibody MS/MS spectra (converted with ProteoWizard)
- Pseudo-ground truth PSMs from MS-GF+ / X!Tandem database search filtered at 1% FDR
- Data set grouping of spectra (for median noise intensity and per-dataset noise factor computation)

## Outputs

- spectrum_quality_annotations.csv with per-spectrum annotations: number of missing cleavage sites and peptide/noise peak labels for every peak
- Per-data-set noise factor = number of high-intensity noise peaks (above median noise intensity) divided by number of fragment ion peaks

## How to apply

Load the MGF-format antibody spectra (converted with ProteoWizard) together with the pseudo-ground truth peptide-spectrum matches from an MS-GF+ / X!Tandem database search filtered at 1% FDR. For each cleavage site of each ground truth peptide, compute theoretical m/z values for the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) and match them to observed peaks within a 0.5 Da tolerance using the Pyteomics framework; declare a cleavage site as missing when no ion type matches. Label every peak in each spectrum as a peptide peak or a noise peak, retaining only noise peaks whose intensity exceeds the median noise intensity of the corresponding data set. Compute the noise factor per data set as the ratio of the number of high-intensity peaks to the number of fragment ion peaks, and write the per-spectrum annotations and per-data-set noise factors to spectrum_quality_annotations.csv.

## Related tools

- **Pyteomics** (Computes theoretical m/z for the eight fragment ion types per cleavage site and matches them to observed MGF peaks within 0.5 Da; supports peptide/noise peak labeling)
- **MS-GF+ / X!Tandem** (Provides the pseudo-ground truth peptide-spectrum matches (filtered at 1% FDR) used as reference peptide sequences for cleavage-site annotation)
- **ProteoWizard** (Converts vendor antibody MS/MS data into the MGF format ingested by the annotation workflow)

## Examples

```
from pyteomics import mgf; spectra = mgf.read('antibody_spectra.mgf')
```

## Evaluation signals

- Reproduce the paper-reported statistics: 90.51% of the 23,227 validated spectra missing at least one fragment ion.
- Reproduce the paper-reported peak-labeling statistic: 84.32% of all peaks in those spectra classified as noise peaks.
- A cleavage site is flagged missing exactly when none of the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) matched within 0.5 Da.
- Noise peaks retained must all exceed the median noise intensity of the corresponding data set; noise factor equals high-intensity peak count divided by fragment ion peak count.
- Output file spectrum_quality_annotations.csv must contain per-spectrum missing-cleavage-site counts and peptide/noise labels plus per-data-set noise factors.

## Limitations

- The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools, so noise factors explain but do not fully determine tool accuracy.
- The 0.5 Da fragment-ion matching tolerance is coarse; ion types not among the eight considered (e.g. other neutral losses) are not matched.
- Noise thresholding depends on the median noise intensity of each data set, making noise factors comparable across data sets but sensitive to data-set composition.

## Evidence

- [methods] For each cleavage site, the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) were matched to spectral peaks using Pyteomics; if no match within 0.5 Da was found: [evidence span withheld — anchor status: not_found]
- [results] 90.51% of the 23,227 validated spectra missing at least one fragment ion and 84.32% of all peaks in those spectra classified as noise peaks: [evidence span withheld — anchor status: not_found]
- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
