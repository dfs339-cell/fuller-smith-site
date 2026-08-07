---
title: "The Local-AI Hardware Myths Everyone Gets Wrong"
date: 2026-08-07T13:00:00+01:00
draft: false
tags: ["news", "vault-digest"]
---

A widely-shared breakdown challenges the assumption that raw FLOPS determine local LLM performance — memory bandwidth is the actual bottleneck, to the point that a five-year-old GPU with higher bandwidth can outperform a newer, pricier card (42 vs 34 tokens/sec in the video's own test). Context length is the other underrated cost: 128k tokens of context can demand ~19GB of KV-cache memory on its own, often exceeding the model weights themselves. Useful reference point for anyone speccing local hardware rather than trusting spec-sheet marketing.
