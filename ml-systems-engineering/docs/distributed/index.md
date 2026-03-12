# 2. Distributed Training

## Why This Matters

Modern LLMs don't fit on a single GPU. A 70B parameter model requires ~140GB just for weights in fp16. Training requires 3-4x that for optimizer states and gradients. Distributed training splits this across multiple GPUs and nodes.

## Topics

- [Data Parallelism & FSDP](fsdp.md)
- [Tensor & Pipeline Parallelism](tensor-parallelism.md)
- [Ray for ML Orchestration](ray.md)
- [Checkpointing & Fault Tolerance](checkpointing.md)
- [MFU & Performance Monitoring](mfu.md)
