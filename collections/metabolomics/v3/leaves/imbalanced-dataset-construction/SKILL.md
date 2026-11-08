---
name: imbalanced-dataset-construction
description: Use when you need user-defined 'relevant'/'other' training classes for
  MS2DeepScore-based spectrum classifiers, by labeling spectral-library entries via
  SMARTS substructure queries.
when_to_use_negative:
- You already have curated labeled training spectra (e.g., reference standards); the
  filtering GUI is only needed to derive labels from structures by SMARTS matching.
- Your downstream analysis requires balanced training data — the article states the
  package 'is not capable of resolving' the imbalance failure mode.
- Your target structures cannot be captured by SMARTS substructure queries (labeling
  is structure-based, not spectrum-similarity based).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2258
tools:
- name: AnnoMe
  role: Provides the annome_filtergui GUI for SMARTS-based substructure filtering
    of MGF spectral libraries into 'relevant'/'other' classes
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Python package manager used to launch the AnnoMe download and filtering entry
    points (`uv run annome_downloadresources`, `uv run annome_filtergui`)
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:6d1b64ae825e81fcf0b11cecee2586c9f85a4282f50676cf2efaa1da37f3c5f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# imbalanced-dataset-construction

## Summary

Constructs a deliberately imbalanced binary training set for HR-MS/MS spectrum classification by SMARTS substructure-filtering a spectral library into a small 'relevant' class and a much larger 'other' class (e.g., 655 (iso-)flavonoid vs. 10000 non-(iso-)flavonoid spectra from MSnLib v6).

## When to use

Use when you need user-defined 'relevant'/'other' training classes for MS2DeepScore-based spectrum classifiers, by labeling spectral-library entries via SMARTS substructure queries.

## When NOT to use

- You already have curated labeled training spectra (e.g., reference standards); the filtering GUI is only needed to derive labels from structures by SMARTS matching.
- Your downstream analysis requires balanced training data — the article states the package 'is not capable of resolving' the imbalance failure mode.
- Your target structures cannot be captured by SMARTS substructure queries (labeling is structure-based, not spectrum-similarity based).

## Inputs

- MSnLib v6 spectral library in MGF format
- (iso-)flavonoid SMARTS substructure query strings
- prenylated-flavonoid and prenylated-chalcone SMARTS substructure definitions
- public MS/MS resources (~11 GB) downloaded via annome_downloadresources

## Outputs

- Filtered 'relevant' class MGF file (655 (iso-)flavonoid spectra for MSnLib v6)
- Filtered 'other' class MGF file (10000 non-(iso-)flavonoid spectra)
- Summary table of 'relevant'/'other' class counts per substructure definition

## How to apply

Download the public MS/MS resources (~11 GB, resumable) with `uv run annome_downloadresources`, then launch the filtering GUI via `uv run annome_filtergui` and load the MSnLib v6 spectral library in MGF format as the input dataset. Apply the (iso-)flavonoid SMARTS substructure queries so that structures matching the SMARTS are labeled 'relevant' and all remaining structures are labeled 'other'; repeat with the prenylated-flavonoid and prenylated-chalcone SMARTS definitions for the Section 3.2 split. Export the filtered spectra as MGF files and compile the class counts into a summary table. Be aware that the resulting 655-relevant vs. 10000-other split is highly imbalanced (≈1:15), and the AnnoMe authors explicitly warn that classifiers most often fail on such imbalanced training data, so treat downstream class assignments with caution.

## Related tools

- **AnnoMe** (Provides the annome_filtergui GUI for SMARTS-based substructure filtering of MGF spectral libraries into 'relevant'/'other' classes) — https://github.com/chrboku/AnnoMe
- **uv** (Python package manager used to launch the AnnoMe download and filtering entry points (`uv run annome_downloadresources`, `uv run annome_filtergui`)) — https://github.com/astral-sh/uv

## Examples

```
uv run annome_filtergui
```

## Evaluation signals

- The (iso-)flavonoid filter yields exactly 655 'relevant' spectra and 10000 'other' spectra on MSnLib v6, matching Section 3.1.
- Every spectrum in the 'relevant' MGF contains a structure whose SMARTS match the (iso-)flavonoid substructure definition; 'other' entries match none.
- The prenylated-flavonoid and prenylated-chalcone filters reproduce the Section 3.2 'relevant'/'other' split.
- Exported files are valid MGF readable by AnnoMe's downstream classification GUI, with total spectra preserved across the two classes.

## Limitations

- Classification on such a highly imbalanced training set (655 vs. 10000) will most often not be successful; the package cannot resolve this, so users must exercise caution with class assignments.
- Resources total ~11 GB and the download can be slow; it can be interrupted and restarted, but beware on metered connections.
- The limiting downstream step is MS2DeepScore embedding generation (4 hours on a standard laptop; failed on a passively cooled MacBook Air M4 with 16 GB RAM), though embeddings are cached on disk after the first run.

## Evidence

- [results] SMARTS filtering class counts: [evidence span withheld — anchor status: not_found]
- [readme] filter GUI invocation: "Start the filtering GUI via the command `uv run annome_filtergui`"
- [methods] resource download: "Download available public MS/MS resources using the command `uv run annome_downloadresources`"
- [discussion] imbalance caveat: "especially when the training dataset is highly imbalanced. The presented python package is not capable of resolving this issue."
