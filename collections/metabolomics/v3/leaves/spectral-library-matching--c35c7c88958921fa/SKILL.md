---
name: spectral-library-matching
description: Use when you have deconvoluted MS/MS peak lists from untargeted LC-MS/MS
  runs (e.g., HILIC or CSH Q Exactive HF) and need to assign compound identifications
  at graded MSI confidence levels 1-3.
when_to_use_negative:
- You already have a feature/annotation table with assigned compound identities and
  no raw MS/MS spectra to match
- You need absolute compound identification without retention-time standards — level
  1 assignments require authentic-standard matching of precursor mass, retention time,
  and MS/MS; mass and MS/MS alone cap at level 2
- You require validated false-discovery rates — the article notes the field lacks
  'generally accepted and validated automated calculations of compound identification
  confidence levels with false-discovery rate assessments'
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: NIST17
  role: Library searched with precursor accurate mass and MS/MS matching for MSI level
    2 annotations and hybrid search for MSI level 3
- name: NIST hybrid search
  role: Combines in-silico predicted fragments with library spectra to annotate modified
    metabolites against parent library entries
- name: CarniBlast
  role: Novel library specifically geared at acylcarnitines, adding 67 level 2 annotations
- name: CSI:FingerID
  role: In-silico fragmentation tool for annotating remaining compounds at MSI level
    3; available via SIRIUS web service
  repo: https://github.com/sirius-ms/sirius
- name: ClassyFire
  role: Ontology classification of level 3 annotations into 21 superclasses / 184
    chemical classes
- name: Sirius/CSI:Finger ID
  role: Java-based framework for LC-MS/MS analysis; fragmentation tree and isotope
    pattern computation feeding CSI:FingerID web services
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:1243787a97d6047687e17029ae70b6602c5ac33a0a62a782641e0a176f102fdd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - CarniBlast
  - ClassyFire
  - NIST hybrid search
  - NIST17
  - Sirius/CSI:Finger ID
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-library-matching

## Summary

Assigns MSI confidence levels to MS/MS spectra from untargeted metabolomics by matching precursor accurate mass, retention time, and fragment ions against authentic-standard libraries (MSI level 1), MoNA/NIST17 and CarniBlast (MSI level 2), and the NIST17 hybrid search (MSI level 3).

## When to use

Use when you have deconvoluted MS/MS peak lists from untargeted LC-MS/MS runs (e.g., HILIC or CSH Q Exactive HF) and need to assign compound identifications at graded MSI confidence levels 1-3.

## When NOT to use

- You already have a feature/annotation table with assigned compound identities and no raw MS/MS spectra to match
- You need absolute compound identification without retention-time standards — level 1 assignments require authentic-standard matching of precursor mass, retention time, and MS/MS; mass and MS/MS alone cap at level 2
- You require validated false-discovery rates — the article notes the field lacks 'generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments'

## Inputs

- Deconvoluted MS/MS spectra (.mgf format) exported from HILIC-Q Exactive HF and CSH-Q Exactive HF runs
- In-house authentic-standard spectral library (precursor mass + retention time + MS/MS)
- MoNA and NIST17 spectral libraries
- CarniBlast acylcarnitine-specific library
- Modified-metabolite MS/MS spectra with non-modified parent library entries (e.g., methyladenosine vs adenosine) for hybrid search

## Outputs

- Annotated compound table with MSI confidence level per compound (175 level 1, 578 level 2)
- NIST17 hybrid search assignments for ~95% of spectra (MSI level 3)
- ClassyFire classification summary (21 superclasses / 184 chemical classes, top classes: 28% organic acid derivatives, 16% heterocyclics, 16% lipids)
- Verification table of modified-metabolite-to-parent matches with precursor/fragment shifts and match scores

## How to apply

First search each MS/MS spectrum against an in-house authentic-standard spectral library requiring agreement in precursor mass, retention time, and MS/MS fragment ions to assign MSI level 1 identifications (175 compounds in this urine study). For unmatched spectra, search against MoNA and NIST17 using precursor accurate mass and MS/MS matching alone to assign MSI level 2 (480 compounds), and search acylcarnitine-bearing spectra against the CarniBlast library to add 67 additional level 2 acylcarnitine annotations (578 total). For all remaining spectra, run the NIST17 hybrid search, which combines in-silico predicted fragments with library spectra via head-to-tail comparison of precursor and fragment mass shifts (e.g., +14 Da methylation, +80 Da phosphorylation, +42 Da acetylation) to link modified metabolites to parent library entries — this assigned 95% of exported patient spectra within 10 minutes. Optionally run CSI:FingerID on the remaining compounds for in-silico structure annotation; note it yielded only 40% top-ranked accuracy when tested against level 1 standards. Map final annotations to InChI keys via the Chemical Translation Service and classify with ClassyFire (21 superclasses, 184 classes).

## Related tools

- **NIST17** (Library searched with precursor accurate mass and MS/MS matching for MSI level 2 annotations and hybrid search for MSI level 3)
- **NIST hybrid search** (Combines in-silico predicted fragments with library spectra to annotate modified metabolites against parent library entries)
- **CarniBlast** (Novel library specifically geared at acylcarnitines, adding 67 level 2 annotations)
- **CSI:FingerID** (In-silico fragmentation tool for annotating remaining compounds at MSI level 3; available via SIRIUS web service) — https://github.com/sirius-ms/sirius
- **ClassyFire** (Ontology classification of level 3 annotations into 21 superclasses / 184 chemical classes)
- **Sirius/CSI:Finger ID** (Java-based framework for LC-MS/MS analysis; fragmentation tree and isotope pattern computation feeding CSI:FingerID web services) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input spectra.mgf --output sirius_results --project myproject; # then export .mgf and batch-search against NIST17 hybrid search, mapping results to InChI keys via CTS before ClassyFire submission
```

## Evaluation signals

- MSI level 1 tally equals 175, with every level 1 hit supported by precursor mass, retention time, AND MS/MS matching against authentic standards
- MSI level 2 tally equals 578 (480 via MoNA+NIST17 precursor-mass/MS-MS matching plus 67 via CarniBlast acylcarnitine annotations)
- Fraction of MS/MS spectra assigned by NIST17 hybrid search reaches approximately 95%, processed within ~10 minutes
- ClassyFire superclass count sums to 21 with 184 chemical classes, and top three classes match 28% organic acid derivatives, 16% heterocyclics, 16% lipids
- For the four hybrid-search demo pairs (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine), each modified spectrum's top-scoring match is the biologically expected parent (adenosine, threonine, arginine, methionine)

## Limitations

- Top-ranked CSI:FingerID annotations yielded only 40% accuracy when tested against MSI level 1 identified compounds, so in-silico annotations require validation with authentic chemicals
- Majority of acquired MS/MS spectra historically cannot be annotated with certainty to specific chemicals; level 3 annotations are the lowest-confidence tier
- The field lacks generally accepted and validated automated calculations of identification confidence levels with false-discovery rate assessments in metabolomics MS/MS
- MSI level 3 class distributions may reflect differences in subjects due to diet and chemical exposure (e.g., in epidemiology studies), complicating cross-study comparison

## Evidence

- [readme] SIRIUS framework description from README: [evidence span withheld — anchor status: external_doc]
