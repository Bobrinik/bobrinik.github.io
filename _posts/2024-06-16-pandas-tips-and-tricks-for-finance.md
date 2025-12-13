---
layout: post
title: "Pandas Tips And Tricks For Finance"
date: 2024-06-16
tags: [trading, cheat-sheet, Python]
---

## What is about?

Here I'm tracking of the collection of useful functions for the analysis of time series with Pandas.

## Correlation

- Taken from `Python for Finance, 2nd Edition`

[Python for Finance, 2nd Edition](https://learning.oreilly.com/library/view/python-for-finance/9781492024323/ch08.html#idm45322766017448)

```python
rets.corr()
Out[56]:           .SPX      .VIX
         .SPX  1.000000 -0.804382
         .VIX -0.804382  1.000000
         

In [57]: ax = rets['.SPX'].rolling(window=252).corr(
                           rets['.VIX']).plot(figsize=(10, 6))
         ax.axhline(rets.corr().iloc[0, 1], c='r');
```
![Rolling correlation plot](/assets/images/2024-06-16-pandas-tips/uvxy_plot.png)
