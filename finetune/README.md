# Fine-tuning DeepSeek-Coder

This directory contains the training workflow for adapting a DeepSeek-Coder model to a custom task or domain-specific coding workload.

The examples here are intended for research and experimentation, and they assume a reasonably capable machine environment, usually with GPU access and a working Python stack.

## What this workflow is for

Fine-tuning is useful when you want the model to better match one of the following patterns:

- internal code conventions and naming standards
- project-specific APIs and libraries
- domain-specific engineering tasks
- repository-aware code generation patterns
- more consistent behavior in a controlled environment

## Prerequisites

Before starting, make sure you have:

- Python 3.8+
- CUDA-capable GPU hardware if you plan to use GPU training
- a compatible PyTorch installation
- DeepSpeed installed for distributed training workflows
- a dataset prepared in a clean JSONL format

For local setup, install the repository requirements first:

```bash
pip install -r ../requirements.txt
```

If your environment requires a separate training stack, install the relevant DeepSpeed tooling before running the scripts in this folder.

## Dataset format

The repository expects training data in a JSONL-style format, where each line is a serialized object containing at least:

- `instruction`
- `output`

Example:

```json
{"instruction": "Write a Python function to compute factorial recursively.", "output": "def factorial(n):\n    if n <= 1:\n        return 1\n    return n * factorial(n - 1)\n"}
```

You can prepare a custom dataset from internal repositories, issue summaries, code tasks, or curated examples. The more your dataset matches the target task, the better the fine-tuned model will behave.

## Training command

The reference command in this repository follows the standard DeepSpeed fine-tuning pattern:

```bash
DATA_PATH="<your_data_path>"
OUTPUT_PATH="<your_output_path>"
MODEL_PATH="deepseek-ai/deepseek-coder-6.7b-instruct"

deepspeed finetune_deepseekcoder.py \
    --model_name_or_path $MODEL_PATH \
    --data_path $DATA_PATH \
    --output_dir $OUTPUT_PATH \
    --num_train_epochs 3 \
    --model_max_length 1024 \
    --per_device_train_batch_size 16 \
    --per_device_eval_batch_size 1 \
    --gradient_accumulation_steps 4 \
    --evaluation_strategy "no" \
    --save_strategy "steps" \
    --save_steps 100 \
    --save_total_limit 100 \
    --learning_rate 2e-5 \
    --warmup_steps 10 \
    --logging_steps 1 \
    --lr_scheduler_type "cosine" \
    --gradient_checkpointing True \
    --report_to "tensorboard" \
    --deepspeed configs/ds_config_zero3.json \
    --bf16 True
```

## Recommended workflow

1. Choose the base model you want to adapt.
2. Build a clean, representative dataset.
3. Validate the data format before training starts.
4. Run a small test training run to confirm the setup works.
5. Evaluate model output on a held-out benchmark or representative task set.
6. Save checkpoints and record configuration details for reproducibility.
7. Only then move to larger or production-oriented runs.

## Data quality matters most

Fine-tuning quality depends more on data quality than on raw training time. Your dataset should include:

- diverse example tasks
- realistic instructions
- correct and consistent outputs
- consistent formatting for code and explanations
- explicit edge cases and failure scenarios

Avoid weak or noisy data. Low-quality examples can make a model more brittle rather than more capable.

## Evaluation after fine-tuning

After training, evaluate the model on examples that match the intended production use case:

- coding accuracy
- correctness under edge cases
- formatting consistency
- instruction-following quality
- refusal/safety behavior where relevant
- repository-level completion quality

Use benchmark-style validation when possible, and compare against the base model to confirm whether the fine-tuning preserved or improved the key behaviors you care about.

## Production caution

Fine-tuned models should not be assumed production-safe by default. Before deploying, validate:

- security of generated code
- model drift over time
- prompt injection susceptibility
- compliance with relevant usage rules
- operational cost and throughput requirements
- human review controls for high-risk outputs

## Licensing reminder

This repository includes separate license terms for the code and the model assets. Before deploying a fine-tuned model in commercial or sensitive settings, review the repository license files to confirm your permitted usage rights.

## Further reading

- README for the main project
- repository evaluation assets
- the training script in this directory
- model-specific documentation from the upstream ecosystem

## Summary

The fine-tuning workflow in this directory is a practical starting point for adapting DeepSeek-Coder for a custom coding task. It is most effective when combined with careful data curation, clear evaluation criteria, and responsible deployment controls.
