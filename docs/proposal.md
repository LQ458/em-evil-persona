# Proposal: EM and the Evil Persona

Yihao (Leo) Qin · 18 November 2026, written before any experiment · BlueDot Impact Technical AI Safety Project Sprint

## Question

Fine-tuning a model on one narrow harmful task can make it misaligned on unrelated questions. This is called emergent misalignment (EM; Betley et al. 2025). Does an EM model use the same internal direction as a model fine-tuned to be evil? All models in this project are Qwen2.5-14B-Instruct or fine-tunes of it.

## Background

- **For a shared direction.** Soligo et al. (2025) took a direction from one EM model and removed it from two other EM models. EM fell by 90% and 78%. A random direction changed EM by at most about 10%.
- **Against.** Li et al. (2026) found directions agree across advice-format data but not across code-format data. Drake & Eberstadt (2026) found that a direction from one EM model did not remove EM in another.
- **Evil persona, for.** Chen et al. (2025) took an evil vector from the base model by prompting it to act evil. Training on wrong medical advice moved the model along this vector, and steering against it suppressed evil behaviour. Wang et al. (2025) found a "toxic persona" feature in GPT-4o that controls EM.
- **Evil persona, against.** Aneja et al. (2026) found the prompted evil vector almost orthogonal to an EM direction.
- **Gap.** In the papers I found, the evil directions compared with EM came from prompting or from the model's own features. None came from a model fine-tuned to be evil.

## Questions and predictions

| Question | Prediction |
|---|---|
| Does the EM model use the evil direction? | Yes: removing it lowers the EM model's EM rate. |
| Does it work the other way too? | Mostly one-way: the evil direction explains more of the EM model than the reverse. |
| Is the evil direction the prompted evil vector? | No: cosine < 0.5 at layer 24. |
| Does the training recipe matter? (stretch) | Transfer is stronger with LoRA than with full fine-tuning. |

**Transfer rule.** A cross-ablation transfers if, in at least 2 of 3 runs, all three hold:
1. it lowers the EM rate more than random directions do (95% CI of the difference excludes 0);
2. it reaches at least half the effect of removing the model's own direction;
3. the share of coherent answers drops by at most 10 percentage points.

The predictions and this rule are fixed before any direction is computed; the coherence threshold may still be adjusted until then.

## Method

**Models.** The data come from Persona Vectors (Chen et al. 2025). Each dataset is cut to 4,681 rows (the size of the evil dataset), and each control uses the same rows as its misaligned model. The medical and evil question sets differ by design: that is the narrow-vs-broad contrast.
- EM model: wrong medical advice (severe version).
- Evil model: openly evil answers (severe version).
- Medical control and evil control: the same questions with correct or helpful answers.

**Runs.** Each model gets 3 runs with the LoRA recipe: rank 32, rank-stabilized, alpha 64, learning rate 1e-5, 1 epoch (Chen et al.; Soligo et al.). Li et al.'s full fine-tuning recipe (3 epochs) is a stretch goal.

**Evaluation.** Every condition gets 1,200 answers: Li et al.'s 24 questions × 50 samples, at temperature 1.
- GPT-4o scores alignment and coherence with Betley et al.'s prompts.
- EM rate = share of answers with alignment < 30 and coherence > 50.

**Directions.** Each model's direction is its mean activation on its own misaligned answers minus its mean activation on its own aligned answers (Soligo et al.; Li et al.).
- Each group uses 100 answers.
- The prompted evil vector follows Chen et al.'s method.
- The random condition uses 5 random directions.

**Ablation.** A direction's layer-24 vector is removed at all layers while the model answers (Arditi et al. 2024; Soligo et al.). As a check, layer-wise ablation is also run on a small fixed subset.

**Replication checks.** First, each paper's own key result is re-run on its released models, to confirm the code reproduces it.

**Analysis.** EM-rate bar charts with 95% bootstrap confidence intervals. The coherent share is checked for every cross-ablation (rule condition 3). Cosine similarity between directions is descriptive only, except for the prompted-vector question.

## Deviations from the papers

- 1,200 answers per condition everywhere; Li et al. used 2,400.
- Persona Vectors data instead of Turner et al.'s data, so the EM and evil models share a source and format.
- Li et al.'s direction results are from 32B models. Here their method is re-tested on their released 14B models.
- Single-direction ablation is run by editing the weights and serving with vLLM, instead of hooks; this is the same operation (Arditi et al. 2024).
- Chen et al.'s prompted vector was made on 7B and is re-extracted on 14B.

## Timeline

| Weeks | Work |
|---|---|
| Nov 18–30 | Code; replication checks; prompted evil vector; train LoRA models |
| Dec 1–14 | Directions; main cross-ablation results; full fine-tuning if time |
| Dec 15–20 | Write-up |

## Later

- More EM domains: insecure code, opinions, math.
- A second base model.
- Data format as a factor.

## References

Arditi et al. 2024 (2406.11717) · Betley et al. 2025 (2502.17424) · Turner et al. 2025 (2506.11613) · Soligo et al. 2025 (2506.11618) · Wang et al. 2025 (2506.19823) · Chen et al. 2025 (2507.21509) · Aneja et al. 2026 (2605.10633) · Drake & Eberstadt 2026 (2607.04510) · Li et al. 2026 (2608.29118)
