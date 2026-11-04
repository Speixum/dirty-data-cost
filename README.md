# How Much Does Dirty Data Cost?

Code, dataset-version metadata and result files for the paper

> **How Much Does Dirty Data Cost? A Controlled Study of Dataset Defects on
> Surface-Defect Classification**

We do not propose a new model. We quantify how data defects propagate to downstream
performance, and we test whether a *training-free* quality score can predict that loss.

Starting from the public NEU-CLS benchmark (six classes of hot-rolled steel surface defects),
we build **26 dataset versions**: five defect families (Gaussian noise, salt-and-pepper noise,
JPEG compression, motion blur, systematic label confusion) at four severity levels each, plus
a cumulative series in which defect types are stacked one by one. One ResNet-18 is trained per
version under an identical budget, so differences in test accuracy are attributable to the
injected defect.

| Result | Value |
|---|---|
| Clean reference (NEU-CLS) | 99.63% test accuracy |
| Most expensive defect | Gaussian noise, sigma = 32: **-67.41 pp** |
| Cheapest defect | 15% systematic label confusion: **-1.11 pp** |
| Stacked defects | not additive: 46.85% -> 78.33% as more defects are added |
| Fidelity score vs accuracy | Spearman rho = **-0.011** (fails) |
| Reference-model agreement (RMA) vs accuracy | Spearman rho = **0.825** (p = 2.1e-7) |
| RMA used as a screening gate | precision **1.00**, recall **0.94**, F1 0.97 |
| Repair loop (worst version) | 32.22% -> **99.63%** by replacing the worst-scored 20% of files |
| Second dataset (KolektorSDD2) | the same corruptions cost at most **2.29 pp** |

## Repository layout

```
src/                     all experiment code (build versions, train, score, analyse, plot)
data/splits/             the fixed stratified 70/15/15 split (seed 42)
data/versions/Vxx/       per-version metadata: manifest.json, labels.csv, problems.md
results/versions/        one JSON per training run (accuracy, quality scores, timing)
results/ksdd2/           the second-dataset runs
paper/figs/              result figures only (the manuscript is not published here)
docs/                    datasets, reproduction, notes
```

**The dataset images are not in this repository.** They are third-party benchmarks with their
own terms. See [docs/DATASETS.md](docs/DATASETS.md) for the download links, the checksums we
observed, and where to put the files so that the code finds them. What *is* included is
everything we produced: the split, the per-version label files, the manifests, and all result
files, so every number in the paper can be re-derived.

## Quick start

```bash
pip install -r requirements.txt
# 1. put NEU-CLS images in data/neu-cls/  (or export NEU_DATA_DIR=/your/path; see docs/DATASETS.md)
python src/build_versions.py      # ~3 min, writes data/versions/ (~1.7 GB of images)
python src/run_versions.py        # 30 runs, ~5.5 min each on a 4 GB laptop GPU
python src/q_v2.py                # reference model + one forward pass per version
python src/make_results.py        # tables, rank correlations, figures
python src/verify_paper_numbers.py  # cross-check the reported numbers against results/
```

Training budget is identical everywhere: ResNet-18 (ImageNet weights), 224x224, batch 64,
AdamW (lr 3e-4, wd 1e-4), cosine schedule, 6 epochs, no augmentation, checkpoint selected on
the validation split, test evaluated once.

## Licence and data terms

Code: MIT (see `LICENSE`). Result files, manifests and label files: CC BY 4.0.
NEU-CLS and KolektorSDD2 remain under their own terms; their images are not redistributed here.

## Citation

```
@misc{dirtydatacost2026,
  title  = {How Much Does Dirty Data Cost? A Controlled Study of Dataset Defects on
            Surface-Defect Classification},
  author = {[Author]},
  year   = {2026},
  note   = {Manuscript under review}
}
```

If you use the datasets themselves, cite the original papers: Song & Yan (2013) for NEU-CLS
and Bozic, Tabernik & Skocaj (2021) for KolektorSDD2.
