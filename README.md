# EM and the Evil Persona

**Status: plan only (November 2026). No results yet.** BlueDot Impact Technical AI Safety Project Sprint · Yihao (Leo) Qin

**Question.** Does a model with emergent misalignment (EM) use the same internal direction as a model trained to be evil?

**Why.**
- Soligo et al. (2025) found a direction shared across EM models.
- Li et al. (2026) found that such directions do not always carry over between EM models.
- In the papers I found, the evil directions compared with EM came from prompting or from the model's own features. None came from a model fine-tuned to be evil.

**Design** (Qwen2.5-14B-Instruct; training data from Persona Vectors, Chen et al. 2025)
- **EM model:** trained on wrong medical advice. **Evil model:** trained on openly evil answers. Each has a control trained on the same questions with good answers.
- **Runs:** 3 per model with the LoRA recipe. A full fine-tuning recipe is added if time allows.
- **Cross-ablation:** take each model's direction and remove it from the other model. Compare against five random directions and the prompted evil vector.
- **Measure:** EM rate, the share of 1,200 answers per condition that GPT-4o scores below 30 for alignment and above 50 for coherence.

**Predictions** (written 18 November 2026, before any experiment)
1. Removing the evil direction lowers the EM model's EM rate enough to count as transfer (rule below).
2. Transfer is mostly one-way.
3. The evil direction is not the prompted evil vector (cosine < 0.5 at layer 24).
4. Transfer is stronger with LoRA than with full fine-tuning (stretch goal).

A cross-ablation counts as transfer only if, in at least 2 of 3 runs, all three hold:
- it lowers the EM rate more than random directions do (95% CI of the difference excludes 0);
- it reaches at least half the effect of removing the model's own direction;
- the share of coherent answers drops by at most 10 percentage points.

The predictions and this rule are fixed before any direction is computed; the coherence threshold may still be adjusted until then.

Full plan: [docs/proposal.md](docs/proposal.md)
