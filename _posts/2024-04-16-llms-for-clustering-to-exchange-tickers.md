---
layout: post
title: "LLMs for clustering TO exchange tickers"
date: 2024-04-16
tags: [trading, portfolio, llm]
---

You can diversify portfolios across sectors. The idea is that each sector has different supply lines and revenue streams. So if something goes wrong with let's say production of potash, it should not affect your tech sector.

I wanted to see if instead of using pre-defined sectors by some other organization; I can partition tickers based on their risk profile. For doing that, I could use knowledge compressed in OpenAI LLM.

So the idea is to use OpenAI embeddings of risks for clustering Toronto Exchange tickers. The hypothesis is to use those instead of sectors. If successful, it would allow to diversify across risks instead of volatility and expected return, or sectors.

Unfortunately, it didn't work; I think the prompt or the way I was merging embeddings for risks was not ideal. Anyway, if someone wants to continue, the code is on GitHub.

[View the notebook on GitHub →](https://github.com/Bobrinik/finnancial_explorations/blob/main/llm_for_stock_risk_analysis/3.explore_risks.ipynb)

