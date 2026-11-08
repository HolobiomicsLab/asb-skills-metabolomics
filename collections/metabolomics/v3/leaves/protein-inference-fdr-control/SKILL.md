---
name: protein-inference-fdr-control
description: Use when spectra have been searched against a protein database and you
  need to infer proteins from peptide identifications and control false discovery
  rate before exporting novel protein-coding gene annotations.
when_to_use_negative:
- Spectra have not yet been converted, centroided, and database-searched — inference/FDR
  is a downstream step and cannot be applied to raw profile-mode spectra
- The goal is peptide-only identification without protein-level grouping or false
  discovery rate assessment
- Data comes from metabolomics or non-proteomics LC-MS workflows for which protein
  inference is not applicable
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0621
tools:
- name: OpenMS
  role: Hosts the protein inference and statistical error control (FDR) tools, plus
    the upstream conversion, centroiding, database searching, feature detection, RT
    alignment and feature linking steps of the proteogenomics workflow
  repo: https://github.com/OpenMS/OpenMS
- name: Python
  role: Used with OpenMS (pyOpenMS bindings) to drive/assemble the workflow and export
    the final identified peptides/proteins to the annotation TSV
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:f0b36703981d5208d3ff136940b18adeffad7fd41830021e05fa03d96d23412d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS
  - Python
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# protein-inference-fdr-control

## Summary

OpenMS-based skill for inferring proteins from identified peptides and applying statistical error control (FDR) after database searching, as used in the GENCODE proteogenomics re-annotation workflow that processed over 55 million public tandem mass spectra.

## When to use

Use when spectra have been searched against a protein database and you need to infer proteins from peptide identifications and control false discovery rate before exporting novel protein-coding gene annotations.

## When NOT to use

- Spectra have not yet been converted, centroided, and database-searched — inference/FDR is a downstream step and cannot be applied to raw profile-mode spectra
- The goal is peptide-only identification without protein-level grouping or false discovery rate assessment
- Data comes from metabolomics or non-proteomics LC-MS workflows for which protein inference is not applicable

## Inputs

- Raw/public tandem MS spectra converted to standardized open formats via OpenMS file conversion and centroiding (e.g. mzML per OpenMS PSI-format support)
- Database search results (peptide-spectrum matches) from OpenMS database searching
- Retention-time aligned, cross-run linked feature results from OpenMS feature detection/alignment/linking
- Protein reference database used for searching (e.g. GENCODE-derived proteome)

## Outputs

- Inferred proteins and their supporting peptides at controlled FDR
- Filtered peptide/protein identifications after statistical error control
- Exported novel protein-coding gene annotation table (proteogenomics_novel_annotations.tsv) comparable against GENCODE

## How to apply

Run protein inference and statistical error control in OpenMS as the two terminal analytical steps of the proteogenomics workflow, after file conversion, centroiding, database searching, feature detection, retention-time alignment and cross-run feature linking. Feed the cross-run linked feature results (from the OpenMS retention-time alignment and feature-linking stage) into protein inference, then apply FDR-based statistical error control to the inferred identifications so that only confidently identified peptides/proteins are retained. Finally, export the identified peptides/proteins and derived novel protein-coding gene annotations to a TSV file (proteogenomics_novel_annotations.tsv) that can be compared against the GENCODE human reference annotation. In the GENCODE case this step operated on over 55 million tandem mass spectra from public datasets searched with an OpenMS proteogenomics workflow, yielding over 40 new protein-coding gene annotations; validate by checking that the exported table reproduces the reported set of novel annotations with all workflow steps completing without error.

## Related tools

- **OpenMS** (Hosts the protein inference and statistical error control (FDR) tools, plus the upstream conversion, centroiding, database searching, feature detection, RT alignment and feature linking steps of the proteogenomics workflow) — https://github.com/OpenMS/OpenMS
- **Python** (Used with OpenMS (pyOpenMS bindings) to drive/assemble the workflow and export the final identified peptides/proteins to the annotation TSV) — https://github.com/OpenMS/OpenMS

## Examples

```
# Within the OpenMS TOPP/pyOpenMS pipeline: ProteinInference → FDR → export proteogenomics_novel_annotations.tsv
python -c "import pyopenms as oms"  # run OpenMS protein inference + FDR tools, then write TSV output
```

## Evaluation signals

- Exported proteogenomics_novel_annotations.tsv reproduces the reported set of novel protein-coding gene annotations relative to the GENCODE reference (over 40 new annotations in the published run)
- All workflow steps from file conversion through protein inference and statistical error control complete without error
- FDR control removes low-confidence identifications, leaving confidently identified peptides/proteins exported downstream
- Novel annotations are consistent with peptide identifications at the controlled FDR and trace back to the searched protein reference database

## Limitations

- Result quality depends on the upstream chain (conversion, centroiding, database searching, feature detection, RT alignment, feature linking); errors upstream propagate into inference
- FDR thresholds and inference settings must be chosen appropriately; the exported annotation set is validated only by reproduction of the reference annotation count in this workflow
- Applies to proteomics/proteogenomics data; not a general-purpose identifications filter for metabolomics workflows

## Evidence

- [methods] Export the final identified peptides/proteins and derived novel protein-coding gene annotations to proteogenomics_novel_annotations.tsv. Validation: accept the run if the exported table reproduces the reported set of novel protein-coding gene annotations relative to the reference annotation, with all workflow steps (conversion through error control) completing without error.: [evidence span withheld — anchor status: not_found]
- [readme] It supports easy integration of OpenMS built tools into workflow engines like nextflow, KNIME, Galaxy, and TOPPAS via the TOPPTools concept and a unified parameter handling via a 'common tool description' (CTD) scheme.: [evidence span withheld — anchor status: external_doc]
- [readme] The main contributors of OpenMS are currently the Eberhard-Karls-Universität in Tübingen, the Freie Universität Berlin, and the ETH Zürich.: [evidence span withheld — anchor status: external_doc]
