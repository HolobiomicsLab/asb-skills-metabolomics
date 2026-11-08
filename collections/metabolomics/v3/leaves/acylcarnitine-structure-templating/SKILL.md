---
name: acylcarnitine-structure-templating
description: Use when untargeted LC-MS/MS metabolomics data (e.g. HILIC-Q Exactive
  HF urine runs) contains unannotated precursor+MS/MS spectra that fall in the acylcarnitine
  chemical space beyond the <50 spectra available in public/commercial libraries.
when_to_use_negative:
- When the target metabolite class is not acylcarnitines — the template rules (headgroup
  fragments plus acyl-chain-diagnostic losses) only apply to the carnitine ester scaffold
- When spectra can already be identified at MSI level 1 with authentic standards (precursor
  mass, retention time, and MS/MS matching) — in silico prediction is unnecessary
- When the goal is de novo structure proposal for unknowns — use SIRIUS/CSI:FingerID
  with NIST17 hybrid search instead, since template libraries only cover enumerated
  structures
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CarniBlast
  role: The in silico tandem MS/MS library produced by this templating workflow; geared
    specifically at acylcarnitines for MSI level 2 annotation
- name: NIST17
  role: Reference spectral database environment whose MSP format is used to assemble
    the library; also used alongside for hybrid search validation
- name: RDKit
  role: Combinatorial SMILES generation of candidate acylcarnitine structures from
    chain-length and modification-class enumeration
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:be8e5630f62ce01af7cec5239c828a7eb3e9877fcbd5890d90d249c9ae12ca87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CarniBlast
  - NIST17
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# acylcarnitine-structure-templating

## Summary

Constructs a combinatorial in silico tandem MS/MS library for acylcarnitines (CarniBlast, 2,400 species) by templating the carnitine headgroup plus enumerated acyl-chain modification classes, closing the coverage gap of public/commercial libraries that contain fewer than 50 acylcarnitine spectra.

## When to use

Use when untargeted LC-MS/MS metabolomics data (e.g. HILIC-Q Exactive HF urine runs) contains unannotated precursor+MS/MS spectra that fall in the acylcarnitine chemical space beyond the <50 spectra available in public/commercial libraries.

## When NOT to use

- When the target metabolite class is not acylcarnitines — the template rules (headgroup fragments plus acyl-chain-diagnostic losses) only apply to the carnitine ester scaffold
- When spectra can already be identified at MSI level 1 with authentic standards (precursor mass, retention time, and MS/MS matching) — in silico prediction is unnecessary
- When the goal is de novo structure proposal for unknowns — use SIRIUS/CSI:FingerID with NIST17 hybrid search instead, since template libraries only cover enumerated structures

## Inputs

- Acylcarnitine core structure template (carnitine headgroup + variable acyl chain attachment)
- List of acyl-chain modification classes to enumerate (saturated, unsaturated, hydroxyl, keto, dicarboxylic, oxidized)
- RDKit (Python) for SMILES structure generation

## Outputs

- NIST MSP-format spectral library file (carniblast_library.msp) with 2,400 acylcarnitine spectral entries
- Per-species precursor m/z, predicted fragment peaks, and metadata (name, formula, SMILES, ion mode)

## How to apply

Define the acylcarnitine core template as a carnitine headgroup with a variable acyl-chain attachment, then combinatorially enumerate chain lengths across six modification classes — saturated, unsaturated, hydroxyl, keto, dicarboxylic, and oxidized — generating candidate structures as SMILES (e.g. via RDKit) until 2,400 species are enumerated. Predict MS/MS fragments for each candidate from template rules: carnitine headgroup fragment ions plus acyl-chain-diagnostic neutral losses, in the style of the LipidBlast in silico library. Assemble precursor m/z, predicted fragment peaks, and structure metadata (name, formula, SMILES, ion mode) for every candidate into a NIST MSP-format library file. The resulting carniblast_library.msp is then used for MSI level 2 annotation by precursor accurate mass and MS/MS matching alone, complementing CSI:FingerID/NIST hybrid search at MSI level 3.

## Related tools

- **CarniBlast** (The in silico tandem MS/MS library produced by this templating workflow; geared specifically at acylcarnitines for MSI level 2 annotation)
- **NIST17** (Reference spectral database environment whose MSP format is used to assemble the library; also used alongside for hybrid search validation)
- **RDKit** (Combinatorial SMILES generation of candidate acylcarnitine structures from chain-length and modification-class enumeration)

## Examples

```
python enumerate_acylcarnitines.py --modifications saturated,unsaturated,hydroxyl,keto,dicarboxylic,oxidized --format msp --output carniblast_library.msp
```

## Evaluation signals

- Parsing carniblast_library.msp yields exactly 2,400 spectral entries
- All six acyl-chain modification classes (saturated, unsaturated, hydroxyl, keto, dicarboxylic, oxidized) are represented in the library
- Each MSP entry carries complete metadata: name, formula, SMILES, ion mode, and predicted precursor m/z
- Every entry contains carnitine headgroup fragment ions plus acyl-chain-diagnostic losses consistent with the template rules
- The library produces acylcarnitine annotations counted within the 578 MSI level 2 assignments when applied to untargeted urine LC-MS/MS data

## Limitations

- Spectra are predicted from template rules (LipidBlast-style), not measured, so annotations are MSI level 2 (accurate mass + MS/MS matching) and require validation with authentic chemicals
- Coverage is limited to the enumerated modification classes and chain lengths; acylcarnitine structures outside the six classes are not represented
- In-source fragmentations and co-eluting interferences in DDA data (only 42% of 9,000+ urine signals triggered MS/MS) can leave acylcarnitines unannotated despite library coverage

## Evidence

- [other] Library construction template: [evidence span withheld — anchor status: not_found]
- [other] Fragment prediction style: [evidence span withheld — anchor status: not_found]
- [other] Validation criteria: [evidence span withheld — anchor status: not_found]
