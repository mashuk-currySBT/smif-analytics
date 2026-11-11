# Curry College SMIF – Analytics Project

This project lets us study our Student Managed Investment Fund with real data instead of updating Excel by hand. You press **Play** on a series of steps; the computer downloads prices, runs the models from class, and draws the charts. **You do not need to know how to program.**

> The results are a *learning tool* and one input into our decisions. They use past data. Past performance does not predict future returns, and nothing here is a recommendation to buy or sell.

---

## 1. What is in this folder

| File | What it is | Do I edit it? |
|---|---|---|
| `SMIF_Analytics_Notebook.ipynb` | The step-by-step page you work in (modules 0–7) | Only the cells marked ✏️ |
| `smif_models.py` | The "engine" with all the calculations | No |
| `data/trades.csv` | What we bought: ETF, shares, price paid | Yes, after a new trade |
| `data/holdings.csv` | Top holdings of each ETF with their weights inside the ETF and an as-of date | Via `sm.add_holdings(...)` |
| `SMIF_Portfolio_V1.xlsx` | The portfolio sheet with the top-10 stocks of each ETF and their 10/19 prices (used by Module 2) | No |
| `data/prices.csv` | Prices the notebook downloads (appears after Module 0c) | No |
| `requirements.txt` | The list of add-on packages | No |

---

## 2. First-time setup in GitHub Codespaces

A **Codespace** is a computer that lives on the internet and opens in your browser. Nothing gets installed on your laptop.

1. Sign in at **github.com** (free account).
2. Open the SMIF repository link from your instructor and click **Fork** (top right) → **Create fork**. You now have your own copy, so your edits do not change anyone else's.
3. In *your* copy click the green **Code** button → **Codespaces** tab → **Create codespace on main**.
4. Wait 1–2 minutes. You will see an editor with a file list on the left.
5. Click `SMIF_Analytics_Notebook.ipynb` in the file list. If a pop-up offers to install the **Python** and **Jupyter** extensions, click **Install**, then reopen the notebook.
6. Top right of the notebook: **Select Kernel → Python Environments →** the recommended Python 3. *(The kernel is the part that runs the code when you press Play.)*
7. Run **Module 0a** (the `%pip install` cell). It takes about 30 seconds.

**Coming back later:** Codespaces stops itself after ~30 minutes idle but keeps your files. Resume at github.com/codespaces. If anything looks stuck, click **Restart** in the notebook toolbar and run from the top.

> Using Google Colab instead? Upload every file in this folder (keeping the `data` folder) to the Colab file browser; the notebook works the same way.

---

## 3. How to run the notebook

* Run cells **in order**, top to bottom. Wait for the ✓ before the next one.
* A cell with a ✏️ is one you may edit. All others: just press Play.
* Output (tables and charts) appears under each cell.
* Red text is an error. See Troubleshooting below.

---

## 4. What to input where

| I want to… | Where | What to type |
|---|---|---|
| Change the **start/end date** of the models | Module 0b, Settings cell | `sm.START = "2023-01-01"` and `sm.END = "2026-10-26"` (`None` = latest). Then run that cell and `px = sm.load()` in 0d |
| Match the original Excel file | Module 0b | `START = "2021-07-01"`, `END = "2026-10-26"` |
| Limit how much the optimizer can put in one ETF | Module 0b | `sm.MAX_W = 0.25` |
| Test a different amount moved into a candidate | Module 0b | `sm.ADD_SIZE = 0.05` |
| Record a **new trade** | `data/trades.csv` | ETF, shares, price paid; and `sm.TRADE_DATE` in 0b |
| Research a **new ETF** | Module 0b `sm.CANDIDATES`, Module 5g | Add the ticker, add its top holdings with `sm.add_holdings(...)`, run `sm.update()` |
| Test a **proposed portfolio** | Module 5c | `sm.look_through({"XLE": .15, "VOO": .30, ...})` |
| Look at a different ETF vs its top stocks | Module 5d | Change `"VHT"` and `n` |
| Add an **event** | Module 6 | `groups["My event"] = ["YYYY-MM-DD"]` |
| Get **fresher prices** | Module 0c | Run `sm.update()` again, then `px_all = sm.load(full=True)` and `px = sm.load()` |

**All dates use the format `"YYYY-MM-DD"` with quotation marks.**

### Adding a new ETF's holdings (Module 5g), step by step
1. Google the ETF's ticker and open the fund company's page → **Holdings** (Vanguard calls it **Portfolio & management**).
2. Copy the top 5–10 stocks: ticker and **weight (%)**, plus the **"as of" date** shown on the page.
3. Fill in the example in 5g (remove the `#` at the start of each line) and press Play. Weights are the % **inside the ETF** and must add up to 100 or less.
4. Add the ETF to `sm.CANDIDATES` in Module 0b, run `sm.update()`, then rerun Modules 4 and 5.

---

## 5. The modules: what happens and what you learn

**Module 0 – Setup.** Installs packages, sets dates, downloads daily prices back to 2000 for all ETFs (adjusted for dividends and splits, so they are total returns) and since 2023 for individual stocks, plus the 3-month Treasury bill rate (our "risk-free" rate) from the Federal Reserve. `px_all` is everything; `px` is only your START–END window.

**Module 1 – Our portfolio since 10/19.** Reads `data/trades.csv` and shows each ETF's shares, price paid, latest close, weight at purchase vs now, % return, dollar profit or loss, and **contribution in basis points**. Charts: portfolio value vs cost, each ETF's return, how weights drift, contributions. A live-price check (can be ~15 minutes delayed) is included.

