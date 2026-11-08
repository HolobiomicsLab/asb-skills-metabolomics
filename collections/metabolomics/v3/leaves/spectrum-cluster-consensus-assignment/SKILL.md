---
name: spectrum-cluster-consensus-assignment
description: Use when a large collection of unidentified MS/MS spectra has been clustered
  (e.g. 3.4M GLEAMS clusters) and the goal is cluster-level peptide assignments plus
  enrichment statistics for variant-consistent mismatches.
when_to_use_negative:
- Input spectra already have confident database-search PSMs — use Casanovo-DB mode
  (`casanovo db_search`) instead of de novo clustering consensus
- Data were acquired in DIA mode — the pipeline targets data-dependent acquisition,
  bottom-up tandem MS datasets
- A per-spectrum confidence-calibrated identification list is required — the article
  notes 'the absence of a rigorous method for confidence estimation for de novo sequencing'
  is a key bottleneck, so cluster consensus is only a proxy
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Transformer de novo sequencer run in de novo mode with beam search to predict
    per-spectrum peptides, exported as mzTab; also supports Casanovo-DB mode (`casanovo
    db_search`) as a learned score function
  repo: https://github.com/Noble-Lab/casanovo
- name: GLEAMS
  role: Provides the cluster identifiers over which per-spectrum predictions are grouped
    and plurality-voted into consensus peptides
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:996320e83d5606288edc314790d7e51cc8e2a63a87de36e28b01b076050859b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - GLEAMS
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spectrum-cluster-consensus-assignment

## Summary

Assign consensus peptide sequences to large sets of unidentified 'dark matter' MS/MS spectra by running de novo sequencing (Casanovo with beam search) over GLEAMS-clustered spectra, voting a per-cluster consensus, and retaining clusters whose consensus maps to predicted proteomes.

## When to use

Use when a large collection of unidentified MS/MS spectra has been clustered (e.g. 3.4M GLEAMS clusters) and the goal is cluster-level peptide assignments plus enrichment statistics for variant-consistent mismatches.

## When NOT to use

- Input spectra already have confident database-search PSMs — use Casanovo-DB mode (`casanovo db_search`) instead of de novo clustering consensus
- Data were acquired in DIA mode — the pipeline targets data-dependent acquisition, bottom-up tandem MS datasets
- A per-spectrum confidence-calibrated identification list is required — the article notes 'the absence of a rigorous method for confidence estimation for de novo sequencing' is a key bottleneck, so cluster consensus is only a proxy

## Inputs

- GLEAMS-clustered unidentified MS/MS spectra (~3.4 million clustered spectra) as annotated MGF files
- Pretrained Casanovo model weights (MassIVE-KB trained, e.g. Zenodo 10.5281/zenodo.12587317)
- Predicted protein sequences / predicted proteomes for proteome-first mapping

## Outputs

- Per-spectrum peptide predictions and confidence scores in mzTab format
- Cluster-level consensus peptide assignments keyed by GLEAMS cluster identifier
- gleams_cluster_assignments_summary.tsv with cluster-assignment counts, assigned-spectra totals, single-nucleotide-substitution fraction, and BLOSUM62 enrichment statistics

## How to apply

Download the GLEAMS clustered unidentified MS/MS spectra and a compatible pretrained Casanovo model (trained on 30 million MassIVE-KB PSMs). Run Casanovo in de novo mode with beam search decoding (not greedy) over all clustered spectra, exporting per-spectrum predictions and confidence scores to mzTab. Group per-spectrum predictions by GLEAMS cluster identifier and take a plurality-vote consensus peptide, yielding cluster-level assignments (e.g. 89,250 of ~560k clusters, 16%). Apply the proteome-first strategy, retaining only clusters whose consensus peptides map to predicted proteomes (reproducing 52,523 clusters, 9%, and 196,724 assigned spectra). For clusters with discordant per-spectrum assignments, test whether amino-acid mismatches are consistent with single-nucleotide substitutions and compute BLOSUM62 enrichment of mismatched assignments.

## Related tools

- **Casanovo** (Transformer de novo sequencer run in de novo mode with beam search to predict per-spectrum peptides, exported as mzTab; also supports Casanovo-DB mode (`casanovo db_search`) as a learned score function) — https://github.com/Noble-Lab/casanovo
- **GLEAMS** (Provides the cluster identifiers over which per-spectrum predictions are grouped and plurality-voted into consensus peptides)

## Examples

```
casanovo sequence --config casanovo.yaml --output dark_proteome_predictions.mztab gleams_clusters.mgf
```

## Evaluation signals

- Cluster-level assignment rate falls in the expected band: ~89,250 clusters (16%) assigned by consensus, ~52,523 clusters (9%) after proteome-first mapping
- Total assigned spectra ≈ 196,724 of the 3.4 million clustered spectra
- Fraction of predicted substitutions consistent with a single-nucleotide substitution is enriched (~83.4%) relative to the expected background (38.6%)
- BLOSUM62 enrichment statistic for mismatched assignments is positive/reported in the summary TSV
- Output mzTab parses and contains one prediction entry per input clustered spectrum with confidence scores

## Limitations

- Lack of a rigorous confidence-estimation method for de novo sequencing means cluster consensus assignments carry no calibrated FDR
- The precursor mass filter is no longer applied in de novo mode; only min_peaks/max_peaks and minimum peptide length filters constrain predictions
- Proteome-first mapping discards clusters whose consensus peptides do not map to predicted proteomes, limiting foreign-peptide discovery (target-decoy matching 'misses out on the power of de novo sequencing to assign peptides to foreign spectra')

## Evidence

- [intro] Beam search decoding over clustered spectra: "We expand our training set to use 30 million confident PSMs from the MassIVE-KB spectral library23, and we add a beam search decoding procedure to predict the best peptide for each MS/MS spectrum."
- [methods] mzTab export of predictions: "Export spectrum predictions to mzTab files."
- [methods] Casanovo-DB mode as learned score function: "Casanovo-DB mode (`casanovo db_search`) to use Casanovo as a learned score function for sequence database searching"
- [abstract] Dark proteome motivation: "Casanovo’s superior performance improves the analysis of immunopeptidomics and metaproteomics experiments and allows us to delve deeper into the dark proteome"
- [methods] Beam search computational optimization: "Beam search decoding has been optimized for computational efficiency"
- [readme] Casanovo de novo purpose: "Casanovo is a state-of-the-art deep learning tool designed for _de novo_ peptide sequencing."
