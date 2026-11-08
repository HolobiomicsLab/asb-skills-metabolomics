---
name: openms-workflow-execution
description: Use when the analysis goal is to re-annotate a reference genome (e.
when_to_use_negative:
- Input is already a processed peptide/protein identification table or a quantified
  feature matrix — the pipeline assumes raw or profile-mode spectra.
- The reference genome annotation is not available as a protein FASTA/database; database
  searching requires a protein sequence reference.
- The goal is de novo peptide sequencing without a genome annotation reference; OpenMS
  supports de novo search, but this skill targets reference-driven re-annotation.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: Core C++ library and TOPP tool suite used for file conversion, centroiding,
    database searching, feature detection, retention-time alignment, cross-run feature
    linking, protein inference, and FDR error control.
  repo: https://github.com/OpenMS/OpenMS
- name: pyOpenMS
  role: Python bindings to the OpenMS API enabling rapid scriptable implementation
    of the proteogenomics workflow steps.
  repo: https://github.com/OpenMS/OpenMS
- name: Comet
  role: State-of-the-art external database-search engine adapter available within
    OpenMS for searching spectra against the reference protein database.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:95fccb0932cfb89b010ad49f87f805905fa6d1e326ab320461fe3f131f1479cb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - Comet
  - OpenMS
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# openms-workflow-execution

## Summary

Executing the OpenMS proteogenomics re-annotation pipeline: convert public tandem MS spectra to standardized open formats, centroid profile-mode data, search against a reference protein database, detect and align features across runs, infer proteins with FDR error control, and export novel protein-coding gene annotations. This skill packages the OpenMS toolchain (150+ TOPP tools, pyOpenMS bindings) into a reproducible end-to-end workflow for discovering new gene annotations from MS data.

## When to use

Use when the analysis goal is to re-annotate a reference genome (e.g., GENCODE) from public tandem mass spectra, or when a pipeline must process profile-mode LC-MS/MS data through file conversion, centroiding, database searching, feature detection, RT alignment, feature linking, protein inference, and statistical error control.

## When NOT to use

- Input is already a processed peptide/protein identification table or a quantified feature matrix — the pipeline assumes raw or profile-mode spectra.
- The reference genome annotation is not available as a protein FASTA/database; database searching requires a protein sequence reference.
- The goal is de novo peptide sequencing without a genome annotation reference; OpenMS supports de novo search, but this skill targets reference-driven re-annotation.

## Inputs

- Public tandem mass spectra in raw instrument or community formats (mzML, mzXML)
- Protein reference database derived from a genome annotation (e.g., GENCODE human reference)
- Profile-mode LC-MS/MS runs for centroiding

## Outputs

- Identified peptides and inferred proteins with statistical error control (FDR)
- Retention-time-aligned and cross-run linked feature tables
- Proteogenomics novel protein-coding gene annotation table (e.g., proteogenomics_novel_annotations.tsv)

## How to apply

Load public tandem MS spectra and convert them into a standardized open format using OpenMS file-conversion tooling (e.g., mzML/mzXML). Centroid profile-mode spectra using OpenMS centroiding tools, then search the spectra against a protein reference database (e.g., GENCODE) via OpenMS database-search adapters (Comet or built-in). Perform feature detection on search results, align retention times across runs, and link cross-run features with OpenMS RT-alignment and feature-linking tools. Carry out protein inference and apply statistical error control (FDR) before exporting identified peptides/proteins and derived novel protein-coding gene annotations (e.g., to a TSV such as proteogenomics_novel_annotations.tsv). Validate the run by checking that the exported annotation table reproduces the expected novel protein-coding gene set relative to the reference annotation with all workflow steps (conversion through error control) completing without error.

## Related tools

- **OpenMS** (Core C++ library and TOPP tool suite used for file conversion, centroiding, database searching, feature detection, retention-time alignment, cross-run feature linking, protein inference, and FDR error control.) — https://github.com/OpenMS/OpenMS
- **pyOpenMS** (Python bindings to the OpenMS API enabling rapid scriptable implementation of the proteogenomics workflow steps.) — https://github.com/OpenMS/OpenMS
- **Comet** (State-of-the-art external database-search engine adapter available within OpenMS for searching spectra against the reference protein database.)

## Examples

```
python -c "import pyopenms; from pyopenms import FileConverter, Centroidizer, CometAdapter, FeatureFinder, MapAlignerPoseClustering, ProteinInference, FDRHandler";
```

## Evaluation signals

- All workflow steps (file conversion through statistical error control) complete without errors in the OpenMS tool logs.
- The exported annotation table reproduces the reported set of novel protein-coding gene annotations relative to the reference annotation.
- Feature detection yields peptide features in every input run, and RT alignment maps them consistently across runs.
- Peptide-spectrum matches pass the applied FDR threshold, and protein inference returns the expected identified protein set.
- Output files follow the OpenMS standardized open formats (mzML, featureXML, idXML) and the final annotation export is a valid TSV with the expected columns.

## Limitations

- OpenMS is optimized for LC-MS proteomics and metabolomics; non-standard data types or vendor-locked raw formats may require extra conversion steps.
- Over 55 million tandem mass spectra were needed in the source study to discover over 40 new protein-coding gene annotations — shallow input data will yield few or no novel annotations.
- pyOpenMS bindings must be regenerated when the C++ API changes; wrapping instructions live in src/pyOpenMS/CLAUDE.md.

## Evidence

- [readme] OpenMS library description: [evidence span withheld — anchor status: not_found]
- [readme] pyOpenMS description: [evidence span withheld — anchor status: external_doc]
