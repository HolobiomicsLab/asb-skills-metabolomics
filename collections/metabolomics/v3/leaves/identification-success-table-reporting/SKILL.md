---
name: identification-success-table-reporting
description: Use when DeepMASS structural-similarity scores have been produced for
  external-test MS/MS spectra (e.g. male-infertility biomarkers on Bruker Compact
  LC-ESI-QTOF) and per-spectrum hit/success tables must be reported.
when_to_use_negative:
- The query metabolite's exact structure is already present in the database, so ordinary
  library matching (not known-to-unknown structural-similarity scoring) suffices
- Data are MS1-only or non-tandem spectra, which cannot drive the DeepMASS MS/MS-based
  structural-similarity model
- The MetDNA-derived experimental spectra are being reused beyond reproducing the
  DeepMASS paper results without the owner's permission (dataset was removed from
  the repo at the owner's request)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_2258
tools:
- name: DeepMASS
  role: Deep-neural-network scorer whose ranked structural-similarity hits and success
    labels populate the reported tables
  repo: github.com/hcji/DeepMASS
- name: KEGG
  role: Known-metabolite reference database whose 662 compounds provide candidates
    and define the structure-related-metabolite success criterion
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument source of the external-test biomarker MS/MS spectra reported in
    the per-instrument success summary
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument source of the training/benchmark biomarker MS/MS spectra compared
    against the QTOF external-test results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:1a998ea1e22f802be74dbce2349244198bcb30c92fdac6d0d28bd29a699f52f0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# identification-success-table-reporting

## Summary

Tabulates DeepMASS top-ranked candidate metabolites and their structural-similarity scores for each query MS/MS spectrum, then labels identification success using the paper's criterion that an unknown is validly identified only when a structure-related metabolite exists in the database (KEGG).

## When to use

Use when DeepMASS structural-similarity scores have been produced for external-test MS/MS spectra (e.g. male-infertility biomarkers on Bruker Compact LC-ESI-QTOF) and per-spectrum hit/success tables must be reported.

## When NOT to use

- The query metabolite's exact structure is already present in the database, so ordinary library matching (not known-to-unknown structural-similarity scoring) suffices
- Data are MS1-only or non-tandem spectra, which cannot drive the DeepMASS MS/MS-based structural-similarity model
- The MetDNA-derived experimental spectra are being reused beyond reproducing the DeepMASS paper results without the owner's permission (dataset was removed from the repo at the owner's request)

## Inputs

- External-test MS/MS spectra of male-infertility biomarker peaks (Bruker Compact LC-ESI-QTOF and Shimadzu LC-ESI-IT-TOF)
- Ranked candidate lists with DeepMASS structural-similarity scores from the deep-neural-network scorer
- KEGG metabolite database entries used as the known-metabolite reference set

## Outputs

- Per-spectrum ranked-hit table (candidate metabolite, structural-similarity score)
- Identification-success table (valid/not-valid label per spectrum per the structure-related-metabolite criterion)
- Per-instrument identification performance summary written to file

## How to apply

Load each external-test MS/MS spectrum (male-infertility biomarker spectra from the Bruker Compact LC-ESI-QTOF, alongside Shimadzu LC-ESI-IT-TOF data) and run it through the trained DeepMASS deep-neural-network scorer to rank KEGG database metabolites by structural-similarity score. Record the top-ranked hits and their scores for every spectrum rather than only the single best hit, since identification relies on structural relatedness, not exact spectral matching. Apply the paper's success criterion: an identification is counted as valid when a structure-related metabolite of the query is available in the database. Aggregate results into a per-instrument identification-performance summary (Shimadzu LC-ESI-IT-TOF vs Bruker Compact LC-ESI-QTOF) and write both the ranked-hit table and the identification-success table to a file. Benchmark plausibility against the leave-one-out cross-validation done on MS/MS spectra of 662 KEGG compounds.

## Related tools

- **DeepMASS** (Deep-neural-network scorer whose ranked structural-similarity hits and success labels populate the reported tables) — github.com/hcji/DeepMASS
- **KEGG** (Known-metabolite reference database whose 662 compounds provide candidates and define the structure-related-metabolite success criterion)
- **Bruker Compact LC-ESI-QTOF** (Instrument source of the external-test biomarker MS/MS spectra reported in the per-instrument success summary)
- **Shimadzu LC-ESI-IT-TOF** (Instrument source of the training/benchmark biomarker MS/MS spectra compared against the QTOF external-test results)

## Examples

```
python example.py
```

## Evaluation signals

- The ranked-hit table contains one entry (with a structural-similarity score) for every input spectrum, not only confidently identified ones
- Every success label follows the paper criterion: valid iff a structure-related metabolite of the query is present in the database
- The summary separates instrument performance (Shimadzu LC-ESI-IT-TOF vs Bruker Compact LC-ESI-QTOF) as required by the external-test design
- Ranking plausibility is checked against the leave-one-out cross-validation baseline on 662 KEGG compounds
- Both the ranked-hit table and the identification-success table are persisted to files, not just printed

## Limitations

- Identification is valid only when a structure-related metabolite is available in the database; unknowns without structurally related database entries cannot be validated
- The experimental spectra (MetDNA dataset) were removed from the repository at the owner's request, so the package cannot be run directly to reproduce tables without obtaining the data
- A separately downloaded pretrained model (model-40V.zip) must be unzipped into /model/40V before scoring; without it the ranked-hit generation cannot run

## Evidence

- [readme] DeepMASS is a known-to-unknown metabolite identification workflow, which includes a deep-learning based model to predict structural similarity: [evidence span withheld — anchor status: external_doc]
- [readme] Since the experimental spectra has been removed, this package cannot be run directly. You can train your own model with your in-house database.: [evidence span withheld — anchor status: external_doc]
