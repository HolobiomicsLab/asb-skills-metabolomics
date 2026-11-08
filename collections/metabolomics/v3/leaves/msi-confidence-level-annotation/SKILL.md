---
name: msi-confidence-level-annotation
description: Use when annotating untargeted LC-MS/MS metabolomics spectra (e.g. urine
  on HILIC/CSH Q Exactive HF) with tiered MSI confidence levels 1-3, or when quantifying
  the annotated vs unannotated fraction of MS/MS spectra.
when_to_use_negative:
- Input is already a feature table with MSI level 1/2 annotations assigned — re-deriving
  levels would duplicate prior curation rather than extend it
- The goal is de novo structure elucidation with validated FDR controls — the article
  notes the field lacks 'generally accepted and validated automated calculations of
  compound identification confidence levels with false-discovery rate assessments'
- The analytical chemistry (chromatography/MS conditions) of the m/z+RT library differs
  from the assay — mass-and-RT-only matching is only valid for same-condition libraries
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS/CSI:FingerID
  role: In silico fragmentation and formula/structure candidate ranking for MSI level
    3 annotation of spectra unmatched at levels 1-2
  repo: https://github.com/sirius-ms/sirius
- name: NIST17 hybrid search
  role: Spectral search of remaining compounds at MSI level 3 and ontology-based classification
    of those annotations
- name: CarniBlast
  role: Acylcarnitine-specific spectral library contributing 67 additional MSI level
    2 annotations
- name: ClassyFire
  role: Chemical ontology classification of MSI level 3 annotations into 21 superclasses
    and 184 classes
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:eb572dcb536bdbb8090632ff925a1a00bac685d64290565d25d4162f96a9e677
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CarniBlast
  - ClassyFire
  - NIST17
  - NIST17 hybrid search
  - SIRIUS/CSI:FingerID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# msi-confidence-level-annotation

## Summary

A tiered Metabolomics Standards Initiative (MSI) confidence-level annotation workflow for untargeted urine LC-MS/MS data, assigning compounds to MSI levels 1-3 by progressively relaxing evidence from authentic-standard matching to in silico fragmentation, and quantifying how much of the acquired MS/MS spectra remain unannotated.

## When to use

Use when annotating untargeted LC-MS/MS metabolomics spectra (e.g. urine on HILIC/CSH Q Exactive HF) with tiered MSI confidence levels 1-3, or when quantifying the annotated vs unannotated fraction of MS/MS spectra.

## When NOT to use

- Input is already a feature table with MSI level 1/2 annotations assigned — re-deriving levels would duplicate prior curation rather than extend it
- The goal is de novo structure elucidation with validated FDR controls — the article notes the field lacks 'generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments'
- The analytical chemistry (chromatography/MS conditions) of the m/z+RT library differs from the assay — mass-and-RT-only matching is only valid for same-condition libraries

## Inputs

- Merged urine MS/MS peak lists from HILIC-Q Exactive HF and CSH-Q Exactive HF untargeted assays (accurate precursor m/z, retention time, MS/MS fragment ions)
- In-house authentic-standard spectral library (precursor mass + RT + MS/MS reference records)
- Public spectral libraries: MoNA and NIST17 (with NIST hybrid search)
- CarniBlast acylcarnitine-specific library
- Same-condition m/z and retention-time annotation libraries for mass+RT-only matching
- SIRIUS/CSI:FingerID in silico fragmentation results for remaining unmatched spectra

## Outputs

- Annotation table with each compound tagged by MSI confidence level (1, 2, or 3) and per-level tallies
- mz_rt_annotations.csv: m/z+RT-only annotations flagged with their MSI-confidence rationale (6 acylcarnitines + 201 low-abundance compounds)
- coverage_summary.csv: coverage fraction (753/3894 ≈ 0.193), residual unannotated fraction, numerator, denominator
- ClassyFire superclass/class assignments (21 superclasses, 184 classes) for MSI level 3 annotations

## How to apply

Start with merged MS/MS peak lists from the HILIC-Q Exactive HF and CSH-Q Exactive HF assays and search spectra against an in-house authentic-standard spectral library requiring agreement in precursor accurate mass, retention time, and MS/MS fragment ions to assign MSI level 1 (175 compounds in the reference study). Search remaining unmatched spectra against MoNA and NIST17 using precursor accurate mass and MS/MS matching alone for MSI level 2, adding acylcarnitines via the CarniBlast library (67 extra annotations); combine per-class tallies (39 lipids, 85 acylcarnitines, 454 hydrophilic compounds = 578 level 2). Optionally match features on m/z and retention time alone (no MS/MS) against same-condition in-house libraries, tagging them as mass-and-RT-only (6 additional acylcarnitines, 201 low-abundance compounds). Assign all remaining compounds MSI level 3 using SIRIUS/CSI:FingerID in silico fragmentation and the NIST17 hybrid search, then classify those annotations with ClassyFire (21 superclasses, 184 classes). Finally compute coverage: 753 level 1+2 annotations out of 3,894 acquired MS/MS spectra = 19.3% coverage, leaving >80% unannotated.

## Related tools

- **SIRIUS/CSI:FingerID** (In silico fragmentation and formula/structure candidate ranking for MSI level 3 annotation of spectra unmatched at levels 1-2) — https://github.com/sirius-ms/sirius
- **NIST17 hybrid search** (Spectral search of remaining compounds at MSI level 3 and ontology-based classification of those annotations)
- **CarniBlast** (Acylcarnitine-specific spectral library contributing 67 additional MSI level 2 annotations)
- **ClassyFire** (Chemical ontology classification of MSI level 3 annotations into 21 superclasses and 184 classes)

## Examples

```
python -c "print(753/3894, (3894-753)/3894)"  # coverage 0.193, unannotated 0.807
```

## Evaluation signals

- MSI level 1 tally must equal 175, with every level 1 hit supported by precursor mass, retention time, AND MS/MS matching to an authentic standard
- MSI level 2 tally must equal 578 (480 via MoNA+NIST17 matching plus 67 via CarniBlast), decomposing as 39 lipids, 85 acylcarnitines, 454 hydrophilic compounds
- Coverage fraction must reproduce 753/3894 ≈ 0.193 (19.3%), i.e. more than 80% of MS/MS spectra remain unannotated at levels 1/2
- m/z+RT-only flags must equal exactly 6 acylcarnitines and 201 further low-abundance compounds, each within assay-specific mass and RT tolerance windows
- CSI:FingerID top-ranked annotations should show ~40% accuracy when cross-checked against the MSI level 1 identified compounds

## Limitations

- The majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals — 19.3% coverage means >80% of the 3,894 spectra remain unannotated at MSI level 1/2
- CSI:FingerID top-ranked annotations yield only 40% accuracy against MSI level 1 compounds, so structure assignments require validation by acquiring spectra from corresponding authentic chemicals
- The field lacks generally accepted, validated automated confidence-level calculations with false-discovery rate assessments in metabolomics MS/MS
- MSI level 3 ClassyFire classifications may reflect diet and chemical exposure differences among subjects, complicating epidemiological interpretation

## Evidence

- [readme] SIRIUS is the software framework for the LC-MS/MS analysis: [evidence span withheld — anchor status: external_doc]
