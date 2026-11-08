---
name: molecular-network-construction
description: Use when you have a large extract library (e.g., thousands of MGF MS/MS
  spectra) and want to cluster spectra into molecular families using a learned spectral
  similarity (TransExION) instead of GNPS cosine networking.
when_to_use_negative:
- Input is raw LC-MS/MS vendor files or a feature table rather than MGF/h5py MS/MS
  spectra — peak picking and MGF export must happen first.
- The goal is de novo structural elucidation of a single unknown spectrum rather than
  network-level compound-family organization.
- A strict peak-matched cosine network (GNPS classical networking) is required for
  comparability with legacy workflows; TransExION edges do not require a minimum matched
  peak count.
edam_operation: http://edamontology.org/operation_3645
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TransExION
  role: Computes all pairwise MS/MS spectral similarities used as network edge weights;
    provides the pre-trained model and ms_test_spectral_similarity.py scoring script.
  repo: github.com/banhdzui/TransExION
- name: meRgeION
  role: Imports the TransExION similarity matrix and builds the molecular network
    (topK, maximum molecular family size) with analogue annotation of nodes.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:3c0280aa201edc334d4047755d433b20683acbb36b238f1b3ac77f607720227d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TransExION
  - meRgeION
  edam_operation: http://edamontology.org/operation_3645
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# molecular-network-construction

## Summary

Constructs a molecular network of MS/MS spectra by computing all pairwise spectral similarities with TransExION, filtering edges (score > 0.6, no minimum matched-peak requirement), and importing the similarity matrix into meRgeION to recover compound-family sub-networks such as those originally derived with SNAP-MS/GNPS.

## When to use

Use when you have a large extract library (e.g., thousands of MGF MS/MS spectra) and want to cluster spectra into molecular families using a learned spectral similarity (TransExION) instead of GNPS cosine networking.

## When NOT to use

- Input is raw LC-MS/MS vendor files or a feature table rather than MGF/h5py MS/MS spectra — peak picking and MGF export must happen first.
- The goal is de novo structural elucidation of a single unknown spectrum rather than network-level compound-family organization.
- A strict peak-matched cosine network (GNPS classical networking) is required for comparability with legacy workflows; TransExION edges do not require a minimum matched peak count.

## Inputs

- MGF file of experimental MS/MS spectra from an extract library (converted to h5py format)
- Pre-trained TransExION model file (from Zenodo record 8175528)
- mergedGNPS spectral library for analogue-based node annotation

## Outputs

- Filtered pairwise spectral similarity matrix (edges > 0.6)
- Molecular network with sub-networks representing compound families (meRgeION)
- Per-node analogue annotations (top 3 analogues, score > 0.6)

## How to apply

Load the experimental spectra (e.g., 2161 spectra from a 925-sample marine bacteria extract library) and the pre-trained TransExION model from Zenodo (record 8175528). Convert MGF files to h5py format with ms_convert_mgf2h5py.py, then compute all pairwise TransExION similarities using ms_test_spectral_similarity.py with every query spectrum paired against every reference spectrum (leave --pairs empty). Filter the similarity matrix to keep only edges above 0.6, deliberately without requiring a minimum number of matched peaks, since TransExION similarity reflects structural similarity rather than peak alignment. Import the filtered matrix into meRgeION and build the network with the original manuscript's parameters (topK, maximum molecular family size). Finally, annotate each node (spectrum) by analogue search against the mergedGNPS spectral library, retaining the top three analogues per node with similarity above 0.6.

## Related tools

- **TransExION** (Computes all pairwise MS/MS spectral similarities used as network edge weights; provides the pre-trained model and ms_test_spectral_similarity.py scoring script.) — github.com/banhdzui/TransExION
- **meRgeION** (Imports the TransExION similarity matrix and builds the molecular network (topK, maximum molecular family size) with analogue annotation of nodes.)

## Examples

```
python ms_convert_mgf2h5py.py --mgf bacteria_extract_library.mgf --out library.db && python ms_test_spectral_similarity.py --db library.db --query library.db --model GNPS_MassBank.ms.model --output pairwise_similarity.csv
```

## Evaluation signals

- The TransExION-meRgeION network recovers all seven sub-networks confidently assigned with compound families by SNAP-MS in the original bacteria-extract analysis.
- Recovered sub-network topology matches the compound-family structure shown in the original molecular network figure.
- Every edge in the imported similarity matrix exceeds the 0.6 score cutoff; edges at or below 0.6 are absent.
- Each annotated node has at most three analogue matches from mergedGNPS, all with similarity above 0.6.
- Pairwise similarity computation covers all query-reference spectrum pairs (i.e., --pairs left empty yields N x N scores).

## Limitations

- TransExION similarity is a learned proxy for structural (Tanimoto) similarity; it was shown to outperform Cosine, Modified Cosine, Spec2Vec, and MS2DeepScore, but quantitative performance metrics per pair are not reported in the discussion.
- Network reproduction depends on matching the original manuscript's meRgeION parameters (topK, maximum molecular family size); deviating from them changes sub-network recovery.
- No minimum matched-peak requirement is applied to TransExION edges, so low-quality spectra with few fragments may still form network edges.

## Evidence

- [discussion] Using the pair-wise spectral similarities computed by TransExION, we built a molecular network of bacteria extracts and retrieved sub-networks representing confidently-identified chemical families: "Using the pair-wise spectral similarities computed by TransExION, we built a molecular network of bacteria extracts and retrieved sub-networks representing confidently-identified chemical families[36]"
- [readme] To work with large scale dataset, the input data files are converted into h5py format.: "To work with large scale dataset, the input data files are converted into h5py format. Our library supports a function to convert a MGF format file into h5py format."
- [readme] If the parameter --pairs is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity.: "If the parameter ```--pairs``` is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity."
- [other] Filter pairwise similarity scores to keep only edges above 0.6, without requiring a minimum number of matched peaks.: [evidence span withheld — anchor status: not_found]
- [other] Annotate each experimental spectrum (node) by analogue search against the mergedGNPS spectral library, retaining the top three analogues per node with similarity score above 0.6.: [evidence span withheld — anchor status: not_found]
