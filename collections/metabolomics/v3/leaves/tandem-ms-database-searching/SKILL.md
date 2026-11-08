---
name: tandem-ms-database-searching
description: Use when you have centroided tandem MS/MS spectra in an open format (e.g.
  after OpenMS file conversion and centroiding) and need peptide/protein identifications
  via database searching before feature detection, retention-time alignment, protein
  inference, and FDR control.
when_to_use_negative:
- Spectra are raw, unconverted, and still in vendor profile mode — run OpenMS file
  conversion and centroiding first.
- The analysis goal is de novo identification without a reference protein database;
  OpenMS also offers de novo algorithms, which this database-search skill does not
  cover.
- The question is quantification (label-free, SILAC, iTRAQ, TMT, SRM, SWATH) rather
  than peptide identification — database searching alone does not answer that.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: Provides the database searching tools used to match spectra against the protein
    reference database within the proteogenomics workflow
  repo: https://github.com/OpenMS/OpenMS
- name: Comet
  role: State-of-the-art external database-search adapter integrated by OpenMS as
    an alternative to built-in search algorithms
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:7146c722232222e10c1033fec4de6e73e6cb838cdf6c3c7670fc579a9ace40bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - Comet
  - OpenMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tandem-ms-database-searching

## Summary

Search centroided tandem mass spectra against a protein sequence database using OpenMS database-searching tooling to identify peptides and proteins, as one step of the OpenMS proteogenomics workflow described by the article.

## When to use

Use when you have centroided tandem MS/MS spectra in an open format (e.g. after OpenMS file conversion and centroiding) and need peptide/protein identifications via database searching before feature detection, retention-time alignment, protein inference, and FDR control.

## When NOT to use

- Spectra are raw, unconverted, and still in vendor profile mode — run OpenMS file conversion and centroiding first.
- The analysis goal is de novo identification without a reference protein database; OpenMS also offers de novo algorithms, which this database-search skill does not cover.
- The question is quantification (label-free, SILAC, iTRAQ, TMT, SRM, SWATH) rather than peptide identification — database searching alone does not answer that.

## Inputs

- Profile-mode tandem mass spectra from public MS datasets (converted to open formats such as mzML)
- Centroided tandem MS/MS spectra
- Protein reference database (e.g. GENCODE human reference protein sequences)

## Outputs

- Peptide-spectrum matches / identified peptides and proteins from database search
- Features for downstream retention-time alignment and cross-run feature linking
- Exported identifications feeding protein inference, FDR, and proteogenomics_novel_annotations.tsv

## How to apply

Convert raw instrument data into standardized open formats with OpenMS file-conversion tooling, then centroid profile-mode spectra so peaks are suitable for searching. Search the centroided spectra against the protein reference database using OpenMS's database searching (built-in algorithms or adapters such as Comet). Carry the resulting identifications forward through feature detection, retention-time alignment and cross-run feature linking, and protein inference, applying statistical error control (FDR) to the search and inference results. In the Wright et al. proteogenomics case, this chain processed over 55 million public tandem mass spectra to discover over 40 new protein-coding gene annotations in the GENCODE human reference genome. Rationale: database searching is the identification core of the OpenMS workflow, and its scores feed downstream statistical error control; OpenMS 2.0 provides ~185 tools and a unified 'common tool description' (CTD) parameter scheme so the search step integrates into workflow engines like nextflow, KNIME, Galaxy, and TOPPAS.

## Related tools

- **OpenMS** (Provides the database searching tools used to match spectra against the protein reference database within the proteogenomics workflow) — https://github.com/OpenMS/OpenMS
- **Comet** (State-of-the-art external database-search adapter integrated by OpenMS as an alternative to built-in search algorithms)

## Examples

```
via OpenMS TOPPTools after conversion/centroiding: `SearchAdapter`-style tool from the OpenMS tool set (e.g. Comet adapter or built-in search) applied to centroided spectra against the GENCODE protein reference database; e.g. from pyOpenMS: `import pyopenms as oms; oms.MascotGenericFile().write(...)` — or, per README tooling, invoke the database-search TOPP tool with CTD parameters: `OpenMSInfo` / adapter command such as `CometAdapter -in centroided.mzML -database gencode_proteins.fasta -out idXML` (representative; exact CLI per OpenMS tool docs)
```

## Evaluation signals

- All prior workflow steps (file conversion, centroiding) completed without error before searching.
- Search results support the subsequent FDR-filtered peptide/protein set — the exported table reproduces the reported novel protein-coding gene annotations relative to the reference annotation.
- Identifications are statistically controlled: protein inference and FDR are applied to search outputs rather than raw scores.
- The search step completes within the full 7-step OpenMS pipeline (conversion → centroiding → searching → feature detection → RT alignment/linking → inference/FDR → export).

## Limitations

- Database searching depends on upstream quality: uncentroided profile spectra or unconverted vendor formats will degrade identification.
- Search scores alone are not final identifications — results require statistical error control (FDR) and protein inference before reporting.
- Results are only as good as the reference database; the GENCODE example relied on the human reference annotation for interpretation.

## Evidence

- [other] Workflow grounding: seven-step chain ending in exported novel annotations table: [evidence span withheld — anchor status: not_found]
- [other] Database search feeds FDR-controlled inference, validating the search step: [evidence span withheld — anchor status: not_found]
- [readme] OpenMS offers database search with adapters: [evidence span withheld — anchor status: external_doc]
