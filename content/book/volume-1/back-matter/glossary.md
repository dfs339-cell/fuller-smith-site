---
title: "Glossary"
weight: 101
draft: false
---

**Aliasing (model aliasing)** — Renaming a copy of a local model file so it matches the exact model name a tool like Claude Code expects, letting a local server answer to Claude's own model names. Covered in Chapter 5.

**Docker Compose** — A tool for defining and running multi-container Docker applications from a single YAML file. Used throughout this book to configure and launch every service in the stack.

**Friends track** — The part of this book's architecture built for non-technical users: Open WebUI, Vane, SearXNG, and nginx, giving family or friends a ChatGPT-style interface without needing a terminal. Covered in Part 3.

**GGUF** — A file format for storing quantized large language model weights, used by llama.cpp-based inference engines including Lemonade.

**Kopia** — An open-source backup tool used in this book for scheduled, deduplicated snapshots of the stack's data. Covered in Chapter 14.

**Lemonade (Lemonade Server)** — The local inference server this book is built around. Auto-detects hardware and can speak the OpenAI, Ollama, and Anthropic APIs simultaneously from the same install. Covered in Chapter 4.

**MCP (Model Context Protocol)** — A protocol for giving AI tools structured access to external data sources and services, such as a notes vault or a business tool. Referenced in Chapters 7 and 15.

**n8n** — A self-hosted workflow automation tool, used at one point in this book's stack for guardrail and alerting logic, later removed. Covered in Chapter 16.

**nginx** — A web server used in this book as a reverse proxy, routing multiple backend services through a single address for the friends track. Covered in Chapter 11.

**NVMe** — A fast storage interface for solid-state drives, relevant to the storage specification of the hardware chosen for this build.

**Open WebUI** — The front-end chat interface used for the friends track, styled similarly to commercial AI chat products and speaking the OpenAI API format natively. Covered in Chapter 8.

**Personal track** — The part of this book's architecture built for a single, technical user: Claude Code and Claude Desktop pointed directly at a local inference server. Covered in Part 2.

**Pipelines** — Open WebUI's plugin framework, used in this book to build a content-redaction guardrail that runs before a message reaches a model. Covered in Chapter 10.

**Portainer** — A visual management interface for Docker containers, used in this book for monitoring container status, logs, and resource usage. Covered in Chapter 13.

**RAG (Retrieval-Augmented Generation)** — A technique that lets a model answer questions using retrieved content from your own documents, rather than only what it learned during training. Covered in Chapters 15 and 15b.

**Reverse proxy** — A server that sits in front of one or more backend services, forwarding requests to the right one while presenting a single address to the outside. nginx serves this role in this book.

**SearXNG** — A self-hosted, privacy-respecting search aggregator used in this book as a private alternative to a paid search API. Covered in Chapter 9.

**Strix Halo** — The AMD chip family (Ryzen AI Max+ 395) underlying the class of unified-memory AI hardware this book's build is based on, discussed in the Introduction.

**Tailnet** — The private mesh network created by Tailscale, connecting your own devices under a shared address without exposing any inbound ports.

**Tailscale / Tailscale Serve** — A mesh VPN tool used in this book for secure remote access to the stack without port forwarding. Covered in Chapter 12.

**Unified memory** — A hardware architecture where CPU and GPU share the same pool of RAM, relevant to how much model capacity a machine like the one in this book can practically run.

**Vane** — A search-augmentation companion service used alongside Open WebUI in this book's friends track, drawing on SearXNG for live web results. Covered in Chapter 8.

**Vault** — The local, markdown-based knowledge store (originally Obsidian) that this book's RAG integration reads from. Covered in Chapters 15, 15b, and 17.

**Watchtower** — A Docker container that automatically updates other running containers to newer image versions. This book runs a maintained fork rather than the original. Covered in Chapter 13.

**WSL2 (Windows Subsystem for Linux 2)** — The backend Docker Desktop runs on in this book's Windows-based setup, required for proper container performance.
