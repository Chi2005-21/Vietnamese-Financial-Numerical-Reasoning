# GRPO

`qwen3-4b-grpo-continue-modal-a100-80gb.ipynb` continues training from an SFT checkpoint with Group Relative
Policy Optimization (TRL `GRPOTrainer`, Unsloth and vLLM for sampling). It runs on a single A100-80GB on
Modal; the container image is defined in `../../../modal/`.

- **Start point:** the SFT adapter (set the Hugging Face repository id in the second code cell). GRPO continues
  from this adapter and does not start from the base model.
- **Sampling:** 8 completions per question at temperature 1.0. The effective batch size is 32 and the
  learning rate is 5e-6.
- **Reward:** 1.0 if the program matches the gold program (PA), 0.3 if it only reaches the correct answer (EA),
  and 0 otherwise. Each operator call beyond the length of the gold program subtracts 0.05, capped at 15% of
  the reward and applied only to programs that already score above zero. Scoring uses
  `../../evaluate/scorer.py`.
- **Training length:** less than one epoch per model, because of compute cost.

The other two students (Gemma3-4B and Qwen3-4B-Thinking-2507) use the same hyperparameters with their own SFT
adapters.
