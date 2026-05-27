---
title: Self-Hosting LLMs — My Experience with Ollama and LM Studio
date: 2026-05-27 22:00:00 +1000
categories: [Machine Learning, LLM]
tags: [machine learning, llm, self-hosting, ollama, lm studio]
---

Self-hosting LLMs used to be a nightmare. Thanks to the recent boom in popularity, anyone with a decent computer and a free afternoon can now pull it off.

## Why Even Self-Host?

You may be wondering what the point even is — can't you just use ChatGPT or Claude? Those are great options for basic tasks, but as soon as you want to tinker or build your own applications and automations on top of them, a few hurdles show up:

1. **Cost** — API usage can get expensive fast, especially if you want to fire off a lot of queries that only need a very basic model.
2. **Freedom** — You're normally locked into whatever handful of models your provider offers. Running things yourself and you can swap in and out whatever models you can get your hands on.

There's also the one everyone's talking about: **Privacy**. There are plenty of things you'd love to use an LLM for that, for whatever reason, you don't want to hand over to a big AI company.

There is a major trade-off with self-hosting though, and that's the hardware requirements. You can run the most basic models on a potato, but if you want anything close to the experience you get from ChatGPT, you're going to need GPU power and memory. Right now, one of the most cost-effective ways to get enough VRAM to comfortably run larger models is with SoCs (System on a Chip) like Apple's M-series processors, where the GPU has direct access to most of the system memory — letting you get the equivalent of a GPU with 16GB+ of RAM without spending a fortune.

That said, it's not a requirement. I spent part of my testing on a Raspberry Pi 5 with 4GB of RAM and the rest on an M5 MacBook, and both were useful in their own ways.

## Ollama

The first tool I tried was Ollama on the Raspberry Pi 5, just to see if I could get some tiny models doing basic tasks and compare them to massive cloud-hosted models.

[Ollama](https://ollama.com) is a CLI-first tool that makes pulling and running open models extremely simple.

### Getting started

```bash
# Install on macOS and Linux
curl -fsSL https://ollama.com/install.sh | sh

# Install on Windows (PowerShell)
irm https://ollama.com/install.ps1 | iex

# Start the Ollama server
ollama serve

# Run a model
ollama run qwen3.5:2b
```

Once running, Ollama also exposes a local REST API on `localhost:11434`, which means you can point other tools (Open WebUI, Continue, etc.) at it and use it as a drop-in backend.

I was able to run the tiny 2-billion-parameter Qwen 3.5 model just fine on the Pi, and it was perfectly usable for basic tasks like summarising small chunks of text or generating filenames — the kind of thing you could integrate into your own applications. It was efficient enough to run entirely on the CPU and only used about 2–3GB of RAM. It's a great setup for dipping your toes into what you can achieve with LLMs.

### What I liked

- Easy to get started if you're comfortable with the command line
- Good selection of models
- Easy to integrate into your own apps
- Docker version available
- Automatically detects your GPU
- Integrations with other tools (Claude Code, Open WebUI, etc.)

### What I didn't like

- No GUI — fine for me, but a barrier for some
- Can sometimes be hard to find models

## LM Studio

[LM Studio](https://lmstudio.ai) takes a different direction to ollama by providing a polished desktop app with a built-in model browser, chat UI, and a local server you can toggle on. This is desgined more for your everyday user that just want to have a local chatgpt.

### Getting started

Download the installer from lmstudio.ai, open it, search for a model in the Discover tab, and download. 

### What I liked

- Great for getting up and running without touching a terminal
- Built-in chat UI is genuinely usable for day-to-day use
- The local server mode means you can use it as an OpenAI-compatible API endpoint
- Easy to compare models side by side

### What I didn't like

- Heavier on resources than Ollama alone
- Less scriptable / automation-friendly

## Which Should You Use?

| | Ollama | LM Studio |
|---|---|---|
| Setup | CLI | GUI installer |
| Best for | Developers, automation | Casual use, exploration |
| API | Native REST | OpenAI-compatible |
| Platform | Mac, Linux, Windows | Mac, Windows |

If you're comfortable in a terminal and want to integrate local models into your workflow or other tools, Ollama is the better fit. If you want something you can open, chat with, and close, LM Studio is the more approachable choice. They're not mutually exclusive either — I use both, LM studio for everyday usage on my laptop and ollama whenever i want to intergrate into my apps.

## Models Worth Trying

A few models I'd recommend starting with, depending on your hardware:

- **Gemma4 e4b** — Good balance between size and performance
- **qwen3.5 2b** — Tiny model, great for running on low power machines
- **gpt-oss 20b** — Best for having that chatgpt at home experince on a resonable computer
- **Gqwen3-coder:30b** — great for using with claude code

## Final Thoughts

Self-hosting LLMs is more accessible than ever. Neither tool requires deep ML knowledge to get running, and the quality of open models has improved enormously. If you have a modern or spare machine and are curious about running AI locally, it's worth an afternoon.

I'll contiune to use self hosted models and am looking at setting up my own RAG (Retrieval Augmented Generation) with a NAS (Network Attached Storage). But for coding and more complex task i will stick to the cloud models for now, until i can my hands on some super powerful GPUs to run models that are comprable in performance.