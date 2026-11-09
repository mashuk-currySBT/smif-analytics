# Curry College SMIF – Analytics Notebook

This project lets us study our Student Managed Investment Fund (SMIF) with real data, without updating Excel by hand. You press **Play** on a series of steps. The computer downloads current prices, runs the models we discussed in class, and shows tables and charts. You do **not** need to know how to program.

> **Important:** these results are a *teaching tool* and one input into our decisions. They use past data. Past performance does not predict future returns, and nothing here is a recommendation to buy or sell.

---

## What is in this project

| File | What it is | Do I edit it? |
|---|---|---|
| `SMIF_Analytics_Notebook.ipynb` | The step-by-step page you work in. | Only the cells marked ✏️ **EDIT ME** |
| `smif_models.py` | The "engine" with all the calculations. The notebook creates this file for you in Step 2. | No (not yet) |
| `SMIF_Portfolio_V1.xlsx` | Our portfolio snapshot with top-10 holdings of each ETF. | No |
| `data/prices.csv` | Daily prices the notebook downloads in Step 4. Appears after you run Step 4. | No |

---

## Part 1 – First-time setup (GitHub Codespaces)

A **Codespace** is a computer that lives on the internet and opens in your browser, so nothing is installed on your own laptop.

1. Sign in at **github.com** with your own GitHub account (free).
2. Open the SMIF repository link from your instructor and click **Fork** (top right) → **Create fork**. This gives you your own copy, so your edits don't change anyone else's.
3. In *your* copy, click the green **Code** button → **Codespaces** tab → **Create codespace on main**.
4. Wait 1–2 minutes while it opens. You'll see a code editor with a file list on the left.
5. Click `SMIF_Analytics_Notebook.ipynb` in the file list to open it.
6. If a pop-up asks to install the **Python** or **Jupyter** extensions, click **Install**, then re-open the notebook.
7. At the top right of the notebook, click **Select Kernel** → **Python Environments** → pick the recommended **Python 3** option.

**What the "kernel" is:** it's the part that actually runs the code when you press Play. Without choosing one, Play does nothing.

**Coming back later:** Codespaces stops itself after about 30 minutes idle. Your files are kept. To resume, go to github.com/codespaces and click your Codespace. The first cell you run may take a few seconds to wake up.

---

## Part 2 – How the notebook works

The notebook is made of **cells**. A *text cell* is an explanation (just read it). A *code cell* has a grey box with a ▶ **Play** button on the left.

**Rules:**
- Run cells **in order, top to bottom**. Each step depends on the one before.
- Wait for the cell to finish. A spinner means it is still working. A ✓ means it is done.
- If something turns red, don't panic. See **Part 5 – Troubleshooting**.

---

## Part 3 – Step by step: exactly what happens

### Step 1 – Install tools
```
!pip -q install scipy pandas numpy matplotlib yfinance openpyxl
```
Python comes with the basics, and these are add-on tool packs we need. `pip` is the program that downloads them. What each one is for:

- **pandas**: tables of data (like Excel in code).
- **numpy**: fast math on lists of numbers.
- **scipy**: the optimizer. It is the same kind of search as Excel Solver.
- **matplotlib**: charts.
- **yfinance**: downloads prices from Yahoo Finance.
- **openpyxl**: reads `.xlsx` Excel files.

`-q` means "quiet", so there is almost no output. A ✓ means success.

### Step 2 – Save the SMIF toolkit
The first line `%%writefile smif_models.py` tells the notebook to save everything below it as a file named `smif_models.py`. It prints "Writing smif_models.py" and nothing else. That file holds every calculation in this project. Think of it as the engine. The later steps are the steering wheel.

### Step 3 – ✏️ Tell it what we own and what we are researching
```python
import smif_models as sm
sm.CURRENT = {"XLE": 31*62.21, ...}
sm.CANDIDATES = {"XLU": "Utilities", ...}
sm.MAX_W = 0.35
sm.ADD_SIZE = 0.10
```
- `import smif_models as sm` loads the engine and nicknames it `sm`.
- `CURRENT` is our portfolio: each ETF's ticker and the dollars we hold (shares × price). The notebook turns dollars into percentage weights for you.
- `CANDIDATES` is the list of *new* ETFs being researched, with a label. **To add an ETF, copy one line, change the ticker and the label, and keep the commas and quotation marks.**
- `MAX_W = 0.35` means the optimizer may not put more than 35% in any single ETF.
- `ADD_SIZE = 0.10` means that when we test a candidate, we pretend to put 10% of the portfolio into it.

Try changing `MAX_W` to `0.25`, then rerun Steps 7–9 and see how the answers change.

