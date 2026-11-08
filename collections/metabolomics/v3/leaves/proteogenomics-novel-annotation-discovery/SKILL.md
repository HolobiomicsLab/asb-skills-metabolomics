---
name: proteogenomics-novel-annotation-discovery
description: Use when you have large collections of public tandem MS spectra (e.g.
  tens of millions) and want to identify peptides supporting novel protein-coding
  gene annotations beyond an existing genome annotation such as GENCODE.
when_to_use_negative:
- When the goal is routine peptide/protein identification within an already-complete
  annotation and no novel-annotation discovery is intended — standard database searching
  without the proteogenomics export step suffices.
- When input spectra cannot be converted to standardized open formats accepted by
  OpenMS file-conversion tooling.
- When the reference genome annotation is absent, since novel annotations are defined
  relative to the reference annotation and cannot be validated.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0621
tools:
- name: OpenMS
  role: 'Provides the full workflow: file conversion, centroiding, database searching,
    feature detection, retention-time alignment, cross-run feature linking, protein
    inference, and statistical error control (FDR), plus adapters to search engines
    like Comet.'
  repo: https://github.com/OpenMS/OpenMS
- name: Comet
  role: Adapter-supported state-of-the-art database search engine invoked through
    OpenMS for searching spectra against the protein reference database.
- name: Python
  role: pyOpenMS Python bindings to the OpenMS C++ API for rapid algorithm development
    and workflow scripting.
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:cb93675512ac3f1cecd5ea9b868ba0187a343331b28b56e7ab0e96b42e7bb10f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - Comet
  - OpenMS
  - Python
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# proteogenomics-novel-annotation-discovery

## Summary

An OpenMS proteogenomics workflow that searches public tandem mass spectra against a protein reference database, controls FDR through protein inference, and exports peptides mapping outside existing annotation to discover new protein-coding gene annotations (e.g. in the GENCODE human reference genome).

## When to use

Use when you have large collections of public tandem MS spectra (e.g. tens of millions) and want to identify peptides supporting novel protein-coding gene annotations beyond an existing genome annotation such as GENCODE.

## When NOT to use

- When the goal is routine peptide/protein identification within an already-complete annotation and no novel-annotation discovery is intended — standard database searching without the proteogenomics export step suffices.
- When input spectra cannot be converted to standardized open formats accepted by OpenMS file-conversion tooling.
- When the reference genome annotation is absent, since novel annotations are defined relative to the reference annotation and cannot be validated.

## Inputs

- Over 55 million public tandem mass spectra (raw MS data converted to standardized open formats such as mzML/mzXML)
- Protein reference database (e.g. GENCODE human reference proteome with candidate novel coding regions)
- Reference genome annotation for comparison of novel annotations

## Outputs

- proteogenomics_novel_annotations.tsv — final identified peptides/proteins and derived novel protein-coding gene annotations
- FDR-controlled peptide and protein identifications from OpenMS protein inference and statistical error control

## How to apply

Convert the public tandem MS spectra into standardized open formats using OpenMS file-conversion tooling, then centroid profile-mode spectra. Search the spectra against the protein reference database with OpenMS database searching (or adapters such as Comet), perform feature detection, and align retention times across runs before linking cross-run features — this consolidation is what allows spectra from many datasets to contribute jointly. Carry out protein inference and apply statistical error control (FDR) in OpenMS so only confidently identified peptides are kept; unfiltered PSMs would inflate false novel annotations. Finally, export the identified peptides/proteins and derive novel protein-coding gene annotations, writing them to proteogenomics_novel_annotations.tsv; validate by checking that the exported set reproduces the reported novel annotations relative to the reference annotation.

## Related tools

- **OpenMS** (Provides the full workflow: file conversion, centroiding, database searching, feature detection, retention-time alignment, cross-run feature linking, protein inference, and statistical error control (FDR), plus adapters to search engines like Comet.) — https://github.com/OpenMS/OpenMS
- **Comet** (Adapter-supported state-of-the-art database search engine invoked through OpenMS for searching spectra against the protein reference database.)
- **Python** (pyOpenMS Python bindings to the OpenMS C++ API for rapid algorithm development and workflow scripting.) — https://github.com/OpenMS/OpenMS

## Examples

```
python proteogenomics_workflow.py --input spectra.mzML --db gencode_reference.fasta --output proteogenomics_novel_annotations.tsv
```

## Evaluation signals

- All workflow steps from file conversion through statistical error control complete without error.
- Exported proteogenomics_novel_annotations.tsv reproduces the reported set of novel protein-coding gene annotations relative to the reference annotation (over 40 new annotations in the case study).
- FDR control is applied: identifications pass the statistical error-control step of OpenMS protein inference.
- Retention-time alignment and cross-run feature linking complete before protein inference, confirming spectra from multiple public datasets are jointly usable.
- Output tsv contains peptide/protein identifications that map outside the existing GENCODE annotation, consistent with the proteogenomics discovery goal.

## Limitations

- Novel annotations are only as good as the protein reference database; false positives in FDR-controlled identification propagate to false novel gene annotations.
- The workflow depends on OpenMS supporting the input raw format via its file-conversion tooling; unsupported formats fail at the first step.
- As an open-source C++ library with Python bindings (pyOpenMS), large-scale processing (tens of millions of spectra) requires adequate compute and careful parameter handling via the CTD scheme.

## Evidence

- [supplementary] workflow with OpenMS proteogenomics: [evidence span withheld — anchor status: not_found]
- [readme] README citation: [evidence span withheld — anchor status: external_doc]
- [readme] README adapters: [evidence span withheld — anchor status: external_doc]
