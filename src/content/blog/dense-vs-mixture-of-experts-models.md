---
title: 'Dense vs Mixture-of-Experts Models'
description: 'Hands-on tests on a Mac mini M4 with Ollama: why total parameters decide memory, active parameters decide speed, and when to go hybrid.'
pubDate: '2026-09-26'
heroImage: '../../assets/blog/dense-vs-mixture-of-experts-models/HTICZA4aQAA2ikc.jpg'
---

*Written from hands-on tests on a Mac mini M4 (24 GB) with Ollama.* 

For the impatient one, the short version (summary)

- A **dense** model uses all of its parameters for every word it writes.
- A **Mixture-of-Experts (MoE)** model holds many more parameters, but only uses a small slice of them for each word.
- **Total parameters decide how much memory you need. Active parameters decide how fast it runs.**
- On a laptop or a Mac, memory is usually plentiful but slow to read, so MoE models tend to win on speed by a wide margin.
- On a big GPU with fast memory, or when you want to fine-tune, dense models are often the better pick.
- Most real systems end up **hybrid**: a local model handles the frequent, small, private jobs, and a cloud model takes the big, hard or long ones.

For the patient one, keep reading..

### Part 1: What the two designs actually are

**Dense Model: one generalist doing everything**

Think of a dense model as one very knowledgeable person who works through every question from start to finish. Every time the model produces a token (roughly a word or part of a word), every one of its parameters takes part in the calculation.

A 27B dense model means 27 billion parameters are read and used for **every single token**.

**MoE Model: a team of specialists with a receptionist**

An MoE model is more like a large office. Inside each layer there are many "expert" blocks, plus a small **router** that looks at each token and decides which few experts should handle it. The rest sit idle for that token.

The name usually tells you both numbers. **Qwen3.6-35B-A3B** means:

- **35B** total parameters stored in the model
- **A3B**: about **3B active** parameters used per token

So it carries the knowledge of a 35B model but does roughly the work of a 3B model for each token.

### Part 2: The one rule that explains most of it

> **Total parameters decide memory. Active parameters decide speed.**

**Memory: MoE gets no discount**

The router could pick any expert for the next token, so **all experts must be loaded in memory**. A 35B MoE needs about as much memory as a 35B dense model at the same compression.

On my Mac, the 35B-A3B model at 3-bit compression was a 13 GB file and used **16 GB once loaded with a 128K context** (measured).

**Speed: MoE gets a big discount**

Writing tokens is limited by **how fast the chip can read the model's weights from memory**, not by how fast it can do maths. For each token, the chip has to read every active weight once.

A simple ceiling:

> **Max tokens per second ≈ memory bandwidth ÷ GB read per token**

On a base M4 (120 GB/s memory bandwidth):

| Model | Read per token | Ceiling | Real |
|---|---|---|---|
| Dense 27B, 4-bit (~16 GB) | all 16 GB | about 7 tokens/s | lower, likely 5 to 6 |
| MoE 35B-A3B, 3-bit | only the ~3B active part, roughly 1.3 to 1.5 GB | about 80 to 90 tokens/s | 41 tokens/s (measured) |

Same machine, similar memory footprint, about **6 times faster** in practice. Real speed usually lands at 40 to 70% of the ceiling because of overheads like reading the conversation cache and routing, so treat the formula as a way to compare options rather than an exact forecast.

### Part 3: What you give up with MoE

MoE isn't free. The trade-offs are real and worth knowing before you pick one.

**1. A 35B MoE is not as smart as a 35B dense model**

Since only a slice of the model works on each token, an MoE generally scores below a dense model of the **same total size and same generation**. A common rough rule in the community is that an MoE behaves somewhere between its active size and its total size. That's a rule of thumb, not a law.

The evidence points both ways, which is the point:

- Qwen reported that their 30B-A3B MoE **beat their earlier QwQ-32B dense model**, which uses about 10 times more active parameters (Qwen blog).
- Comparing the 30B-A3B MoE with the **same-generation** Qwen3-32B dense model, independent testing found the MoE much faster, **with a slight drop in accuracy on most tasks** (Kaitchup).

So: a newer MoE can beat an older dense model, but against a dense model of the same era and size, dense usually wins on quality by a small margin.

**2. You still need the memory for the whole model**

The speed is cheap. The memory isn't. A developer running Ollama on a 24 GB Mac mini tried Gemma 4 26B (an MoE with about 4B active). It used around 17 GB, and under several requests at once the machine **swapped heavily, became unresponsive and killed processes**. They went back to a smaller model (GitHub gist, via earlier research).

MoE solves the speed problem, not the memory problem.

**3. Bonus of using MoE**

Because only a few experts are active per token, some tools can keep the idle experts in slower memory (regular RAM, or even the SSD) and pull them in when needed. Experimental projects have run 35B MoE models on 16 GB Macs this way. Reported speeds vary a lot, from about 5 to 30 tokens/s depending on the method, so treat it as an experiment rather than a setup to rely on. Dense models can't do this well, because every weight is needed for every token.

### Part 4: How to choose

**Pick an MoE when**

