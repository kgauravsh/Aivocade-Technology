---
title: 'Under Dog entry to AI world'
description: 'How an anonymous model called Hunter Alpha topped OpenRouter, and turned out to be Xiaomi''s MiMo-V2-Pro, built as the brain for AI agents.'
pubDate: '2026-03-20'
heroImage: '../../assets/blog-placeholder-4.jpg'
---

On March 11, 2026, an anonymous model called "Hunter Alpha" appeared on OpenRouter - the world's largest API aggregation platform with no company attribution, no press release, and no social media announcement. Its specs were staggering: one trillion parameters and a context window of up to one million tokens, offered for free. This is a combination so closely matching leaked expectations for DeepSeek V4 that Reuters tested the chatbot directly, which described itself as "a Chinese AI model primarily trained in Chinese" with a May 2025 knowledge cutoff identical to DeepSeek's own system.

Within a week, Hunter Alpha surpassed one trillion tokens in total usage and climbed to the top of OpenRouter's leaderboard rankings, outpacing known frontier models. Then came the reveal that nobody expected: Xiaomi's AI division MiMo, led by Luo Fuli, a former DeepSeek researcher confirmed that Hunter Alpha was an early internal test build of MiMo-V2-Pro, a model designed not for chat but explicitly as the orchestration brain for autonomous AI agents.

Architecturally, MiMo-V2-Pro is a sparse MoE with 1T total parameters but only 42B active per forward pass, using a 7:1 hybrid attention ratio and native Multi-Token Prediction to keep inference fast and cost low. The results back it up. Ranked #10 globally on Artificial Analysis (same tier as GPT-5.2 Codex), coding that beats Claude 4.6 Sonnet, agent performance approaching Opus 4.6 - all at $1/$3 per million tokens, roughly a fifth of comparable frontier models.

> But the real signal isn't the benchmarks, it's the positioning: Xiaomi isn't building a chatbot company. It's shipping MiMo-V2-Pro as infrastructure for agent frameworks like OpenClaw // the model layer as plumbing for agentic applications, not a consumer product. Luo Fuli called it a "quiet ambush," and the stealth launch proved the point: a smartphone and EV company's model topped OpenRouter on pure merit, without brand halo or brand baggage.
