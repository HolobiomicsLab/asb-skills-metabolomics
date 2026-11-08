---
name: massbank-spectrum-parsing
description: Reference for generating MetaboAnnotatoR fragment-library entries from
  public MS/MS databases when you have a MassBank record or .msp spectrum and need
  peaks scored as marker fragments for AIF LC-MS annotation.
when_to_use_negative:
- When annotating an already-built library against AIF LC-MS data — use MetaboAnnotatoR's
  annotation workflow instead of generating new library entries.
- When only peak tables from XCMS or similar are available and no MS/MS spectrum or
  .msp record exists — this skill requires an actual MS/MS spectrum as input.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: R package whose genFragEntry and mspToLib functions convert MassBank / .msp
    MS/MS spectra into scored fragment-library entries (.csv)
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MassBank
  role: Public MS/MS spectral database whose records (e.g. MSBNK-RIKEN-PR100295) are
    the source spectra for library entry generation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:5e2d91ddabeaa4ae25eee835d576e16c4d93b2447633928859b3cedece649d68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MassBank
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# massbank-spectrum-parsing

## Summary

Parse MassBank-format MS/MS records (e.g. Pantothenic acid [M+H]+, MSBNK-RIKEN-PR100295) and generate MetaboAnnotatoR fragment-library entries via genFragEntry, or import .msp records directly with mspToLib, writing scored peak lists as .csv library files.

## When to use

Reference for generating MetaboAnnotatoR fragment-library entries from public MS/MS databases when you have a MassBank record or .msp spectrum and need peaks scored as marker fragments for AIF LC-MS annotation.

## When NOT to use

- When annotating an already-built library against AIF LC-MS data — use MetaboAnnotatoR's annotation workflow instead of generating new library entries.
- When only peak tables from XCMS or similar are available and no MS/MS spectrum or .msp record exists — this skill requires an actual MS/MS spectrum as input.

## Inputs

- Experimental MS/MS spectrum of a metabolite standard (e.g. MassBank record MSBNK-RIKEN-PR100295, Pantothenic acid [M+H]+)
- Public-database MS/MS record in .msp format
- User-specified parameters: metabolite name, adduct, accurate adduct m/z, output filename, mpeaksThres threshold, noise level

## Outputs

- MetaboAnnotatoR .csv fragment-library entry with scored marker peaks and occurrence scores
- Library entry produced from .msp records via mspToLib in the same format

## How to apply

Load the experimental MS/MS spectrum (or .msp record) of a metabolite standard, e.g. the Pantothenic acid [M]+ (MassBank MSBNK-RIKEN-PR100295) example used in the MetaboAnnotatoR vignette. Run MetaboAnnotatoR's genFragEntry on the spectrum: it attributes occurrence scores to 'marker peaks' above the mpeaksThres threshold and above the noise level, with metabolite name, adduct, accurate adduct m/z and output filename explicitly defined by the user. Write the scored peak list as a MetaboAnnotatoR .csv library entry for use in fragment matching against AIF datasets. As an alternative import path, use mspToLib to parse the .msp record directly into the same library-entry format; external libraries can be supplied as .txt and .msp files to customize the default libraries. Validate the generated entry against the vignette's example Pantothenic acid [M]+ entry, checking that fragment m/z values and occurrence scores match; discrepancies in scored peaks indicate incorrect occurrence attribution.

## Related tools

- **MetaboAnnotatoR** (R package whose genFragEntry and mspToLib functions convert MassBank / .msp MS/MS spectra into scored fragment-library entries (.csv)) — https://github.com/gggraca/MetaboAnnotatoR
- **MassBank** (Public MS/MS spectral database whose records (e.g. MSBNK-RIKEN-PR100295) are the source spectra for library entry generation)

## Examples

```
genFragEntry(name='Pantothenic acid', adduct='[M+H]+', adduct_mz=200.1001, spectrum=mb_record, mpeaksThres=0.05, noise=0.01, outfile='PA_fraglib.csv'); mspToLib(file='PA.msp')
```

## Evaluation signals

- The generated .csv library entry matches the vignette's example Pantothenic acid [M+H]+ entry (MSBNK-RIKEN-PR100295) in fragment m/z values and occurrence scores.
- All retained peaks are above the mpeaksThres threshold and above the noise level; peaks below either cutoff are absent from the entry.
- The entry records the metabolite name, adduct, and accurate adduct m/z exactly as specified by the user.
- An entry produced via mspToLib from the same .msp record is consistent with the genFragEntry output (same peak set and occurrence scoring).

## Limitations

- Occurrence scoring depends on the mpeaksThres threshold and noise level — incorrect settings mis-score marker peaks.
- External fragment libraries are limited by the spectral coverage of the source database; small libraries (e.g. nonlipid) yield low recall in downstream annotation.
- The MetaboAnnotatoR workflow currently supports AIF acquisition only, so libraries built this way are validated for AIF (not SWATH or other DIA schemes).

## Evidence

- [methods] genFragEntry attributes occurrence scores to marker peaks: [evidence span withheld — anchor status: not_found]
- [methods] mspToLib parses .msp records: [evidence span withheld — anchor status: not_found]
- [readme] README lists the .msp vignette: [evidence span withheld — anchor status: external_doc]
- [discussion] Customization via .txt/.msp import: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
- [discussion] Low recall for small libraries: "These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library."
