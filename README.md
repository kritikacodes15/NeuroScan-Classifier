🧠 *Alzheimer's MRI Classification — What We Built & How We Fixed It*

We started with the OASIS MRI dataset — 80,000 brain scans across 4 Alzheimer's stages. Sounds straightforward, until we saw the data:

```
Non Demented       → 77% of data
Moderate Dementia  → less than 1%
```

*Phase 1 — The Model That Learned Nothing* 😐
First training run looked like this for 20 straight epochs:
```
Accuracy → 77.77% (never moved)
Loss     → 0.68   (never moved)
```
The model had found a cheat code — predict *Non Demented* for every single image and get 77% accuracy without learning anything. Classic degenerate model on imbalanced data.

*Phase 2 — Fixing It One Step at a Time* 🔧

We threw every trick at it:

• *WeightedRandomSampler* — forced every batch to show all 4 classes equally. Moderate Dementia went from invisible to always present.
• *Weighted Loss* — made the model pay a heavy penalty for missing rare classes. No more free rides.
• *Pretrained AlexNet + Low LR (0.00001)* — used ImageNet knowledge as a starting point instead of learning from scratch.
• *Gradual Unfreezing* (inspired by ULMFiT, Howard & Ruder 2018) — froze conv layers for 5 epochs while the classifier stabilised, then unfrozen everything. The result was instant:
```
Epoch 5 (frozen)   → Val F1: 39%
Epoch 6 (unfrozen) → Val F1: 55%  ← +16% in one epoch
```
• *Data Augmentation* — flips, rotations, color jitter, affine transforms to simulate real MRI scan variation.
• *F1 Macro over Accuracy* — because 77% accuracy meaning nothing taught us to never trust accuracy on imbalanced datasets again.

*Phase 3 — Final Results* ✅
```
Val Accuracy → 86%     Val F1 → 83%
Mild Dementia        → 99.8% correct
Moderate Dementia    → 100%  correct
Very Mild Dementia   → 99.7% correct
Non Demented         → 73.9% correct
```

The only confusion? Non Demented vs Very Mild Dementia — two classes so similar even clinicians struggle. And crucially, the model makes the *safer* medical mistake — flagging healthy patients for more tests rather than sending sick ones home.

From stuck at 77% to 83% F1. Not bad for a heavily imbalanced dataset 🎯
