---
name: massive-dataset-retrieval
description: Use when a published lipidomics workflow (e.g., LipiDex 2 Peak Finder
  benchmarks) cites a MassIVE accession such as MSV000091033 and the goal is to download
  the deposited instrument files for independent reprocessing and count verification.
when_to_use_negative:
- The target dataset is not publicly deposited under a MassIVE accession (e.g., raw
  files are only in a proprietary local archive or a different repository without
  an accession ID).
- The goal is to analyze an already-extracted peak or feature table rather than to
  reprocess original instrument files from a cited public deposit.
edam_operation: http://edamontology.org/operation_2422
tools:
- name: LipiDex 2
  role: Downstream pipeline that consumes the retrieved public dataset; the paper
    explicitly demonstrates reprocessing MSV000091033 with LipiDex 2 modules (Spectrum
    Searcher, Peak Finder).
  repo: https://github.com/coongroup/LipiDex-2
- name: Compound Discoverer
  role: Peak extraction step applied to the retrieved .raw files; version 3.3 output
    (.cdResult) is required by LipiDex 2 Peak Finder.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:6d4077ceef8800a1ae43b06f3236723c833eb1b1821c40fe33245f0921f631f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - Compound Discoverer
  - LipiDex 2
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# massive-dataset-retrieval

## Summary

Retrieve a publicly deposited LC-MS lipidomics dataset from MassIVE by accession (e.g., MSV000091033) so it can be reprocessed with the LipiDex 2 pipeline. This skill anchors reproducibility checks in shared public data rather than locally held Thermo .raw files.

## When to use

Use when a published lipidomics workflow (e.g., LipiDex 2 Peak Finder benchmarks) cites a MassIVE accession such as MSV000091033 and the goal is to download the deposited instrument files for independent reprocessing and count verification.

## When NOT to use

- The target dataset is not publicly deposited under a MassIVE accession (e.g., raw files are only in a proprietary local archive or a different repository without an accession ID).
- The goal is to analyze an already-extracted peak or feature table rather than to reprocess original instrument files from a cited public deposit.

## Inputs

- MassIVE dataset accession (e.g., MSV000091033)
- Publicly deposited Thermo .raw LC-MS/MS instrument files (40 Nile rat plasma experimental samples in the reference case)

## Outputs

- Local copy of the deposited .raw files and metadata for the accession
- Verified sample inventory (40 experimental samples) ready for Compound Discoverer 3.3 peak extraction and LipiDex 2 reprocessing

## How to apply

Locate the MassIVE accession cited in the paper — for the LipiDex 2 study this is MSV000091033, a reprocessed public dataset containing 40 experimental samples generated from Nile rat plasma. Download the deposited Thermo .raw files (and any metadata) from the MassIVE repository for that accession. Confirm the downloaded sample count matches the paper's description before analysis, since downstream steps such as Compound Discoverer 3.3 peak extraction (m/z 100-5000, RT 0.4-21 min, 5e5 intensity threshold) assume the full 40-sample set. Feed the retrieved .raw files into the downstream LipiDex 2 modules (Spectrum Searcher, then Peak Finder, which reads Compound Discoverer 3.3 .cdResult peak tables directly) so that feature counts can be compared against published values (14826 unidentified / 779 identified pre-filter; 13898 / 719 post-Peak Finder filtering).

## Related tools

- **LipiDex 2** (Downstream pipeline that consumes the retrieved public dataset; the paper explicitly demonstrates reprocessing MSV000091033 with LipiDex 2 modules (Spectrum Searcher, Peak Finder).) — https://github.com/coongroup/LipiDex-2
- **Compound Discoverer** (Peak extraction step applied to the retrieved .raw files; version 3.3 output (.cdResult) is required by LipiDex 2 Peak Finder.)

## Evaluation signals

- Downloaded file inventory contains the 40 experimental Nile rat plasma samples described for MSV000091033.
- Retrieved files are Thermo .raw format suitable for direct Compound Discoverer 3.3 ingestion.
- Downstream reprocessing reproduces published counts: 14826 unidentified and 779 identified features before Peak Finder filtering; 13898 and 719 after.
- The accession ID resolves on MassIVE and the deposited data description matches the paper's dataset claim.

## Limitations

- MassIVE accessions must still be reprocessed with matching software versions and parameters (Compound Discoverer 3.3, LipiDex 2) for counts to reproduce; retrieval alone does not guarantee reproducibility.
- LipiDex 2 requires .NET 7.0 and Compound Discoverer 3.3 .cdResult files for Peak Finder, so the retrieved raw data cannot bypass those dependencies.

## Evidence

- [methods] Download the MSV000091033 Nile rat plasma dataset (40 experimental samples) from MassIVE: [evidence span withheld — anchor status: not_found]