### Step 4 – Download prices
```python
sm.update()
px = sm.load()
```
- Downloads **daily prices from July 2021 to today** for our 6 ETFs, the 7 candidates, and some extra ETFs used for factors. `px` is the name we give the resulting table.
- Prices are **adjusted** for dividends and splits, so they reflect what an investor actually earned (total return).
- Also downloads the **3-month Treasury bill rate** from the Federal Reserve (FRED). That's our "risk-free rate", the return you could earn with almost no risk.
- Saves everything to `data/prices.csv` and shows the most recent rows.

Run this step again any time you want fresh data.

### Step 5 – Performance table and chart
The **chart** shows what $1 invested in July 2021 would be worth for each ETF. The **table** (`sm.stats`) has one row per ETF:

| Column | Meaning |
|---|---|
| `ann_return` | Average yearly return over the period |
| `vol` | Volatility: how much the returns bounce around, per year (risk) |
| `sharpe` | (Return − risk-free rate) ÷ volatility. Return per unit of risk. Higher is better. |
| `beta` | How much it moves when the overall market (VTI) moves. 1.0 = same as the market. 1.3 = 30% more. 0.6 = less. |
| `capm_return` | The return the CAPM says the ETF *should* have earned given its beta: risk-free rate + beta × (market return) |
| `alpha` | Actual return minus CAPM return. Positive means it beat what its risk "should" earn. |
| `alpha_tstat` | How confident we can be that the alpha is real and not luck. **Above 2 (or below −2) is meaningful. Under 2 is probably noise.** |
| `max_drawdown` | The worst peak-to-trough fall. −0.35 means it once fell 35% from a high. |

**How it's calculated:** daily returns are the change in log price from one day to the next. Yearly return and volatility are scaled up from daily (×252 and ×√252, because there are about 252 trading days per year). Beta comes from a regression of the ETF's excess returns on the market's excess returns. This is the same math as the Excel Sharpe optimization file.

**Think about:** Which ETFs have the best Sharpe? Does a high alpha come with an alpha_tstat above 2, or is it probably luck?

### Step 6 – Correlation heatmap
Correlation measures how closely two ETFs move together day by day. 1.00 means identical movement and 0 means unrelated. **Diversification only helps when correlations are low.** If two ETFs have correlation 0.95, owning both is almost the same as owning one.

**Think about:** Which pairs are above 0.9? (We expect VGT and VOO to be high, because big tech is a large part of the S&P 500.) Which new candidate looks most different from what we own?

### Step 7 – Suggested weights (`sm.optimize`)
The computer searches for the mix of our 6 ETFs that best meets a goal. Every method is long-only (no shorting), sums to 100%, and obeys `MAX_W`.

| Method | Goal |
|---|---|
| `max_sharpe` | Highest Sharpe ratio (what we did in Excel with Solver) |
| `min_variance` | Lowest possible volatility |
| `risk_parity` | Each ETF contributes the same amount of *risk* |
| `equal_weight` | Same dollars in every ETF |
| `CURRENT` | What we actually own, for comparison |

The first table shows the weights. The second shows each mix's return, risk and Sharpe **in-sample**, meaning on the same data used to find the weights. In-sample results are flattering, because the optimizer picked the weights *knowing* what already happened.

The third table is the honest test, a **walk-forward test**. Every 3 months it finds the best weights using only the previous 2 years, holds them for the next 3 months on data it has never seen, then repeats. The `oos_` columns are the results on that unseen data ("oos" = out of sample).

**Think about:** Which method did best out of sample? Why might max-Sharpe look great in-sample but disappoint later? (Hint: it chases whatever happened to win in the past.)

### Step 8 – Which new ETF helps most? (`sm.candidates`)
For each candidate we pretend to move 10% of the portfolio into it (taking a bit from every holding in proportion) and measure what changes.

| Column | Meaning |
|---|---|
| `corr_w_portfolio` | Correlation of the candidate with our current portfolio. Lower = more diversifying. |
| `most_similar` / `max_corr` | Which single holding it most resembles, and how closely |
| `R2_by_holdings` | The share of the candidate's movement that our existing holdings already explain. **Low means it gives us something new.** |
| `d_vol` | Change in portfolio risk if we add it. **Negative = it lowers risk.** |
| `d_exp_ret` | Change in historical average return |
| `d_sharpe` | Change in Sharpe ratio |

**Caution:** historical returns are noisy, so `d_exp_ret` and `d_sharpe` mostly tell you what did well recently. `d_vol`, the correlations and `R2_by_holdings` are the more trustworthy columns for judging diversification.

**Think about:** Does SMH add anything beyond VGT? Does XLU or XLP lower risk?

