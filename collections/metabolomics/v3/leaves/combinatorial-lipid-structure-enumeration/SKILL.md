---
name: combinatorial-lipid-structure-enumeration
description: Use when a lipid class (e.g., urinary acylcarnitines) is underrepresented
  in public/commercial MS/MS libraries (<50 entries) and you need a template-driven
  in silico library to annotate precursor accurate mass + MS/MS matches at MSI level
  2.
when_to_use_negative:
- The compound class already has comprehensive experimental spectra in public or commercial
  libraries (fewer than 50 acylcarnitine structures was the trigger for building CarniBlast)
- You need MSI level 1 identifications — in silico library matches only support precursor
  accurate mass + MS/MS matching (MSI level 2), not authentic-standard confirmation
- The analyte class lacks a well-defined core template and diagnostic fragment rules,
  so fragment prediction from template losses is not defensible
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CarniBlast
  role: In silico tandem MS/MS library of 2,400 acylcarnitine species produced by
    the enumeration workflow; used for MSI level 2 annotation in untargeted urine
    metabolomics
- name: NIST17
  role: Spectral database searched alongside CarniBlast; NIST17 hybrid search results
    are combined with SIRIUS/CSI:FingerID for structure assignment
- name: SIRIUS/CSI:FingerID
  role: Complementary in silico fragmentation framework for annotating the remaining
    compounds (MSI level 3) that CarniBlast/NIST library matching leaves unassigned
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:6f6d5dbb33260d79a3c742847b8d6e260883c87495f21ea504371d6e184a8d11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CarniBlast
  - NIST17
  - SIRIUS/CSI:FingerID
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# combinatorial-lipid-structure-enumeration

## Summary

Enumerate chemically diverse lipid structures (e.g., acylcarnitines) combinatorially from a core template plus chain-modification classes, and predict their tandem MS/MS fragments to build an in silico spectral library (CarniBlast, 2,400 acylcarnitines in NIST MSP format) that fills the coverage gap of public/commercial libraries holding fewer than 50 acylcarnitine spectra.

## When to use

Use when a lipid class (e.g., urinary acylcarnitines) is underrepresented in public/commercial MS/MS libraries (<50 entries) and you need a template-driven in silico library to annotate precursor accurate mass + MS/MS matches at MSI level 2.

## When NOT to use

- The compound class already has comprehensive experimental spectra in public or commercial libraries (fewer than 50 acylcarnitine structures was the trigger for building CarniBlast)
- You need MSI level 1 identifications — in silico library matches only support precursor accurate mass + MS/MS matching (MSI level 2), not authentic-standard confirmation
- The analyte class lacks a well-defined core template and diagnostic fragment rules, so fragment prediction from template losses is not defensible

## Inputs

- Acylcarnitine core structure template (carnitine headgroup + variable acyl chain attachment)
- Chain-modification class definitions (saturated, unsaturated, hydroxyl, keto, dicarboxylic, oxidized)
- Target structure count / enumeration scope (2,400 acylcarnitine species)
- Untargeted LC-MS/MS data from HILIC-Q Exactive HF and CSH-Q Exactive HF assays for library searching

## Outputs

- NIST MSP-format in silico spectral library (carniblast_library.msp) with 2,400 acylcarnitine entries
- Per-candidate precursor m/z, predicted fragment peaks, and structure metadata (name, formula, SMILES, ion mode)
- MSI level 2 annotations of urinary metabolome signals matched via precursor accurate mass and MS/MS

## How to apply

Define the carnitine headgroup template with a variable acyl chain attachment and enumerate chain-modification classes — saturated, unsaturated, hydroxyl, keto, dicarboxylic, and oxidized — combinatorially over acyl chain lengths (e.g., via RDKit SMILES generation) until 2,400 acylcarnitine species are covered. Predict MS/MS fragments for each candidate from template rules: carnitine headgroup fragment ions plus acyl-chain-diagnostic losses, in the style of the LipidBlast in silico library. Assemble precursor m/z, predicted fragment peaks, and structure metadata (name, formula, SMILES, ion mode) for every candidate into a NIST MSP-format library file (carniblast_library.msp). Search the library against untargeted HILIC- and CSH-Q Exactive HF MS/MS data to yield MSI level 2 annotations by precursor accurate mass and MS/MS matching alone, as done alongside CSI:FingerID and NIST17 hybrid search for higher-level annotation.

## Related tools

- **CarniBlast** (In silico tandem MS/MS library of 2,400 acylcarnitine species produced by the enumeration workflow; used for MSI level 2 annotation in untargeted urine metabolomics)
- **NIST17** (Spectral database searched alongside CarniBlast; NIST17 hybrid search results are combined with SIRIUS/CSI:FingerID for structure assignment)
- **SIRIUS/CSI:FingerID** (Complementary in silico fragmentation framework for annotating the remaining compounds (MSI level 3) that CarniBlast/NIST library matching leaves unassigned) — https://github.com/sirius-ms/sirius

## Examples

```
python build_carniblast.py --core carnitine_headgroup.smiles --mods saturated,unsaturated,hydroxy,keto,dicarboxylic,oxidized --out carniblast_library.msp  # then verify: 2400 entries, 6 modification classes
```

## Evaluation signals

- Parsing carniblast_library.msp yields exactly 2,400 spectral entries
- All six acyl-chain modification classes (saturated, unsaturated, hydroxyl, keto, dicarboxylic, oxidized) are represented among enumerated structures
- Every MSP entry carries precursor m/z, predicted fragment peaks, and structure metadata (name, formula, SMILES, ion mode)
- Library search against the urine datasets annotates spectra at MSI level 2 (article reports 578 compounds annotated by precursor accurate mass and MS/MS matching, including CarniBlast)
- Carnitine headgroup fragment ions and acyl-chain-diagnostic losses are present in predicted spectra for representative candidates

## Limitations

- In silico library matches support only MSI level 2 annotation by precursor accurate mass and MS/MS matching; structure assignments should be validated by acquiring spectra from corresponding authentic chemicals
- The field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments in metabolomics MS/MS, so library hit confidence is hard to quantify
- Enumerated species are template-driven predictions; coverage is bounded by the predefined modification classes and chain-length ranges chosen during enumeration

## Evidence

- [methods] CarniBlast library construction from article task card: [evidence span withheld — anchor status: not_found]
- [intro] Public library coverage gap: [evidence span withheld — anchor status: not_found]
