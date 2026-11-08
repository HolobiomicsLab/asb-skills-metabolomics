---
name: proteome-first-peptide-annotation
description: Use when you have large collections of unidentified MS/MS spectra (e.g.
  GLEAMS clusters) and want proteome-anchored de novo peptide assignments plus variant-enrichment
  evidence for single-mismatch predictions.
when_to_use_negative:
- Spectra already confidently assigned by database search — sequence database searching
  is the standard route when the set of proteins in the sample is known in advance
- Single isolated spectra without cluster support — the consensus/plurality vote and
  proteome-first filter depend on GLEAMS cluster structure
- Datasets requiring PTM- or contaminant-driven discovery outside any reference proteome
  — the proteome-first retention step discards clusters whose consensus peptides do
  not map to predicted proteomes
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Casanovo
  role: De novo transformer that translates MS/MS peaks into amino-acid sequences
    using beam search decoding and exports predictions to mzTab
  repo: https://github.com/Noble-Lab/casanovo
- name: PDV
  role: Graphical inspection of Casanovo per-spectrum peptide predictions during QA
  repo: https://github.com/wenbostar/PDV/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:c042c225cc225e95141c0496bac14d1a632947458abe18b6b43f24347c2e8be9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - PDV
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# proteome-first-peptide-annotation

## Summary

Assign previously unidentified 'dark proteome' MS/MS spectra by running Casanovo de novo sequencing on GLEAMS-clustered spectra, deriving cluster-level consensus peptides, and retaining only clusters whose consensus peptides map to predicted proteomes.

## When to use

Use when you have large collections of unidentified MS/MS spectra (e.g. GLEAMS clusters) and want proteome-anchored de novo peptide assignments plus variant-enrichment evidence for single-mismatch predictions.

## When NOT to use

- Spectra already confidently assigned by database search — sequence database searching is the standard route when the set of proteins in the sample is known in advance
- Single isolated spectra without cluster support — the consensus/plurality vote and proteome-first filter depend on GLEAMS cluster structure
- Datasets requiring PTM- or contaminant-driven discovery outside any reference proteome — the proteome-first retention step discards clusters whose consensus peptides do not map to predicted proteomes

## Inputs

- GLEAMS-clustered unidentified MS/MS spectra (~3.4 million clustered spectra from MassIVE)
- Pretrained Casanovo model weights (transformer trained on 30 million MassIVE-KB PSMs)
- Predicted/target proteome sequences for the proteome-first mapping step

## Outputs

- Per-spectrum peptide predictions with confidence scores in mzTab format
- Cluster-level consensus peptide assignments keyed by GLEAMS cluster ID
- Cluster-assignment counts and assigned-spectra totals (e.g. gleams_cluster_assignments_summary.tsv)
- Single-nucleotide-substitution consistency rates and BLOSUM62 enrichment statistics for mismatched assignments

## How to apply

Download the GLEAMS clustered unidentified MS/MS spectra (~3.4 million clustered spectra) and a compatible pretrained Casanovo transformer checkpoint trained on 30 million MassIVE-KB PSMs. Run Casanovo in de novo mode with beam search decoding (not greedy decoding), exporting per-spectrum peptide predictions and confidence scores to mzTab. Group per-spectrum predictions by GLEAMS cluster identifier and derive a cluster-level consensus peptide by plurality vote — in the source study this yielded 89,250 assigned clusters (~16%). Apply the proteome-first filter: keep only clusters whose consensus peptides map to predicted proteomes, which gave 52,523 clusters (~9%) covering 196,724 assigned spectra. For clusters with discordant per-spectrum assignments, test whether amino-acid mismatches are consistent with a single-nucleotide substitution and compute BLOSUM62 enrichment of mismatched assignments (observed 83.4% potential single-nucleotide substitutions vs 38.6% expected). Write cluster-assignment counts, assigned-spectra totals, and enrichment statistics to a summary TSV (e.g. gleams_cluster_assignments_summary.tsv).

## Related tools

- **Casanovo** (De novo transformer that translates MS/MS peaks into amino-acid sequences using beam search decoding and exports predictions to mzTab) — https://github.com/Noble-Lab/casanovo
- **PDV** (Graphical inspection of Casanovo per-spectrum peptide predictions during QA) — https://github.com/wenbostar/PDV/

## Examples

```
casanovo denovo gleams_clustered.mgf -o casanovo_predictions.mztab --beam_size 5
```

## Evaluation signals

- Assigned-spectra total matches the expected order of magnitude (~196,724 spectra out of 3.4M unidentified clustered spectra)
- Cluster-level plurality vote reproduces ~89,250 assigned clusters (~16% of GLEAMS clusters)
- Proteome-first retention yields ~52,523 assigned clusters (~9%), a strict subset of the consensus assignments
- mzTab output parses against the mzTab schema and contains per-spectrum scores and peptide sequences
- Mismatched assignments show enriched single-nucleotide-substitution consistency (83.4% observed vs 38.6% expected) with positive BLOSUM62 enrichment

## Limitations

- No rigorous method for confidence estimation for de novo sequencing exists — a key bottleneck — so cluster consensus and proteome mapping act as surrogate filters
- A target-decoy matching approach misses out on the power of de novo sequencing to assign peptides to foreign spectra; the proteome-first strategy still excludes non-proteome clusters
- The precursor mass filter is not applied in de novo mode, so low-quality spectra must be removed via options such as min_peaks before sequencing

## Evidence

- [abstract] dark proteome via proteome-first mapping: "Casanovo’s superior performance improves the analysis of immunopeptidomics and metaproteomics experiments and allows us to delve deeper into the dark proteome"
- [intro] beam search decoding rationale: "We expand our training set to use 30 million confident PSMs from the MassIVE-KB spectral library23, and we add a beam search decoding procedure to predict the best peptide for each MS/MS spectrum."
- [methods] mzTab export of per-spectrum predictions: "Export spectrum predictions to mzTab files."
- [discussion] why de novo (not target-decoy) for foreign/dark spectra: "we have matched Casanovo predictions to a target and corresponding decoy peptide database, but such an approach misses out on the power of de novo sequencing to assign peptides to foreign spectra."
- [readme] scope of the tool for unexpected peptides: "Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset"
