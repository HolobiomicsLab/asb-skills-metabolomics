---
name: eic-baseline-correction
description: Use when raw LC-MS EICs for targeted metabolites carry a sloping or curved
  baseline that must be removed before peak detection with MassSpecWavelet in TarMet.
when_to_use_negative:
- Input is already a peak table or feature matrix (TarMet's quantitative analysis
  section) rather than raw EICs
- The analysis goal is untargeted feature detection on raw spectra rather than targeted
  EIC extraction with defined metabolite masses
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: TarMet
  role: Shiny/GUI application that orchestrates the EIC baseline correction step (airPLS)
    as part of its isotopologues analysis workflow
  repo: https://github.com/hcji/TarMet
- name: airPLS
  role: Algorithm used to remove the baselines of all the EICs before peak detection
- name: mzR
  role: Bioconductor dependency used by TarMet to read raw mass-spectrometry files
    from which EICs are derived
- name: shiny
  role: Framework providing the reactive GUI through which the baseline-correction
    workflow is run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:b0abfcfee80cc56ca09bcc1114fcf68d17b68b36294cbb640faa661687550cf9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  - enviPat
  - mzR
  - shiny
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# eic-baseline-correction

## Summary

Baseline correction of extracted ion chromatograms (EICs) in targeted LC-MS metabolomics using the airPLS method, as implemented in TarMet prior to peak detection.

## When to use

Use when raw LC-MS EICs for targeted metabolites carry a sloping or curved baseline that must be removed before peak detection with MassSpecWavelet in TarMet.

## When NOT to use

- Input is already a peak table or feature matrix (TarMet's quantitative analysis section) rather than raw EICs
- The analysis goal is untargeted feature detection on raw spectra rather than targeted EIC extraction with defined metabolite masses

## Inputs

- Raw LC-MS data (e.g. mzML/mzData readable via mzR) for a targeted metabolite set
- Target metabolite list with masses/formulas used to generate EICs

## Outputs

- Baseline-corrected EIC chromatograms for each target metabolite (and isotopologue)
- Peak positions suitable for downstream MassSpecWavelet detection and PAFFT alignment

## How to apply

Within the TarMet isotopologues analysis workflow, compute the EIC for each target metabolite's M+H mass (or isotopologue mass) and then apply the airPLS method to remove the baseline from every EIC before peak positions are detected with the MassSpecWavelet package. AirPLS fits a penalized least-squares baseline to the chromatographic signal and subtracts it, which improves peak-position detection and subsequent retention-time alignment with PAFFT across samples. The corrected EICs then feed the isotopologue extraction pipeline where candidate peaks are filtered with a relative abundance threshold of 0.001. Because the baseline correction operates on the EIC level (not raw spectra), it should be run after the target metabolite list (including formulas for enviPat isotope pattern calculation) has been defined. Correct application is judged by cleaner peak picking and, ultimately, by isotopic similarity scores between observed and theoretical isotope patterns computed via enviPat at the instrument resolution.

## Related tools

- **TarMet** (Shiny/GUI application that orchestrates the EIC baseline correction step (airPLS) as part of its isotopologues analysis workflow) — https://github.com/hcji/TarMet
- **airPLS** (Algorithm used to remove the baselines of all the EICs before peak detection)
- **mzR** (Bioconductor dependency used by TarMet to read raw mass-spectrometry files from which EICs are derived)
- **shiny** (Framework providing the reactive GUI through which the baseline-correction workflow is run)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- All EICs for the defined target metabolites have had a baseline removed before MassSpecWavelet peak-position detection is attempted
- Corrected EICs show peak positions that align across samples under the PAFFT retention-time alignment step
- Downstream isotopic similarity scores for detected metabolites (TarMet: 56 of 76 detected metabolites with similarity > 0.90 on the human blood metabolites dataset) remain within the reported range
- No candidate peak is lost or spuriously detected at the relative abundance threshold of 0.001 applied after correction

## Limitations

- Baseline correction is performed only on EICs derived from a predefined target list; metabolites absent from that list will not be processed
- Future features listed in the README (data-independent acquisition, MS/MS library verification) are not yet supported, so the correction is restricted to the current targeted workflow

## Evidence
