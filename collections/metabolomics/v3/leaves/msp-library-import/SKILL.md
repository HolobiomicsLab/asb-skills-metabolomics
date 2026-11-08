---
name: msp-library-import
description: Use when the default MetaboAnnotatoR fragment libraries lack the target
  metabolite class and you have external MS/MS spectra in .msp (or .txt) format to
  import as a custom fragment library.
when_to_use_negative:
- You already have a MetaboAnnotatoR-compatible .csv fragment-library entry — re-importing
  is redundant.
- The spectra are raw vendor-format MS/MS files (e.g. vendor centroid chromatograms)
  rather than .msp or .txt spectral lists — convert them first or use the genFragEntry
  path on the experimental spectrum directly.
- The target annotation workflow is MS-DIAL, LipidMatch, or R-MetaboList, which use
  different library formats.
edam_operation: http://edamontology.org/operation_0405
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnnotatoR
  role: R package whose library-import functions (mspToLib and .txt import) convert
    external MS/MS spectra into the package's .csv fragment-library format for use
    in AIF annotation
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: runtime environment in which the MetaboAnnotatoR library-import and vignette
    steps are executed
- name: MassBank
  role: public spectral database supplying .msp records (e.g. MSBNK-RIKEN-PR100295)
    used as import material
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:5cfa090f68586cbf64caef7ac0905b14b763f2ea8e467e9881c71036274bad9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MassBank
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_0405
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# msp-library-import

## Summary

Customizes MetaboAnnotatoR's default fragment libraries by importing external metabolite MS/MS spectra as .txt and .msp files, enabling annotation workflows beyond the shipped default libraries.

## When to use

Use when the default MetaboAnnotatoR fragment libraries lack the target metabolite class and you have external MS/MS spectra in .msp (or .txt) format to import as a custom fragment library.

## When NOT to use

- You already have a MetaboAnnotatoR-compatible .csv fragment-library entry — re-importing is redundant.
- The spectra are raw vendor-format MS/MS files (e.g. vendor centroid chromatograms) rather than .msp or .txt spectral lists — convert them first or use the genFragEntry path on the experimental spectrum directly.
- The target annotation workflow is MS-DIAL, LipidMatch, or R-MetaboList, which use different library formats.

## Inputs

- External MS/MS spectral records in .msp format (e.g. from METLIN, MassBank, or GNPS spectral databases)
- External metabolite fragment/MS2 spectral lists in .txt format
- Metabolite name, adduct label, and accurate adduct m/z for each library entry

## Outputs

- MetaboAnnotatoR .csv fragment-library entry with metabolite name, adduct, adduct m/z, fragment m/z values, and occurrence scores
- Customized fragment library ready to be supplied to the MetaboAnnotatoR annotation workflow

## How to apply

Convert or import the external MS/MS spectral records (e.g. from public repositories such as METLIN, MassBank, or GNPS) into the MetaboAnnotatoR library format by using the package's .msp and .txt import functions; the README explicitly states libraries 'can be customized by the user, for instance, by importing external libraries as .txt and .msp files'. The parsing path is the same as the vignette for 'Generation of Metabolite fragment database entry from MS/MS spectra from public databases (in .msp format)', where mspToLib reads the .msp record, attributes occurrence scores to fragment peaks, and writes a scored peak list as the .csv fragment-library entry. Keep the metabolite name, adduct label, and accurate adduct m/z defined alongside the entry so that later library matching can assign the correct adduct assignment. Validate the imported entry by comparing fragment m/z values and occurrence scores against the source .msp record (as done for the Pantothenic acid [M+H]+ example, MassBank MSBNK-RIKEN-PR100295). The resulting .csv library can then be supplied as a custom fragment library to the MetaboAnnotatoR annotation workflow alongside the default lipidomics and nonlipid libraries.

## Related tools

- **MetaboAnnotatoR** (R package whose library-import functions (mspToLib and .txt import) convert external MS/MS spectra into the package's .csv fragment-library format for use in AIF annotation) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (runtime environment in which the MetaboAnnotatoR library-import and vignette steps are executed)
- **MassBank** (public spectral database supplying .msp records (e.g. MSBNK-RIKEN-PR100295) used as import material)

## Examples

```
library(MetaboAnnotatoR); libEntry <- mspToLib("MSBNK-RIKEN-PR100295.msp", metabolite = "Pantothenic acid", adduct = "[M+H]+", mz = 220.1182)
```

## Evaluation signals

- The imported .csv entry must contain the same fragment m/z values and occurrence scores as the source .msp record (verified in the vignette against the Pantothenic acid [M]+ example, MassBank MSBNK-RIKEN-PR100295).
- Each imported entry must carry a metabolite name, an adduct label, and an accurate adduct m/z — fields required by downstream library matching.
- Imported entries must fall within the .csv schema the MetaboAnnotatoR library-matching functions expect (same columns as the shipped default libraries).
- After import, running the standard annotation workflow with the custom library should produce adduct assignments for the imported metabolite classes instead of missing-library warnings.

## Limitations

- Only the AIF (all-ion fragmentation) DIA scheme is currently supported; other DIA schemes such as SWATH are not (article discussion).
- Imported .msp entries are scored using the package's occurrence-attribution rules (mpeaksThres and noise-level filtering), so entries derived from very different acquisition energies may carry scores that bias downstream library matching.
- The MetaboAnnotatoR default nonlipid library is small; importing external .msp spectra does not fix low recall for nonlipid-dominated matrices (article discussion).
- Installation can fail with 'mzR has been built against a different Rcpp version' errors; the README recommends setting Sys.setenv(R_REMOTES_NO_ERRORS_FROM_WARNINGS="true") or installing with INSTALL_opts = c('--no-multiarch', '--no-lock').

## Evidence

- [discussion] Import external fragment libraries as .txt and .msp files to customize default libraries: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
- [readme] Vignettes folder lists .msp database import as an example use case: [evidence span withheld — anchor status: external_doc]
- [full_text] Library entry generation attributes occurrence scores to marker peaks above mpeaksThres and the noise level: [evidence span withheld — anchor status: not_found]
- [full_text] mspToLib converts .msp libraries to the MetaboAnnotatoR format: [evidence span withheld — anchor status: not_found]
- [intro] Researchers have built metabolite spectral libraries containing thousands of MS/MS spectra: "researchers have built metabolite spectral libraries containing thousands of MS/MS spectra and developed spectral matching algorithms to annotate unknowns using spectral similarity."
