---
name: top-k-annotation-accuracy-evaluation
description: Use when you have predicted fingerprints for test spectra plus candidate
  InChIKey lists per compound and need to quantify annotation accuracy as top-1/top-k
  hit rate on structure-disjoint test sets.
when_to_use_negative:
- When the task is spectral-library matching against curated reference MS/MS spectra
  (the 'known unknowns' scenario the authors contrast with fingerprint prediction)
- When candidate lists are not structure-disjoint from the training compounds — accuracy
  will be inflated and not comparable to the reported ~50% top-1 rate
- When you need false discovery rate estimation for rankings — the article's discussion
  does not address specificity or FDR in candidate ranking
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: MetFID
  role: CNN model predicting the 528-bit fingerprint from each test MS/MS spectrum;
    similarity ranking then drives the annotation
  repo: https://github.com/aohongyu/MetFID
- name: PyFingerprint
  role: Computes OpenBabel (fp2/fp3/fp4-style) molecular fingerprints for candidate
    compounds to enable similarity scoring
  repo: https://github.com/hcji/PyFingerprint
- name: PubChemPy
  role: Queries the PubChem database by molecular formula to retrieve candidate compounds
    when MetaboQuest retrieval is not used
- name: tabulate
  role: Formats the per-block ranked InChIKey/score tables written to testing_compound_prediction.txt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:a72502b12c9a644fe8e4a67403ab5b0724b0091885a484e0207b354d17419d0f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MetaboQuest
  - MoNA
  - PubChemPy
  - PyFingerprint
  - TensorFlow
  - tabulate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# top-k-annotation-accuracy-evaluation

## Summary

Evaluates metabolite annotation performance by ranking candidate compounds by fingerprint-similarity score and computing the fraction of testing compounds whose true InChIKey appears in the top-k ranked list, benchmarked against MetFrag, ChemDistiller, and CSI:FingerID.

## When to use

Use when you have predicted fingerprints for test spectra plus candidate InChIKey lists per compound and need to quantify annotation accuracy as top-1/top-k hit rate on structure-disjoint test sets.

## When NOT to use

- When the task is spectral-library matching against curated reference MS/MS spectra (the 'known unknowns' scenario the authors contrast with fingerprint prediction)
- When candidate lists are not structure-disjoint from the training compounds — accuracy will be inflated and not comparable to the reported ~50% top-1 rate
- When you need false discovery rate estimation for rankings — the article's discussion does not address specificity or FDR in candidate ranking

## Inputs

- testing_compound.txt — per-compound blocks (#digit headers) with precursor mass, ionization mode, and m/z-intensity peak pairs
- inchikey_list.txt — per-compound blocks of candidate InChIKeys matching the testing blocks
- Pretrained CNN model file MetFID_CNN_40088_5618.h5
- True compound InChIKeys for structure-disjoint testing compounds (NIST, CASMI 2016, in-house cancer biomarker study)
- Candidate lists retrieved from MetaboQuest (10 PPM, H+ adduct) or by molecular formula from PubChem via PubChemPy

## Outputs

- testing_compound_prediction.txt — per-block tables of candidate InChIKeys with Tanimoto similarity scores, ranked in descending order
- Top-k annotation accuracy (e.g., top-1 hit rate) over structure-disjoint testing compounds
- Comparative accuracy metrics against MetFrag, ChemDistiller, and CSI:FingerID baselines

## How to apply

For each testing spectrum (e.g., blocks in testing_compound.txt with precursor mass, ionization mode, and intensity pairs), predict the 528-bit fingerprint (MACCS + FP3 + FP4) with the pretrained CNN (MetFID_CNN_40088_5618.h5) and compute the candidate fingerprint for every compound in the corresponding inchikey_list.txt block using OpenBabel via PyFingerprint. Score each candidate with delta(f',f) = 1 - sum|f'-f|/n, sort candidates in descending score order, and treat the top-ranked candidate as the annotation. Then search for the true compound's InChIKey in the top k of each ranked list to compute top-k annotation accuracy, using candidates retrieved from MetaboQuest (10 PPM, H+ adduct) or by formula via PubChemPy. Compare the resulting top-1 rate (~50% with MetaboQuest candidates) and the >5% improvement over MetFrag, ChemDistiller, and CSI:FingerID on the structure-disjoint NIST testing set. Validation output is testing_compound_prediction.txt, containing per-block tables of InChIKeys with scores ranked descending.

## Related tools

- **MetFID** (CNN model predicting the 528-bit fingerprint from each test MS/MS spectrum; similarity ranking then drives the annotation) — https://github.com/aohongyu/MetFID
- **PyFingerprint** (Computes OpenBabel (fp2/fp3/fp4-style) molecular fingerprints for candidate compounds to enable similarity scoring) — https://github.com/hcji/PyFingerprint
- **PubChemPy** (Queries the PubChem database by molecular formula to retrieve candidate compounds when MetaboQuest retrieval is not used)
- **tabulate** (Formats the per-block ranked InChIKey/score tables written to testing_compound_prediction.txt)

## Examples

```
cd MetFID && python3 main.py  # reads _files/testing_compound.txt and inchikey_list.txt, writes _files/testing_compound_prediction.txt
```

## Evaluation signals

- testing_compound_prediction.txt exists and each block's table is ranked in descending score order
- Recomputed top-1 annotation accuracy with MetaboQuest candidates approximates the reported ~50%
- Accuracy improvement over MetFrag, ChemDistiller, and CSI:FingerID exceeds 5% on the structure-disjoint NIST testing set
- Number of #digit blocks in testing_compound.txt matches the number of blocks in inchikey_list.txt (mismatch triggers an error per the README)
- Every candidate InChIKey in a ranked table belongs to the corresponding #digit candidate block

## Limitations

- Reported accuracy depends on structure-disjoint train/test splits; performance on compounds similar to training data is not comparable
- The discussion notes the target-decoy approach (Gupta et al. 2011) as future work — no false discovery rate estimation is provided for rankings
- The pretrained model MetFID_CNN_40088_5618.h5 is distributed externally (Google Drive link in README), not in the repository itself

## Evidence