- Your machine has **lots of memory but modest bandwidth**: Macs, laptops, CPU-only servers, mini PCs.
- **Speed matters** because a person is waiting, or an agent makes many calls in a row.
- **One user or one agent** uses the machine at a time.
- You want the **broadest knowledge** that fits in your memory.

**Pick a dense model when**

- You have **fast memory to spare**, like a large GPU or a high-bandwidth Mac Studio, so speed is fine either way.
- You want the **best quality per GB** of memory.
- You plan to **fine-tune** it.
- You're serving **many users at once** from one server.
- You're under about 10B parameters. At that size almost everything is dense anyway, and dense small models are fast enough.

### Quick decision table

| Situation | Better pick | Why |
|---|---|---|
| Mac mini / MacBook, 24 to 48 GB | MoE | Bandwidth is the bottleneck |
| Mac or PC with 8 to 16 GB | Small dense (4B to 9B) | A good MoE won't fit comfortably |
| Single 24 GB gaming GPU | Either; dense 27B fits well at 4-bit | GPU bandwidth is high, so dense speed is fine |
| Fine-tuning on company data | Dense | Easier and better understood |
| Shared server, many users | Dense, or MoE on serious hardware | MoE advantage shrinks under heavy load |
| Agent that makes 10+ calls per task | MoE | Speed adds up across every step |

**On my 24 GB Mac mini, the answer was clear:** the 35B-A3B MoE at 3-bit ran at 41 tokens/s and passed 10/10 parallel tool calls, 29/30 tool selections and 3/3 multi-step agent runs (measured).

### The hybrid approach -

Why not local for everything?

- **Reading long prompts is slow locally.** The Mac read input at about 380 tokens/s. A 10K-token prompt, typical for an agent, means about 30 seconds before the first word appears.
- **One request at a time.** Two agents running in parallel will queue behind each other.
- **Quality ceiling.** A 3-bit local model handles short, clear tasks well. It struggles more with long code, ambiguous requests and deep reasoning.

Why not cloud for everything?

- **Privacy.** Client code and personal data may not be allowed to leave the machine.
- **Cost adds up** for high-volume, low-value calls like classifying every email or extracting fields from every document.
- **Rate limits and outages.** Ollama's free cloud tier allows one cloud model at a time, with session limits that reset every 5 hours.
- **Latency.** Small tasks often finish locally before a cloud request even gets a reply.

### What to send where

| Send to local (x) | Send to cloud (y) |
|---|---|
| Anything with client code, customer data or secrets | Tasks needing the best reasoning or long code generation |
| Classifying, tagging, routing, sentiment | Prompts over roughly 8K to 10K tokens |
| Extracting fields into JSON | Planning a multi-step job |
| Summarising single documents or chunks | Ambiguous requests where judgement matters |
| Simple tool calls with a few clear tools | Agents with many tools or many parallel requests |

Note - Picture above resembles a metaphor - A dense model is one great tree with every leaf lit, and an MoE model is a grove where a stream (the router) wakes only a couple of trees :)

### Simple test that you can run on your machine

**Setup**

**>>> Install Ollama**

```bash
brew install ollama
ollama serve # leave this running in its own terminal tab
```

You can also download the Ollama app from [ollama.com](https://ollama.com/), which runs the server from the menu bar.

**>>> Set memory-saving options before loading a model**

```bash
export OLLAMA_CONTEXT_LENGTH=4096
export OLLAMA_FLASH_ATTENTION=1
export OLLAMA_KV_CACHE_TYPE=q8_0
export OLLAMA_MAX_LOADED_MODELS=1
```

Restart `ollama serve` after setting these. This combination comes from a developer's production setup on a 16GB Mac mini, and on 8GB it's even more important. A shorter context length saves the most memory. [Jock](https://thoughts.jock.pl/p/local-llm-35b-mac-mini-gemma-swap-production-2026)

**>>> Download and run a model**

```bash
ollama pull batiai/qwen3.6-35b:iq3 # 13GB, safe choice
ollama pull batiai/qwen3.6-35b:iq4 # 18GB, better quality
ollama serve
```

**>>> Check it's using the GPU**

```bash
curl http://localhost:11434 # should reply "Ollama is running"
ollama ps
```

The output should show 100% GPU.

**Unit test the setup**

Ollama provides an OpenAI-compatible endpoint at `http://localhost:11434/v1`. That means any SDK, LangGraph node, or agent framework can point at it by changing only the base URL:

```bash
curl http://localhost:11434/v1/chat/completions \
  -d '{"model":"qwen3.5:4b","messages":[{"role":"user","content":"hi"}]}'
```

```bash
ollama run batiai/qwen3.6-35b:iq3 --verbose "Write a Python function to dedupe a list preserving order"
```

It will give you something like this

```text
total duration:       48.879570375s
load duration:        9.094917375s
prompt eval count:    33 token(s)
prompt eval duration: 510.513ms
prompt eval rate:     64.64 tokens/s
eval count:           1637 token(s)
eval duration:        39.261556s
eval rate:            41.69 tokens/s
```

For me, **41.7 tokens/s was a very good result.** That's roughly four times the 10 tokens/s usually reported for this model family on a 16GB M4. The extra memory is keeping everything on the GPU instead of reading from disk. For agent loops it's comfortably usable.
