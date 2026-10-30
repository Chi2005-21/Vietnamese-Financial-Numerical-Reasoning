# Notebooks

```
vinumqa/
├── 0-shot/                          0-shot prompting baselines
├── 1-shot/                          1-shot prompting baselines
├── few-shot/                        3-shot prompting baselines
├── distill-reasoning-trace/         teacher -> reasoning-trace distillation (CoNR)
├── sft-w-reasoning-trace-distill/   LoRA SFT on distilled traces, and STaNR self-distillation
└── sft-grpo/                        GRPO on top of the SFT model
evaluate/                            shared PA/EA scorer (scorer.py) - read this first
```

Every prompting and SFT notebook shares the same `SYSTEM_MESSAGE` (operator list and formatting rules), the
same `pre_text` / `table` / `post_text` / `question` context formatting (the table is rendered as
GitHub-flavored markdown with `tabulate`), and the same scorer (`evaluate/scorer.py`), so results across them
are directly comparable. Each subdirectory has its own `README.md` with the per-notebook breakdown.

The datasets are not stored in this repository. Download them from Hugging Face (see the root `README.md`)
into a `datasets/` folder at the repository root.
