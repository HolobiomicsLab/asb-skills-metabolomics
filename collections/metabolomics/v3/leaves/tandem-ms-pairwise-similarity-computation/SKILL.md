---
name: tandem-ms-pairwise-similarity-computation
description: Use when you hold a cleaned set of annotated tandem MS spectra (e.g.
  MGF from GNPS) and need an all-pairs spectral similarity or score matrix to retrieve
  chemically related compound pairs or cluster spectra.
when_to_use_negative:
- You need scores with an intuitive chemical explanation of why a pair matched — MS2DeepScore
  lacks explainability of individual scores
- Spectra are unannotated and uncleaned with wildly mixed instrument types and quality
  — library bias toward certain compound classes limits transfer to experimental data
- You need exact compound identification rather than structural-similarity retrieval;
  high scores cannot discriminate Tanimoto 0.8–0.9 from a near-complete chemical match
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS2DeepScore
  role: Siamese neural network computing spectrum-pair similarity via 200-dim embeddings
    and cosine similarity
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library for spectra import, filtering (DEFAULT_FILTERS), and score
    computation pipeline
- name: Spec2Vec
  role: Unsupervised spectral similarity baseline for comparison against MS2DeepScore
- name: RDKit
  role: Computes Morgan fingerprint Tanimoto similarities as structural ground truth
    for evaluating spectral scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:d96f81001fc4179481d37bba74a6dd4a9230d636affa2da3b316c811d60289ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - RDKit
  - Spec2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tandem-ms-pairwise-similarity-computation

## Summary

Compute pairwise spectral similarity scores between MS/MS spectra using a Siamese-network embedding model (MS2DeepScore), classical modified Cosine, or Spec2Vec, producing all-pairs score matrices suitable for retrieval and clustering analyses.

## When to use

Use when you hold a cleaned set of annotated tandem MS spectra (e.g. MGF from GNPS) and need an all-pairs spectral similarity or score matrix to retrieve chemically related compound pairs or cluster spectra.

## When NOT to use

- You need scores with an intuitive chemical explanation of why a pair matched — MS2DeepScore lacks explainability of individual scores
- Spectra are unannotated and uncleaned with wildly mixed instrument types and quality — library bias toward certain compound classes limits transfer to experimental data
- You need exact compound identification rather than structural-similarity retrieval; high scores cannot discriminate Tanimoto 0.8–0.9 from a near-complete chemical match

## Inputs

- Cleaned MS/MS spectra file (MGF, MSP, MZML, MZXML, JSON, USI) with precursor m/z, ionmode, and SMILES/InChI annotations where available
- Pretrained MS2DeepScore model file (ms2deepscore_model.pt)
- Optional: molecular fingerprints (Morgan-2/Morgan-3 via RDKit) for structural ground-truth Tanimoto scores

## Outputs

- All-pairs spectral similarity matrix (numpy array of MS2DeepScore predictions between 0 and 1)
- 200-dimensional spectrum embedding array for downstream clustering/UMAP
- Optionally, threshold-swept precision/recall table for comparing similarity measures (precision_recall_curves.csv)

## How to apply

Load the spectra (MGF/MSP/MZML etc.) through the matchms Pipeline with DEFAULT_FILTERS, then instantiate MS2DeepScore with a pretrained model ('ms2deepscore_model.pt') and register it as the score computation to obtain an all-pairs similarity matrix. MS2DeepScore embeds each spectrum into a 200-dimensional vector (binned over 10,000 equal-width bins from 10 to 1000 m/z) and scores pairs by cosine similarity of embeddings, so embeddings can be cached and reused. Classical alternatives are the two modified Cosine variants from matchms and the unsupervised Spec2Vec metric. For model-free comparisons that depend on peak matching, ensure precursor m/z and ionmode metadata survive filtering, whereas MS2DeepScore relies only on peak m/z positions and intensities. Scores range from 0 to 1 and can be thresholded (e.g. swept 0–1) against structural ground truth (RDKit Tanimoto on Morgan fingerprints) for precision/recall evaluation.

## Related tools

- **MS2DeepScore** (Siamese neural network computing spectrum-pair similarity via 200-dim embeddings and cosine similarity) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library for spectra import, filtering (DEFAULT_FILTERS), and score computation pipeline)
- **Spec2Vec** (Unsupervised spectral similarity baseline for comparison against MS2DeepScore)
- **RDKit** (Computes Morgan fingerprint Tanimoto similarities as structural ground truth for evaluating spectral scores)

## Examples

```
from ms2deepscore import MS2DeepScore; from ms2deepscore.models import load_model; scores = MS2DeepScore(load_model("ms2deepscore_model.pt")).matrix(spectra, spectra)
```

## Evaluation signals

- The similarity matrix is symmetric with unit diagonal (each spectrum paired with itself)
- Scores lie between 0 and 1; predicted Tanimoto RMSE against RDKit fingerprints should be around 0.15 on ~3600 spectra of 500 unseen compounds
- In a threshold sweep (0–1) against Tanimoto > 0.6 positives, MS2DeepScore precision-recall curve should dominate modified Cosine and Spec2Vec curves
- Median aggregation over multiple spectrum pairs of the same compound pair should improve prediction reliability versus individual predictions
- Embeddings should separate chemical classes in t-SNE/UMAP colored by ClassyFire superclasses

## Limitations

- Scores are not deterministic and may change when a new model version is trained with different parameters or training data
- Deep-learning scores lack an intuitive explanation for why a particular score was assigned
- It is difficult to discriminate high Tanimoto scores (0.8–0.9) from a near-complete chemical match; identification requires additional algorithms
- Training spectral libraries are biased toward certain compound classes and mixed instrumentation quality, limiting application to experimental data

## Evidence

- [abstract] MS2DeepScore: a novel Siamese neural network to predict the structural similarity between two chemical structures solely based on their MS/MS fragmentation spectra.: "Here, we propose MS2DeepScore: a novel Siamese neural network to predict the structural similarity between two chemical structures solely based on their MS/MS fragmentation spectra."
- [intro] Only peak m/z and intensities are required as inputs.: "our approach only relies on peak m/z positions and intensities without requiring further spectrum information"
- [results] MS2DeepScore is integrated in matchms for spectra handling and comparisons.: "MS2DeepScore is integrated in matchms, a recently developed Python library for mass spectra import, handling and comparisons"
- [readme] README example of running the pipeline to compute an all-pairs similarity matrix.: "pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS,
                                    score_computations=[[MS2DeepScore, {"model": model}]]))"
- [discussion] Spectrum binning specification from the discussion/methods.: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