### Step 9 – Factor exposure (`sm.factors`)
A *factor* is a characteristic that drives returns, such as company size or style. We build simple factors from ETFs that capture them:

| Factor | Built as | In plain words |
|---|---|---|
| `MKT` | VTI − risk-free rate | The overall market |
| `SIZE` | IWM − VOO | Small companies vs large |
| `VALUE` | VTV − VUG | Cheap "value" stocks vs "growth" stocks |
| `MOM` | MTUM − VTI | Recent winners vs the market |
| `QUALITY` | QUAL − VTI | Profitable, stable companies |
| `LOWVOL` | USMV − VTI | Low-volatility stocks |

For each ETF the table shows `b_` (exposure to a factor) and `t_` (confidence; above 2 or below −2 is meaningful). `R2` is the share of the ETF's movement the factors explain, and `alpha_ann` is the return left over after accounting for those factors.

**Think about:** What does the table say each sector ETF is really a bet on? Does VGT behave more like "growth" or "momentum"?

### Step 10 – Top-10 holdings vs the rest (`sm.holdings`)
Reads `SMIF_Portfolio_V1.xlsx`, which lists the **top 10 stock holdings inside each of our ETFs**. Because the ETFs overlap, the same stock can appear more than once, and the file combines them. The step shows:

1. **Each stock's contribution** in basis points (1 bp = 0.01% of the whole portfolio): how many hundredths of a percent it added or took away.
2. **Top-10 basket vs everything else:** the return of those top stocks compared with the other roughly 40% of what we own.
3. **Contribution by ETF.**

**Note:** this compares the two dates in the Excel file (the trade date and the update date). It is a snapshot, not a long history, so don't read too much into a short period.

Make sure the file name in the quotation marks matches the name in the file list on the left exactly.

### Step 11 – Save the data
In Codespaces, right-click `data/prices.csv` in the file list → **Download**.

---

## Part 4 – Glossary

- **ETF:** a fund that holds a basket of investments and trades like a stock.
- **Return:** how much an investment gained or lost.
- **Volatility (standard deviation):** how much the returns swing up and down. It is our measure of risk.
- **Sharpe ratio:** extra return above the risk-free rate, per unit of volatility.
- **Risk-free rate:** return on a 3-month U.S. Treasury bill.
- **Beta:** sensitivity to the overall market.
- **CAPM:** a model saying expected return = risk-free rate + beta × (market return − risk-free rate).
- **Alpha:** actual return minus the CAPM expectation.
- **Correlation:** how closely two investments move together (−1 to +1).
- **Diversification:** combining investments that don't all move together, to reduce risk.
- **Drawdown:** a fall from a previous high.
- **Minimum variance portfolio:** the mix with the lowest volatility.
- **In-sample / out-of-sample:** the data used to build a model vs. data it hasn't seen.
- **Basis point (bp):** one-hundredth of one percent.

---

## Part 5 – Troubleshooting

| What you see | What to do |
|---|---|
| `No module named 'scipy'` (or `pandas`, `yfinance`...) | Run Step 1 again. If it still fails, click **Restart** at the top of the notebook, then run Steps 1, 2 and 3 again. |
| `No module named 'smif_models'` | Step 2 did not save the file. Run Step 2, then Step 3. |
| `NameError: name 'sm' is not defined` | Run Step 3 first. |
| `NameError: name 'px' is not defined` | Run Step 4 first. |
| Step 4 returns an empty table or errors | Yahoo may be busy. Wait a minute and run Step 4 again. |
| `FileNotFoundError` in Step 10 | The file name is wrong. Copy the exact name from the file list. |
| `No module named 'google.colab'` | That cell is for Google Colab. In Codespaces, skip it. |
| Nothing happens when you press Play | Click **Select Kernel** at the top right and choose Python 3. |
| Everything looks broken | Click **Restart**, then run the cells from Step 1 down in order. |

When asking for help, send a screenshot and the **last line** of the red error message.

---

## Part 6 – Practice ideas (in order of difficulty)

1. Run all steps and write down the ETF with the best Sharpe, and the best and worst alpha_tstat.
2. In Step 3, change `MAX_W` to 0.25, then 0.50. How do the optimal weights change?
3. In Step 3, add a new candidate ETF that was not on the list. Rerun Steps 4 and 8.
4. Change `ADD_SIZE` to 0.05, then 0.20. Does the ranking of candidates change?
5. For your assigned ETF, write a one-page recommendation using **the numbers from Steps 5–9 plus your own sector research**: buy more, hold, reduce or sell.

---

*Questions? Ask your instructor. Don't make trades based on this notebook alone.*
