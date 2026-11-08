---
name: blank-derived-word-filtering
description: Use when MS2-derived MEMO fingerprints contain word tokens contributed
  by blank/injection-control samples, and you need to remove those blank-derived peaks
  and precursor neutral losses before clustering or visualizing samples.
when_to_use_negative:
- No blank or injection-control samples are present in the dataset, so there is no
  blank reference to justify removing word tokens
- Input is raw MS2 spectra that have not yet been converted into MEMO word documents
  — vectorization (import_data with the spec2vec-style parameters) must happen before
  blank-word filtering
- You need RT-based feature alignment outputs rather than retention-time-agnostic
  fingerprints (MEMO deliberately bypasses prior feature alignment)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: memo-ms
  role: Python package used to import .mgf MS2 spectra as word-token documents and
    build/filter the MemoMatrix by removing blank-derived peaks/losses
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying package MEMO is built on for handling and cleaning the MS2 spectra
    before word binning
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying package providing the document/word encoding of peaks and losses
    that defines the word tokens being filtered
  repo: https://github.com/iomega/spec2vec
- name: Python 3.8
  role: Runtime for the memo conda environment in which the filtering workflow is
    executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:94de460e74d72b49ba8e0648dca54e11a8ddc4c36caf3ba07b5bd5b106cfa48a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - Python
  - Python 3.8
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# blank-derived-word-filtering

## Summary

Filters MEMO MS2 fingerprint matrices by removing word tokens (peak@mz, loss@mz) that originate from blank samples, so that contamination/carryover peaks do not drive sample clustering. This is the canonical post-vectorization filtering step of the MEMO retention-time-agnostic sample comparison method.

## When to use

Use when MS2-derived MEMO fingerprints contain word tokens contributed by blank/injection-control samples, and you need to remove those blank-derived peaks and precursor neutral losses before clustering or visualizing samples.

## When NOT to use

- No blank or injection-control samples are present in the dataset, so there is no blank reference to justify removing word tokens
- Input is raw MS2 spectra that have not yet been converted into MEMO word documents — vectorization (import_data with the spec2vec-style parameters) must happen before blank-word filtering
- You need RT-based feature alignment outputs rather than retention-time-agnostic fingerprints (MEMO deliberately bypasses prior feature alignment)

## Inputs

- .mgf MS2 spectra files from an FBMN job (e.g. MassIVE MSV000083306) including blank/control sample spectra
- Per-sample MS2 fingerprint vectors / MemoMatrix with peak@mz and loss@mz word-token columns
- Sample metadata from .mgf filenames distinguishing blanks from biological samples

## Outputs

- Filtered MemoMatrix (sample-by-word matrix) with blank-derived peak/loss word columns removed
- memo_matrix.csv with rows corresponding exactly to input (non-blank-filtered) samples and remaining word-token columns

## How to apply

First build the MEMO fingerprints: import .mgf MS2 spectra with memo_ms (spec2vec-style documents) using min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, and n_decimals=2, so peaks and precursor neutral losses become rounded word tokens grouped into per-sample documents. Identify word tokens that occur in blank samples (the MEMO matrix rows tagged as blanks in the .mgf filename metadata), then remove those columns from the MemoMatrix — i.e. remove peaks/losses from blanks — while keeping the row-per-sample structure intact. The rationale is that blank-derived words represent contamination or solvent/background signals that would inflate similarity between unrelated samples; MEMO's design explicitly exposes this filtering stage between fingerprint generation and downstream visualization (MDS/PCoA, TMAP, Heatmap). Implementation care is needed: the memo codebase history notes filtering was changed to avoid unintentionally erasing data (classes.py), so verify the filtered matrix retains all non-blank sample rows. Write the filtered matrix (e.g. memo_matrix.csv) only after confirming blank-only words are absent.

## Related tools

- **memo-ms** (Python package used to import .mgf MS2 spectra as word-token documents and build/filter the MemoMatrix by removing blank-derived peaks/losses) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying package MEMO is built on for handling and cleaning the MS2 spectra before word binning) — https://github.com/matchms/matchms
- **spec2vec** (Underlying package providing the document/word encoding of peaks and losses that defines the word tokens being filtered) — https://github.com/iomega/spec2vec
- **Python 3.8** (Runtime for the memo conda environment in which the filtering workflow is executed)

## Examples

```
from memo_ms.import_data import SpectraDocuments; docs = SpectraDocuments('MSV000083306_spectra.mgf', min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, n_decimals=2)
```

## Evaluation signals

- Filtered memo_matrix.csv parses as a matrix whose rows correspond exactly to the input samples (.mgf filenames) — no sample rows lost during filtering
- No word tokens identified as blank-derived peaks/losses remain as columns in the filtered matrix
- Columns of the filtered matrix remain interpretable peak@mz / loss@mz word tokens with n_decimals=2 rounding, consistent with the paper's counted MS2 peaks and neutral losses
- Matrix remains well-formed for downstream similarity computation (fingerprints still alignable across samples without RT-based feature alignment)
- Filtering did not silently erase non-blank data — a regression noted in the memo changelog ('Changed classes.py to avoid erasing when filtering')

## Limitations

- Filtering is optional and example-level in the paper: 'remove peaks/losses from blanks' is given as an example, with no fixed blank/sample intensity-ratio threshold specified
- A memo code change was needed so that filtering avoids unintentionally erasing data (classes.py), implying earlier versions could lose fingerprints when filtering
- Removal of blank words is only as good as blank coverage — words occurring in both blanks and real samples are handled by column removal but genuine low-level sample signal shared with blanks will also be discarded

## Evidence

- [intro] Once fingerprints are obtained, filtering (e.g. removing peaks/losses from blanks) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used.: "different filtering (remove peaks/losses from blanks for example)"
- [methods] Blank-word removal happens on the aligned fingerprint matrix after counting peaks and losses.: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample"
- [readme] The MEMO repository README repeats the blank-filtering step as part of the standard workflow.: "Once obtained, different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used."
- [discussion] The memo changelog documents a filtering bug/regression fix relevant to correct application of this skill.: "Changed classes.py to avoid erasing when filtering"
