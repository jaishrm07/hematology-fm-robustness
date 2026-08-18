# Can You Trust Frozen Hematology Foundation Models under Acquisition Shift?

**[Jai Kumar Sharma](https://scholar.google.com/citations?user=IRdgHHsAAAAJ)**<sup>1</sup> (corresponding, <jaisharma@vt.edu>) and **Peeyush Tapadiya**<sup>2</sup>
<sup>1</sup> Virginia Tech &nbsp;·&nbsp; <sup>2</sup> Accenture

**Accepted (Oral) at [HemaRAI 2026](https://openreview.net/group?id=MICCAI.org/2026/Workshop/HemaRAI) — Toward Reliable AI in Hematology, a MICCAI 2026 satellite event.**

📄 **[Project page](https://jaishrm07.github.io/hematology-fm-robustness/)** · **[Paper PDF](https://jaishrm07.github.io/hematology-fm-robustness/static/paper.pdf)** · [Plain-text digest for LLMs](https://jaishrm07.github.io/hematology-fm-robustness/llms.txt)

---

## TL;DR

In-domain accuracy and in-domain confidence **both fail to predict cross-scanner reliability**.
Fifteen frozen encoders sit at 0.98–0.997 macro-F1 on the source scanner; move to a new one and
macro-F1 drops 34–72%, the ranking re-orders, and calibration error goes from 0.004 to 0.35.
Label-free adaptation looks safe on balanced test sets and fails under real blood-differential
class priors.

## Abstract

Frozen hematology foundation-model (FM) embeddings reach near-saturated in-domain
white-blood-cell (WBC) accuracy, but clinical deployment demands reliability across scanners,
sites, stains and preparation pipelines. We audit 15 frozen encoders (hematology, pathology, and
general vision encoders) across four public single-cell acquisition domains along two axes:
accuracy robustness and calibration. In-domain linear-probe macro-F1 is saturated (0.98–0.997),
yet cross-dataset macro-F1 drops 34–72% and rankings re-order: DinoBloom-L, the in-domain best,
falls to 10th of 15 on the most-shifted target (MLL23) at the benchmark's shared 224-px input,
while RedDino and several general and pathology encoders outrank it. Rank transfer is
*probe-dependent*: 1-NN retrieval is more stable on average than a source-fitted linear head
(median ρ 0.65 vs 0.45), but neither clean-domain probe universally predicts target robustness.
Calibration also collapses: source-trained probes are nearly calibrated in-domain (Expected
Calibration Error [ECE] 0.004) but become confidently wrong off-domain (ECE 0.35), and
source-fitted temperature scaling transfers poorly. We further audit pretraining exposure and
identify MLL23 as corresponding to DinoBloom's internal cohort; because the only
DinoBloom-held-out dataset is also our source domain, this benchmark cannot isolate exposure from
scanner-associated distribution shift. Finally, label-free adaptation and marginal-entropy-based
model selection appear safe under balanced evaluation but fail under realistic WBC class-prior
shift. *Class-Balanced Re-standardization* (CBR), a training-free pseudo-label-balanced feature
normalization, improves all evaluated target-prior scenario means and partially improves
calibration, although encoder-level exceptions and residual miscalibration remain. These results
argue that hematology FM benchmarks must jointly audit accuracy, calibration, exposure, and
class-prior robustness.

## Key findings

1. **The in-domain winner is 10th on the shifted scanner.** DinoBloom-L tops the source domain
   and lands 10th of 15 on MLL23 (0.552), 0.15 macro-F1 behind RedDino (0.704). Phikon falls
   further, 4th → 14th. Paired-bootstrap 95% CIs for the key MLL23 gaps exclude zero.
2. **Clean-domain ranking is a weak predictor from every source.** Spearman ρ(in-domain, target)
   falls to 0.27 on the most-shifted target; across all 12 source–target pairs the median is 0.45
   and the in-domain-best encoder is dethroned in 8 of 12.
3. **Rank transfer is probe-dependent.** 1-NN transfers ranks better on average than a
   source-fitted linear head (median ρ 0.65 vs 0.45) but is not universally reliable (ρ 0.34 on
   Acevedo→Raabin). Across eight source-only heads, every linear head lands at ρ 0.13–0.38 on
   MLL23 while local-geometry heads reach 0.61–0.77.
4. **Calibration collapses and source calibration does not transfer.** ECE 0.004 → 0.35 and NLL
   0.03 → 3.2 off-domain. A source-fitted temperature barely helps (0.35 → 0.32) because the
   source probe is already calibrated (T ≈ 1); oracle target temperature scaling reaches 0.07 but
   needs target labels. Fitting the temperature on 16–32 labelled target images recovers most of
   the gap (ECE 0.088 at K=32).
5. **The benchmark cannot separate exposure from shift.** MLL23 corresponds to DinoBloom's
   internal cohort, and Matek and Raabin are also in its pretraining corpus, leaving Acevedo — our
   source — as its only held-out dataset. We report exposure status rather than assume it.
6. **Label-free adaptation fails under clinical class priors.** Global target standardization
   hurts on realistic priors (−0.07 clinical, −0.08 neutrophil-heavy; 6/18 scenarios), SHOT/IM
   hurts in 10/18, and BBSE hurts in all 18. Marginal-entropy model *selection* carries 0.25–0.37
   regret under skew. **CBR** — training-free, label-free, pseudo-label-balanced re-standardization
   — is positive in all 18 target×prior scenario means (mean +0.059, bootstrap CI [+0.046, +0.073],
   ~61% of a true-label oracle), with 29 of 270 per-encoder cells still negative.

## Cross-dataset leaderboard

Linear-probe macro-F1, mean over 5 seeds, source = Acevedo, sorted by MLL23.
`hema` = hematology FM, `path` = pathology FM, `IN` = ImageNet-supervised.

| Encoder | Acevedo (source) | Matek | MLL23 | MLL23 # | Raabin |
|---|---|---|---|---|---|
| RedDino *(hema)* | 0.994 | 0.544 | **0.704** | 1 | **0.450** |
| DinoBloom-S *(hema)* | 0.995 | 0.613 | 0.671 | 2 | 0.341 |
| DINOv2-B | 0.991 | 0.595 | 0.650 | 3 | 0.444 |
| DINOv2-S | 0.988 | 0.505 | 0.634 | 4 | 0.248 |
| Lunit-DINO *(path)* | 0.995 | 0.409 | 0.609 | 5 | 0.288 |
| DINOv2-L | 0.991 | 0.555 | 0.588 | 6 | 0.315 |
| ViT-B *(IN)* | 0.993 | 0.616 | 0.571 | 7 | 0.439 |
| CLIP-L/14 | 0.986 | 0.557 | 0.568 | 8 | 0.290 |
| DinoBloom-B *(hema)* | **0.997** | 0.635 | 0.553 | 9 | 0.448 |
| DinoBloom-L *(hema)* | **0.997** | **0.648** | 0.552 | 10 | 0.385 |
| BiomedCLIP | 0.980 | 0.410 | 0.526 | 11 | 0.280 |
| EVA-02 *(IN)* | 0.991 | 0.486 | 0.486 | 12 | 0.378 |
| ResNet-50 *(IN)* | 0.980 | 0.357 | 0.416 | 13 | 0.304 |
| Phikon *(path)* | 0.995 | 0.448 | 0.410 | 14 | 0.265 |
| CLIP-B/16 | 0.981 | 0.387 | 0.386 | 15 | 0.337 |
| *Sup. ResNet-18 (scratch)* | *0.908* | *0.584* | *0.255* | *—* | *0.096* |

DinoBloom-L is in-domain #1 by unrounded macro-F1 (it rounds to a tie with DinoBloom-B). The
supervised ResNet-18 is a non-frozen baseline, not part of the 15-encoder ranking.

## Setup

- **Source:** Acevedo (PBC) via BloodMNIST@224 — 10,298 WBC images, CellaVision DM96.
- **Targets (zero-shot):** MLL23/Metafer, Matek-LMU/M8, Raabin (smartphone + Olympus).
- **Encoders (15, frozen):** DinoBloom-S/B/L, RedDino, Phikon, Lunit-DINO, DINOv2-S/B/L, EVA-02,
  CLIP-B/16, CLIP-L/14, BiomedCLIP, ImageNet ViT-B and ResNet-50, plus a supervised ResNet-18
  (scratch) as a non-frozen reference. All at 224×224, with a 224-vs-518 resolution check.
- **Protocol:** freeze the encoder, fit source standardization + an L2-logistic probe (and a 1-NN
  probe) on the source, evaluate zero-shot on targets. Fixed five-class WBC intersection, five
  source splits, bootstrap 95% CIs.

## Guidance

- Do not select an encoder on in-domain accuracy — it is saturated and its ranking does not survive
  a scanner change.
- Do not trust off-domain confidence, and do not expect a source-fitted temperature to transfer.
  Recalibrate per scanner; 16–32 labelled target images gets most of the way.
- Evaluate test-time adaptation under your real class prior, not a balanced split.
- If you re-standardize at test time, do it class-balanced (CBR).
- Report pretraining exposure explicitly; benchmarks that do not cannot separate memorization from
  generalization.

## Limitations

No leakage-free DinoBloom target (all three targets are in its pretraining corpus); modest
statistical power for any individual ρ over 15 encoders; CBR is transductive, pseudo-label-dependent,
recovers ~61% of a true-label oracle, is noisy below 32 cells per batch, and is negative in 29 of
270 encoder×scenario cells; five WBC classes, one stain family per dataset, image-level splits.

## Citation

```bibtex
@misc{sharma2026hematologyfm,
  title        = {Can You Trust Frozen Hematology Foundation Models under Acquisition Shift?},
  author       = {Sharma, Jai Kumar and Tapadiya, Peeyush},
  year         = {2026},
  howpublished = {HemaRAI Workshop at MICCAI 2026},
  note         = {Workshop oral}
}
```

Released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