**Module 2 – Top 10 vs the rest.** Compares the top-10 stocks of each ETF (from the Excel file) with everything else in the ETFs, day by day since 10/19, and ranks the stocks that helped or hurt most. Because it uses holdings we actually had on 10/19 over a short period, there is no look-ahead problem.

**Module 3 – The Excel optimization in Python.**
* *3a* annualized return, volatility, **Sharpe ratio**, **beta** (vs VTI), **CAPM** expected return, **alpha** (actual − CAPM) and its t-stat.
* *3b* correlation heatmap.
* *3c* **max-Sharpe, minimum-variance, risk-parity, equal-weight** portfolios and the **efficient frontier** (the best return for each level of risk), with the capital allocation line.
* *3d* what we chose vs what the models said (our choice also reflects sector research).
* *3e* **out-of-sample test**: pick weights using only the previous 2 years, hold for 3 months, repeat. This shows how much optimizers overfit the past.

**Module 4 – With the seven new ETFs.** The same steps on all 13, plus a before/after frontier chart and a **candidate screen**: what happens to portfolio risk if 10% moves into each candidate.

**Module 5 – Top holdings and overlap.** Top 5 of each ETF, an **overlap matrix** (how much of two ETFs is the same stocks), our **look-through** exposure (e.g. NVDA through VOO and VGT combined), one **ETF vs its top N stocks**, every ETF at once, and our **whole portfolio vs the top-5 stocks of each of our ETFs**.

**Module 6 – Event studies.** Market-model event studies around midterm elections (the day after), Fed rate hikes, and oil price spikes. Shows which sectors earned **abnormal returns** around the event.

**Module 7 – Factor exposure.** Regresses each ETF on market, size, value, momentum, quality and low-volatility factors built from ETFs.

---

## 6. After a new trade

1. Add the new ETF/share lines to `data/trades.csv` (ETF, shares, price actually paid).
2. If the trade date changed, update `sm.TRADE_DATE` in Module 0b. *(This version measures performance from a single trade date. Ask your instructor if you need several dates at once.)*
3. Add holdings for any new ETF (Module 5g) and add it to `sm.CANDIDATES` or `sm.OWNED` if it is now a holding.
4. Run `sm.update()` and rerun the modules.
5. Click the Source Control icon (left bar) → **Commit** → **Sync Changes** to save your changes to GitHub.

---

## 7. Troubleshooting

| What you see | What to do |
|---|---|
| `No module named 'scipy'` (or pandas, yfinance, …) | Run Module 0a again. If it persists, click **Restart**, then run 0a and 0b |
| `No module named 'smif_models'` | The notebook must be in the same folder as `smif_models.py`. Open it from the repository's main folder |
| `NameError: name 'sm' is not defined` or `'px' is not defined` | Run Module 0b (and 0d) first |
| `No data yet. Run sm.update() first` | Run Module 0c |
| `Only N days between START and END` | The window is too short or outside the data. Widen `sm.START` / `sm.END`, run 0b and 0d |
| `Only N common days ... starts` | One ETF (named in the message) started trading recently. Move `START` later or remove it |
| `No price data for [...]` | Yahoo has no data for that ticker. Check the spelling (class shares use a dash: `BRK-B`) or try `sm.update()` again later |
| Download seems stuck | Yahoo can be slow. Wait 2–3 minutes, then run `sm.update()` again |
| Changed `smif_models.py` but nothing changed | Click **Restart** (Python remembers the old version until you restart) |
| Charts do not appear | Make sure you selected the Python kernel (top right), then rerun the cell |
| Everything looks broken | **Restart**, then run cells in order from Module 0a |

When asking for help, send a screenshot and the **last line** of the red error message.

---

## 8. Limits to keep in mind

* **Past data only.** Averages from a few years are noisy. That is why almost no alpha t-stat is above 2, and why max-Sharpe weights do poorly out of sample.
* **Short windows prove nothing.** Since 10/19 is only a few weeks.
* **Top-N vs ETF (Module 5d–f) has look-ahead bias:** it uses today's top holdings on past prices. Use it to study concentration, not to forecast.
* **Holdings are a snapshot** with different as-of dates by fund (Vanguard funds 9/30/2026, others early November). Stocks not on our lists are in the `REST` row; overlap numbers are minimums.
* **Yahoo Finance data is free but unofficial.** Spot-check an important price against our brokerage before trading.
* **Event studies** use daily data, so they cannot separate things that happen within one day. Always verify the event dates.

---

## 9. Glossary

* **ETF** – a fund that holds a basket of investments and trades like a stock.
* **Volatility** – how much returns swing; our measure of risk.
* **Sharpe ratio** – (return − risk-free rate) ÷ volatility; return per unit of risk.
* **Beta** – sensitivity to the overall market (1.0 = same as the market).
* **CAPM** – expected return = risk-free rate + beta × (market return − risk-free rate).
* **Alpha** – actual return minus the CAPM expectation. **t-stat** above 2 = unlikely to be luck.
* **Correlation** – how closely two investments move together (−1 to +1).
* **Efficient frontier** – the best possible return for each level of risk.
* **Minimum-variance portfolio** – the mix with the lowest volatility.
* **Out-of-sample** – data the model has not seen; the honest test.
* **Look-through** – adding up the stocks inside all our ETFs to see our true exposure.
* **Abnormal return (AR) / CAR** – return beyond what the market model predicts / its running total.
* **Basis point (bp)** – one-hundredth of one percent.

---

*Questions? Ask your instructor. Do not trade on this notebook alone.*
