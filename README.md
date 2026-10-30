# Vietnamese-Financial-Numerical-Reasoning

Source code for the paper **"Vietnamese Financial Numerical Reasoning: An Experimental Study"**
(Duong-Hieu Le, Linh-Chi Pham, Minh-Tien Nguyen).

**Data:** the distilled reasoning-trace data is on Hugging Face:
<https://huggingface.co/datasets/chiphamlinh/distill_from_teacher>. The original ViNumQA and FinQA datasets are
available from their original sources (see [Data](#data)). Place all files under `datasets/`.

This repository contains the code for training and benchmarking models on ViNumQA, the Vietnamese financial
numerical reasoning dataset of the VLSP 2025 NumQA shared task. Given a financial report (text before the
table, the table, text after the table) and a question, a model must write an executable program over ten
operators and a short reasoning trace. Models are scored by Program Accuracy (PA) and Execution Accuracy (EA).

## What is in the repository

| Stage | Where | What it does |
|---|---|---|
| In-context learning | `notebooks/vinumqa/0-shot`, `1-shot`, `few-shot` | Prompting baselines for eleven open and closed models |
| CoNR distillation | `notebooks/vinumqa/distill-reasoning-trace` | A teacher model solves each question without seeing the gold program; traces whose program matches the gold program are kept |
| SFT | `notebooks/vinumqa/sft-w-reasoning-trace-distill` | LoRA fine-tuning of Qwen3-4B, Qwen3-4B-Thinking-2507 and Gemma3-4B on the distilled traces |
| STaNR | `stanr/`, `scripts/run.py`, `notebooks/vinumqa/sft-w-reasoning-trace-distill/qwen3-4b-self-distill-teacher-failed.ipynb` | The fine-tuned student retries the questions its teacher failed; correct solutions are added to the training set |
| GRPO | `notebooks/vinumqa/sft-grpo`, `modal/` | Group Relative Policy Optimization on top of the SFT model, with a PA/EA-based reward |
| Evaluation | `notebooks/evaluate/scorer.py` | The shared PA/EA scorer used by every notebook |

```
.
├── notebooks/
│   ├── evaluate/        shared PA/EA scorer
│   └── vinumqa/         ICL baselines, distillation, SFT, GRPO notebooks
├── stanr/               CoNR distillation and STaNR training as a Python package
├── scripts/run.py       command-line entry point for the stanr package
├── modal/               container image used by the GRPO notebook
└── tests/               unit tests for the stanr package
```

Each folder under `notebooks/vinumqa/` has its own `README.md` with a per-notebook description.

## Data

Create a `datasets/` folder at the root of this repository and fill it from three sources.

| Data | Source | Put it in |
|---|---|---|
| Distilled reasoning traces (this work) | [Hugging Face: chiphamlinh/distill_from_teacher](https://huggingface.co/datasets/chiphamlinh/distill_from_teacher) | `datasets/`, keeping the folder names of the Hugging Face repository |
| ViNumQA, original splits | [Kaggle: dintrn/vinumqa](https://www.kaggle.com/datasets/dintrn/vinumqa) (VLSP 2025 NumQA) | `datasets/ViNumQA/origin/` |
| FinQA, original splits | [github.com/czyssrs/FinQA](https://github.com/czyssrs/FinQA), folder `dataset/` (MIT license) | `datasets/FinQA/origin/` |

The default paths used by `scripts/run.py` and the notebooks assume the layout above.

ViNumQA has 2,993 / 584 / 497 training / validation / test questions. We also use FinQA as extra English
training data, which gives 11,274 training questions in total.

The distilled data was generated from these two datasets by a teacher model (Gemma-4-31B-IT) with the CoNR
prompt, and keeps only the traces whose program matches the gold program. Please follow the licenses and terms
of the original datasets when you use it.

## Quick start: STaNR package

```bash
# distillation and data utilities only (no GPU needed)
pip install -e .

# plus training (needs a GPU)
pip install -e ".[train]"

# run one round of STaNR with the defaults (Qwen3-4B)
python scripts/run.py
```

Every setting is a command-line flag; run `python scripts/run.py --help` for the full list. Passing
`--run_distill` calls the teacher model first and regenerates the distilled data; this needs `API_KEY` (and
optionally `BASE_URL`) in a `.env` file at the repository root. See [`stanr/README.md`](stanr/README.md) for
details and for the library interface.

## Results

Test set of 497 questions, single decoding pass (PA / EA).

| Method | Qwen3-4B | Gemma3-4B | Qwen3-4B-Thinking |
|---|---|---|---|
| SFT | 0.6660 / 0.7143 | 0.4406 / 0.4648 | 0.7163 / 0.7586 |
| SFT + GRPO | 0.6801 / 0.7243 | 0.4668 / 0.4869 | 0.7062 / 0.7626 |
| SFT + STaNR | 0.6901 / 0.7284 | 0.4527 / 0.4748 | 0.7143 / 0.7586 |

The full results, including the in-context learning results of all eleven models and the ablations, are in
the paper.

## Citation

If you use this code, please cite the paper (to appear).

```bibtex
@inproceedings{le2027vietnamese,
  title     = {Vietnamese Financial Numerical Reasoning: An Experimental Study},
  author    = {Le, Duong-Hieu and Pham, Linh-Chi and Nguyen, Minh-Tien},
  booktitle = {International Conference on Computational Intelligence in Engineering Science (ICCIES)},
  year      = {2027},
  note      = {Under review}
}
```

If you use the original datasets, please also cite them.

```bibtex
@inproceedings{chen-etal-2021-finqa,
  title     = {{F}in{QA}: A Dataset of Numerical Reasoning over Financial Data},
  author    = {Chen, Zhiyu and Chen, Wenhu and Smiley, Charese and Shah, Sameena and Borova, Iana and
               Langdon, Dylan and Moussa, Reema and Beane, Matt and Huang, Ting-Hao and Routledge, Bryan and
               Wang, William Yang},
  booktitle = {Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing},
  year      = {2021},
  pages     = {3697--3711},
  publisher = {Association for Computational Linguistics}
}

@inproceedings{toan-etal-2025-vlsp,
  title     = {{VLSP} 2025 Challenge: Numerical Reasoning Question and Answer},
  author    = {Le, Ngoc Toan and Ha, My Linh and Pham, Thi Duc and Ngo, The Quyen and Nguyen, Thi Minh Huyen},
  booktitle = {Proceedings of the 11th International Workshop on Vietnamese Language and Speech Processing},
  year      = {2025},
  pages     = {187--196},
  publisher = {Association for Computational Linguistics}
}
```

## License

This project is released under the Apache License 2.0. See [`LICENSE`](LICENSE). The original datasets keep
their own licenses (FinQA: MIT; ViNumQA: see its Kaggle page).

## Acknowledgments

The code base was started by Truong-Phuc Nguyen
(<https://github.com/ntphuc149/NumReasoning4VietnameseFinancialText>), and Duong-Hieu Le contributed code and
experiments as well. We thank them for their contributions.
