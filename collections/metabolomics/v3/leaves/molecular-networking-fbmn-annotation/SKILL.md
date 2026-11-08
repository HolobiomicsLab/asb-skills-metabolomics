---
name: molecular-networking-fbmn-annotation
description: Use when you have MZmine-exported feature quantification tables (.csv)
  and .mgf fragmentation spectra — especially ions present only in a scheduled precursor
  list and never fragmented by DDA — that need multi-level annotation.
when_to_use_negative:
- Nodes that already have a direct spectral library match from the FBMN/GNPS2 spectral
  pairing step — no propagation or in silico layer is needed for them
- List-only ions for which no fragmentation spectrum (.mgf entry) exists, since FBMN,
  ChemWalker, and SIRIUS all require MS/MS spectra
- Data acquired or preprocessed outside MZmine without a GNPS-FBMN-compatible export
  (row ID, m/z, retention time, and MS2-filtered rows)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GNPS2
  role: Platform hosting the Feature-Based Molecular Networking (FBMN) workflow for
    molecular networking and spectral pairing annotation, and the graphical interface
    for ChemWalker annotation propagation
- name: GNPS2 library
  role: Public spectral library searched during spectral pairing to assign first-level
    annotations
- name: ChemWalker
  role: Propagates annotations along the molecular network topology for nodes without
    spectral library matches, using adduct [M+H]+ and 15 ppm tolerance
- name: SIRIUS 5.8.5
  role: Second in silico annotation layer for still-unannotated spectra, using Q-TOF
    parameters and formulas from all databases with adducts [M+H]+, [M+K]+, [M+Na]+
- name: MZmine
  role: Preprocessing tool whose exports (feature quantification .csv and .mgf spectra)
    feed the FBMN workflow; Export/Submit to GNPS-FBMN filters rows to only those
    with MS2
- name: RegFilter
  role: Upstream feature selection tool producing the scheduled precursor list whose
    list-only ions are annotated by this skill
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:bd50c79abea24cd92c080cdf4bbe4cccf3b77f14c87cf1a54de4a3818a01833d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - ChemWalker
  - GNPS
  - GNPS2
  - GNPS2 library
  - MZmine
  - RegFilter
  - SIRIUS 5.8.5
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# molecular-networking-fbmn-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Three-level annotation of fragmentation spectra — GNPS2 FBMN spectral pairing, ChemWalker annotation propagation over the molecular network, and SIRIUS 5.8.5 in silico annotation — used here to annotate list-only precursor ions from RegFilter scheduled precursor lists that DDA mode missed, recovering 43 (BRA006), 46 (BRA010), and 47 (BRA177) annotations.

## When to use

Use when you have MZmine-exported feature quantification tables (.csv) and .mgf fragmentation spectra — especially ions present only in a scheduled precursor list and never fragmented by DDA — that need multi-level annotation.

## When NOT to use

- Nodes that already have a direct spectral library match from the FBMN/GNPS2 spectral pairing step — no propagation or in silico layer is needed for them
- List-only ions for which no fragmentation spectrum (.mgf entry) exists, since FBMN, ChemWalker, and SIRIUS all require MS/MS spectra
- Data acquired or preprocessed outside MZmine without a GNPS-FBMN-compatible export (row ID, m/z, retention time, and MS2-filtered rows)

## Inputs

- MZmine-exported SPL feature quantification table (.csv, with row ID, row m/z, row retention time)
- List-only fragmentation spectra in .mgf format
- GNPS2 account/platform access for FBMN and ChemWalker web interface

## Outputs

- Molecular network with spectral pairing annotations from the GNPS2 library
- ChemWalker-propagated annotations for network nodes lacking direct library matches
- SIRIUS best-ranked candidate formula/annotation per unannotated spectrum
- Compiled per-strain annotation table (e.g. 43 annotations for BRA006, 46 for BRA010, 47 for BRA177)

## How to apply

Import the MZmine-exported SPL feature quantification table (.csv) and list-only fragmentation spectra (.mgf) into GNPS2 and run the Feature-Based Molecular Networking (FBMN) workflow (Nothias et al., 2020) so molecular networking and spectral pairing annotation are performed against the GNPS2 library, one of the largest public spectral libraries. For nodes without spectral library matches, propagate annotations along the molecular network topology using ChemWalker through its GNPS2 graphical interface, restricting to adduct [M+H]+ with a 15 ppm tolerance. For still-unannotated nodes, run SIRIUS 5.8.5 with recommended Q-TOF instrument parameters, searching formulas from all databases and considering adducts [M+H]+, [M+K]+, and [M+Na]+. Assign the best-ranked candidate as the annotation for each spectrum and compile the per-strain results (BRA006, BRA010, BRA177) into a single table.

## Related tools

- **GNPS2** (Platform hosting the Feature-Based Molecular Networking (FBMN) workflow for molecular networking and spectral pairing annotation, and the graphical interface for ChemWalker annotation propagation)
- **GNPS2 library** (Public spectral library searched during spectral pairing to assign first-level annotations)
- **ChemWalker** (Propagates annotations along the molecular network topology for nodes without spectral library matches, using adduct [M+H]+ and 15 ppm tolerance)
- **SIRIUS 5.8.5** (Second in silico annotation layer for still-unannotated spectra, using Q-TOF parameters and formulas from all databases with adducts [M+H]+, [M+K]+, [M+Na]+)
- **MZmine** (Preprocessing tool whose exports (feature quantification .csv and .mgf spectra) feed the FBMN workflow; Export/Submit to GNPS-FBMN filters rows to only those with MS2)
- **RegFilter** (Upstream feature selection tool producing the scheduled precursor list whose list-only ions are annotated by this skill) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
sirius --qtof --adducts "[M+H]+,[M+K]+,[M+Na]+" --db all spectra.mgf
```

## Evaluation signals

- Recovered list-only ion annotation counts per strain match reported values: BRA006 = 43, BRA010 = 46, BRA177 = 47
- ChemWalker propagation is restricted to adduct [M+H]+ with 15 ppm tolerance; any annotation outside this criterion is invalid
- SIRIUS is run with version 5.8.5, Q-TOF instrument parameters, all formula databases, and adducts [M+H]+, [M+K]+, [M+Na]+
- Each list-only spectrum receives at most one annotation: the best-ranked candidate from spectral pairing, ChemWalker propagation, or SIRIUS in that priority order
- The final annotation table is compiled per strain and covers only nodes lacking direct GNPS2 library matches for the propagation/in silico layers

## Limitations

- The stochastic nature of DDA means ions with similar intensities may or may not be fragmented, so DDA-fragmented spectra and SPL-only spectra must be reconciled before annotation
- Almost all precursor ions selected in the fragmentation list were present in the MS1 of the DDA method but not all were fragmented; ions with low m/z were more frequently captured by the SPL than fragmented by DDA
- RegFilter can select ions of low relative abundance, which DDA misses, but such low-intensity spectra may yield weaker annotations
- The article notes no code repository or DOI is provided for the RegFilter package despite its mention as a newly introduced tool

## Evidence
