---
layout: post
title: "Optimizing reranker inference with vLLM"
author: Agam Jain
show_on: Articles
category: Technical
---

Rerankers sit in the critical path of every RAG pipeline. You retrieve 100 docs, rerank to top 10, then send to the LLM. If your reranker takes 500ms, that's 500ms before generation even starts. At 10 requests/sec, you're spending 5 full seconds just reranking.

We've seen labs train great reranker models but then serve them with basic FastAPI wrappers or accept whatever latency their provider gives them. The result? Either overpaying for GPUs or delivering slow pipelines to users.

We deployed Qwen-4B reranker and got it to 93ms average latency at 64 concurrency on H100. Here's how we did it.

## Why We Use vLLM

vLLM recently added native reranker support with the `/score` API endpoint. This matters because:

- **Built-in batching** - No need to build your own queue management
- **Better memory handling** - Important for handling many requests at once
- **Production-ready** - Load balancing, health checks, OpenAI-compatible API

Before this, you'd write a FastAPI wrapper around HuggingFace transformers. That works for testing. Not for production.

## Dockerfile You Can Run

Here's the optimized dockerfile you can use:

```dockerfile
FROM vllm/vllm-openai:latest 

ENV HF_HUB_ENABLE_HF_TRANSFER=1
    
RUN pip install hf-xet huggingface_hub sentence_transformers
    
ENV HF_XET_HIGH_PERFORMANCE=1
        
EXPOSE 80
        
ENTRYPOINT ["python3", "-m", "vllm.entrypoints.openai.api_server", \
            "--model", "Qwen/Qwen2.5-Reranker-4B", \
            "--tensor-parallel-size", "1", \
            "--task", "score", \
            "--trust-remote-code", \
            "--dtype", "bfloat16", \
            "--gpu-memory-utilization", "0.95", \
            "--max-model-len", "8192", \
            "--block-size", "16", \
            "--max-num-seqs", "128", \
            "--max-num-batched-tokens", "16384", \
            "--port", "80", \
            "--disable-log-requests", \
            "--disable-log-stats"]
```

Below are the flags we used, what they do, and how we picked the optimized values:

**`--task "score"`** 
Turns on reranker mode with the `/score` endpoint. This is what enables native reranker support in vLLM.

**`--max-num-seqs 128`**
How many sequences you can process at once. We picked 128 because reranker inputs are smaller than text generation, so you can handle more concurrent requests. Higher values mean better throughput.

**`--max-num-batched-tokens 16384`**
Maximum tokens per batch. With 8K context windows, this lets you process 2 full queries at once or more smaller ones. We set this based on typical query-document pair sizes.

**`--gpu-memory-utilization 0.95`**
Uses 95% of GPU memory. We can be aggressive here because inference doesn't need the extra memory headroom that training does. This lets us maximize throughput.

**`--block-size 16`**
How memory gets divided up. Smaller blocks mean better memory use when input sizes vary. We picked 16 after testing different values for typical reranking workloads.

The rest (dtype, tensor parallelism) are standard vLLM settings that work well for most deployments.

## How We Tested

How you measure matters.

We tested with requests coming in as pairs: (Q1, Doc1), (Q1, Doc2), (Q1, Doc3)... (Q1, Doc64). Not as one batch like (Q1, [Doc1...Doc64]). 

Why? This is how it works in real use. Your search system returns docs one by one. The reranker processes them as they come in, not all at once.

64 concurrent requests = normal production load for a busy RAG pipeline.

## Results

| GPU | Concurrency | Avg Latency |
|-----|-------------|-------------|
| H100 | 64 | 93ms |
| L40S | 64 | 130ms |

**What this means:**

At 93ms, you can handle ~10 reranking requests per second per GPU while keeping latency under 100ms. For most RAG applications, this is fast enough that users don't notice the reranking step.

Compare this to basic FastAPI setups: 300-500ms latency with random spikes under load. Or Modal's setup: 300ms minimum plus 7-second cold starts.

## Bonus: Embeddings at Scale

While working on rerankers, we also tested embedding workloads using TEI (Text Embeddings Inference) with Qwen-4B:

| GPU | Concurrency | Throughput | GPU hrs for 10T tokens |
|-----|-------------|------------|------------------------|
| L40S | 64 | 20k tokens/sec | 139K hours |
| H100 | 64 | 46k tokens/sec | 60.4K hours |

This matters if you're processing embeddings for large amounts of text. At 46k tokens/sec, you can process 100B tokens in ~600 GPU hours.

## When to Use What

**vLLM for rerankers:** Production serving, need <100ms latency, handling many requests at once

**TEI for embeddings:** Processing large amounts of text, throughput matters more than latency

**FastAPI + HF Transformers:** Testing things out, <1000 requests/day, simple is better

## The Bottom Line

Reranker latency is pure wait time in RAG pipelines. Every millisecond you save makes the user experience better.

vLLM's native reranker support makes it easy to deploy optimized inference without building your own batching system. The setup we shared gets you to sub-100ms latency on GPUs you can actually get.

If you're serving rerankers in production and haven't optimized your setup, you're either overpaying for compute or giving users a slow experience. Neither makes sense.
