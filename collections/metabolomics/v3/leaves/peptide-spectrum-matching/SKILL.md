---
name: peptide-spectrum-matching
description: Use when raw centroided MS/MS spectra need to be assigned peptide identities
  by searching them against a protein reference database, e.g. after converting public
  tandem MS data into a standardized open format.
when_to_use_negative:
- Input is already a list of identified peptides or a feature table — downstream feature
  detection and linking steps apply instead.
- The task is targeted quantitation or SWATH/DIA extraction rather than discovery-style
  database searching.
- Spectra are profile-mode and uncentroided — centroiding must precede database searching.
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: OpenMS
  role: Provides the database searching step that assigns peptide-spectrum matches
    from centroided spectra against the protein reference database; also supplies
    the surrounding file conversion, centroiding, feature detection, retention-time
    alignment, feature linking, protein inference and FDR control that share its API.
  repo: https://github.com/OpenMS/OpenMS
- name: Python
  role: Accessible via the pyOpenMS bindings to drive the search and surrounding workflow
    programmatically.
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:9f7326d5d09d20bcd407edba938baa85e77ca33ff997a61ebccac6bee3dd9e09
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS
  - Python
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# peptide-spectrum-matching

## Summary

Matches experimental tandem MS/MS spectra against theoretical fragment spectra from a protein sequence database using OpenMS database searching, producing peptide-spectrum matches (PSMs) that feed protein inference and downstream proteogenomics annotation.

## When to use

Use when raw centroided MS/MS spectra need to be assigned peptide identities by searching them against a protein reference database, e.g. after converting public tandem MS data into a standardized open format.

## When NOT to use

- Input is already a list of identified peptides or a feature table — downstream feature detection and linking steps apply instead.
- The task is targeted quantitation or SWATH/DIA extraction rather than discovery-style database searching.
- Spectra are profile-mode and uncentroided — centroiding must precede database searching.

## Inputs

- centroided tandem mass spectra (e.g. mzML) from public proteomics datasets
- protein reference database (FASTA) including gene annotations to be tested for proteogenomic evidence

## Outputs

- peptide-spectrum matches (PSMs) with scores
- identified peptides/proteins exportable to proteogenomics_novel_annotations.tsv

## How to apply

First convert and centroid the profile-mode spectra with OpenMS file-conversion and centroiding tools so that peak lists are in a standardized open data format such as mzML. Then run OpenMS database searching (e.g. via its search adapters such as Comet) against the protein reference database, using the tool's precursor and fragment mass tolerances appropriate for the instrument type. OpenMS assigns peptide-spectrum matches and scores them for statistical error control (FDR) in subsequent protein inference steps. In the proteogenomics workflow the search is one step in a chain that also includes feature detection, retention-time alignment, and cross-run feature linking, so PSM outputs must be compatible with those downstream consumers. Acceptance is judged by the identified peptides/proteins being exportable and reproducible against a reference annotation.

## Related tools

- **OpenMS** (Provides the database searching step that assigns peptide-spectrum matches from centroided spectra against the protein reference database; also supplies the surrounding file conversion, centroiding, feature detection, retention-time alignment, feature linking, protein inference and FDR control that share its API.) — https://github.com/OpenMS/OpenMS
- **Python** (Accessible via the pyOpenMS bindings to drive the search and surrounding workflow programmatically.) — https://github.com/OpenMS/OpenMS

## Examples

```
python -c "import pyopenms; print(pyopenms.__version__)"  # drive OpenMS search via pyOpenMS bindings
```

## Evaluation signals

- All workflow steps (conversion through error control) complete without error before PSMs are accepted.
- Exported identified peptides/proteins reproduce the reported set of novel protein-coding gene annotations relative to the reference annotation.
- PSM outputs are compatible with downstream feature detection, retention-time alignment, and cross-run feature linking steps.
- FDR-controlled peptide and protein identifications are produced (statistical error control applied in OpenMS).

## Limitations

- OpenMS supports PSI formats for MS data (mzML, mzXML, mzIdentXML, pepXML, mzTab, etc.), so non-PSI inputs may need conversion first.
- OpenMS is designed for flexible, reproducible analysis of high-throughput MS data; results depend on parameters chosen for the instrument type and the completeness of the protein reference database.

## Evidence

- [readme] It offers an infrastructure for rapid development of mass spectrometry-related software: [evidence span withheld — anchor status: external_doc]
- [readme] It provides built-in algorithms for de novo identification and database search, as well as adapters to other state-of-the-art tools like Comet, etc.: [evidence span withheld — anchor status: external_doc]
- [readme] With pyOpenMS, OpenMS offers Python bindings to a large part of the OpenMS API to enable rapid algorithm development.: [evidence span withheld — anchor status: external_doc]
