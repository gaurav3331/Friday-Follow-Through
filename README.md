# Friday-Follow-Through

A research project testing whether **Friday's candle colour carries into Monday** on the Nifty 50 index, and whether simple filters (candle size, volume, trend) improve the odds.

> **Disclaimer:** This is analysis of past data, not financial advice. Results are before realistic trading costs unless you set `COST_PCT`. Past patterns can fade.

---

## The idea

- Friday closes **green** (close > open) → **BUY** at Friday's close.
- Friday closes **red** (close < open) → **SELL** (short) at Friday's close.
- Exit both at **Monday's close**.

Only clean Thursday → Friday → Monday weeks are used. Weeks with a Friday or Monday market holiday are skipped.

**Trade return** = `direction × (Monday close / Friday close − 1)` in %, where direction is +1 for a buy and −1 for a sell.

---

## What the data shows (Nifty 50, 2000 to Oct 2026, 1,160 weeks)

**Does Friday's colour carry into Monday?**

| | Monday same colour |
|---|---|
| After a green Friday | 55.2% |
| After a red Friday | 54.2% |
| Baseline: Monday green (all Mondays) | 50.8% |

A chi-square test of Friday colour against Monday colour gives p = 0.0015. The link is real but small.

**The simple strategy (no filters)**

| | Trades | Win rate |
|---|---|---|
| Green Friday → BUY | 583 | 61.4% |
| Red Friday → SELL | 577 | 51.3% |
| Both sides | 1,160 | 56.4% |
| Benchmark: BUY every Friday | 1,160 | 55.1% |

The benchmark ignores colour entirely. Its win rate looks decent only because the market rises more often than it falls, so the colour-based trades should be judged against it, not against 50%.

**Factors that could improve the odds**

- **Candle body size.** Tiny candles (body under 0.25%) show almost no edge. Larger bodies follow through more often, though not in a smooth line.
- **Red Friday with volume > Thursday volume.** This looked strong in 2000-2012 (62.6% win rate) but was no better than baseline in 2013-2026 (50.0%). It has faded.
- **Green Friday with a big body (> 0.5%), a gap in Friday's direction, and a close above the 50-day average.** This combination won 72.5% in the first half and 74.5% in the second half (131 trades). It was picked after viewing both halves, so treat it as a hypothesis, not a proven rule.

**Filtered strategy (red volume rule + green combo)**

| | Trades | Win rate | Max drawdown |
|---|---|---|---|
| Base (all Fridays) | 1,160 | 56.4% | −24.3% |
| Filtered | 392 | 61.2% | −9.2% |

The filtered strategy's win rate was 66.8% in 2000-2012 and 56.1% in 2013-2026, so the edge has weakened in recent years.

**Five-year blocks vs Nifty buy-and-hold**

The notebook compares the strategy with Nifty buy-and-hold in 2005-10, 2010-15, 2015-20, 2020-25 and 2025-now. The filtered strategy was profitable in every block but beat buy-and-hold only in 2025-now. It had much smaller drawdowns and was invested only about 5-6% of trading days. See the notebook for the figures.

**Costs matter.** The edge per trade is small, so trading costs can erase much of it. The notebook has a cost sensitivity table, and `COST_PCT` lets you test your own costs.

---

## Files

| File | What it is |
|---|---|
| `friday_monday_strategy.ipynb` | Main notebook: the full analysis with explanations, charts and the strategy backtest |
| `red_friday_sell.py` | Standalone script for one rule only: SELL red Fridays where volume > Thursday volume |
| `red_friday_trades.csv` | Trade list produced by `red_friday_sell.py` |
| `NIFTY50.csv` | Input data (you supply this; not included in the outputs) |

### Notebook sections

1. Load and clean the data
2. Build features at Friday's close
3. Does a green Friday lead to a green Monday?
4. Strategy on the full data set, with a buy-every-Friday benchmark
5. Factors: correlations, quintiles, yes/no filters, combination search with a first-half / second-half check, and candle body size
6. The filtered strategy and a cost sensitivity table
7. Five-year blocks vs buy-and-hold
8. Summary of key numbers

---

## How to run

**Requirements:** Python 3.9+ with `pandas`, `numpy`, `scipy`, `matplotlib` and `jupyter`.

```bash
pip install pandas numpy scipy matplotlib jupyter
jupyter notebook friday_monday_strategy.ipynb
```

Put `NIFTY50.csv` in the same folder as the notebook, then run all cells.

**Expected CSV format** (newest-first is fine):

```
Date,Close,Open,High,Low,Volume
16-10-2026,"23,146.15","23,065.10","23,149.60","22,915.50",391.56M
```

Dates are `dd-mm-yyyy`. Prices may contain commas. Volume may end in `K`, `M` or `B`.

---

## Settings you can change

In the notebook:

| Setting | Where | Meaning |
|---|---|---|
| `COST_PCT` | Setup cell | Round-trip cost per trade in %. Starts at 0 |
| `SPLIT_YEAR` | Setup cell | Year that splits the discovery half (up to and including it) from the validation half |
| `GREEN_RULE`, `RED_RULE` | Section 5 | Lists of filter names applied to each side. An empty list means "take every trade" |
| `PERIODS` | Section 6 | The blocks used for the buy-and-hold comparison |

---

## Method notes and limits

- **Look-ahead:** every feature uses only information available at Friday's close.
- **Data snooping:** many factors and combinations were tested. The combination search ranks rules on the first half only and then reports the second half, which the search never saw. Selecting defaults after viewing both halves removes that protection, so the filtered results are optimistic.
- **Sample size:** strict filters leave few trades (tens to low hundreds), so small differences are unreliable.
- **Buy-and-hold** uses the price index, so dividends are not included.
- **Idle capital:** the strategy holds a position only from Friday close to Monday close, so compare per-trade edge and drawdown, not just total return.
- **Not tested:** real execution (futures or options costs, slippage, margin), and options pricing. For options, the premium, weekend time decay and bid-ask spread must all be beaten by a small index-level edge.
- **Red Friday volume rule:** it did not hold up after 2012 in this data.
