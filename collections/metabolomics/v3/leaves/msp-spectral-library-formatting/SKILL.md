---
name: msp-spectral-library-formatting
description: Use when enumerating in silico candidate metabolite structures (e.g.
  acylcarnitines covering saturated, unsaturated, hydroxyl, keto, dicarboxylic, and
  oxidized acyl chains) and you need them packaged as a NIST MSP spectral library
  for MSI level 2 precursor-accurate-mass and MS/MS matching.
when_to_use_negative:
- When the goal is MSI level 1 identification — that requires authentic standards
  with precursor mass, retention time, and MS/MS matching, not an in silico library
- When spectra are already in a searchable format or the target compound class is
  not represented in the enumeration template (e.g. non-acyl-carnitine metabolites
  not covered by the carnitine headgroup template)
- When structure candidates come from CSI:FingerID or NIST hybrid search — those tools
  return annotations directly and do not require building a custom MSP library
edam_operation: http://edamontology.org/operation_0399
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CarniBlast
  role: The in silico tandem MS/MS library built via this skill; a novel library specifically
    geared at acylcarnitines used for MSI level 2 annotation
- name: NIST17
  role: Spectral search engine queried against the formatted MSP library during MS/MS
    matching of untargeted metabolomics spectra
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:7a890476f849d887082cbffc103d53b43f919fe3bd86b1e995c3a3595061a35e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CarniBlast
  - NIST17
  edam_operation: http://edamontology.org/operation_0399
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# msp-spectral-library-formatting

## Summary

Assembles enumerated metabolite candidate structures (e.g. 2,400 acylcarnitine species from the CarniBlast template approach) into a NIST MSP-format tandem MS/MS spectral library so that precursor m/z, predicted fragment peaks, and structure metadata can be matched against untargeted LC-MS/MS acquisitions.

## When to use

Use when enumerating in silico candidate metabolite structures (e.g. acylcarnitines covering saturated, unsaturated, hydroxyl, keto, dicarboxylic, and oxidized acyl chains) and you need them packaged as a NIST MSP spectral library for MSI level 2 precursor-accurate-mass and MS/MS matching.

## When NOT to use

- When the goal is MSI level 1 identification — that requires authentic standards with precursor mass, retention time, and MS/MS matching, not an in silico library
- When spectra are already in a searchable format or the target compound class is not represented in the enumeration template (e.g. non-acyl-carnitine metabolites not covered by the carnitine headgroup template)
- When structure candidates come from CSI:FingerID or NIST hybrid search — those tools return annotations directly and do not require building a custom MSP library

## Inputs

- Enumerated candidate metabolite structures (e.g. 2,400 acylcarnitine species generated from a carnitine headgroup plus acyl chain template, via SMILES enumeration)
- Template-based fragmentation rules for the compound class (carnitine headgroup fragment ions plus acyl-chain-diagnostic losses)
- Structure metadata per candidate: name, molecular formula, SMILES, ion mode
- Predicted precursor m/z and fragment peak lists per candidate

## Outputs

- NIST MSP-format tandem MS/MS spectral library file (e.g. carniblast_library.msp) with one spectral entry per enumerated candidate
- Validated library metadata confirming entry count (2,400) and coverage of all six acyl-chain modification classes

## How to apply

Define a structural template for the compound class (e.g. the carnitine headgroup plus a variable acyl chain attachment) and enumerate chain modifications across classes such as saturated, unsaturated, hydroxyl, keto, dicarboxylic, and oxidized substitutions until the candidate set is complete (CarniBlast enumerates 2,400 acylcarnitine species). For each candidate, predict tandem MS/MS fragments from template rules — carnitine headgroup fragment ions plus acyl-chain-diagnostic losses — in the style of the LipidBlast in silico library. Assemble per-entry records of precursor m/z, predicted fragment peak lists, and structure metadata (compound name, formula, SMILES, ion mode) into a single NIST MSP-format file. Validate the finished library by parsing the .msp and verifying the total number of spectral entries (2,400 for CarniBlast) and that all modification classes are represented. The library is then used for MS/MS matching against untargeted HILIC-Q Exactive HF and CSH-Q Exactive HF urine metabolomics data, contributing to MSI level 2 annotations alongside precursor accurate mass.

## Related tools

- **CarniBlast** (The in silico tandem MS/MS library built via this skill; a novel library specifically geared at acylcarnitines used for MSI level 2 annotation)
- **NIST17** (Spectral search engine queried against the formatted MSP library during MS/MS matching of untargeted metabolomics spectra)

## Evaluation signals

- Parsing the output .msp yields exactly the expected number of spectral entries (2,400 for CarniBlast)
- All six acyl-chain modification classes (saturated, unsaturated, hydroxyl, keto, dicarboxylic, oxidized) are represented in the parsed library
- Each MSP entry contains required metadata fields: name, formula, SMILES, ion mode, precursor m/z, and a fragment peak list
- Running MS/MS matching against untargeted urine HILIC/CSH Q Exactive HF data produces candidate MSI level 2 annotations attributable to the library

## Limitations

- In silico predicted fragment spectra lack authentic-standard validation, so annotations made via the library are capped at MSI level 2; the article notes many structure assignments 'worthy to be validated by acquiring spectra from corresponding authentic chemicals'
- Coverage is bounded by the enumeration template — classes outside the defined template (e.g. non-acyl-carnitine scaffolds) are not represented
- The field currently lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments in metabolomics MS/MS

## Evidence

- [other] workflow 1: [evidence span withheld — anchor status: not_found]
- [other] workflow 4: [evidence span withheld — anchor status: not_found]
- [other] workflow 5: [evidence span withheld — anchor status: not_found]
