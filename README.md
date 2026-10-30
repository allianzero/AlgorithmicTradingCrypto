# AlgorithmicTradingCrypto

Algorithmic crypto trading: strategies, backtesting, and risk. Algorithmic trading means using computer programs to execute trades according to predefined rules — instead of placing each trade by hand.

> This guide is educational only, not financial advice. Most algorithmic strategies fail in live markets, and a backtest that looks profitable often loses money in reality. Never run a bot with money you cannot afford to lose.

## 1. What algorithmic trading is

An algorithmic trading system has four parts:

1. **Data:** price, volume, order book, and on-chain data, arriving in real time.
2. **Signal:** a rule that decides when to trade. For example: "buy when the 50-period average crosses above the 200-period average" or "buy when price falls 5% below a moving average and volume spikes".
3. **Execution:** the code that places, manages, and cancels orders on an exchange, via its API.
4. **Risk controls:** position limits, stop-losses, maximum daily loss, and kill switches that stop the bot when something goes wrong.

The bot does not "understand" the market. It follows rules, tirelessly, without emotion — which is both its advantage (no panic, no FOMO) and its danger (it will follow a bad rule into a loss just as tirelessly).

## 2. Common strategy families

### Trend following
Buy assets in an uptrend, sell (or short) in a downtrend, using moving averages, breakouts, or momentum indicators.

- Works best in sustained trends. Fails repeatedly in sideways markets, where it buys highs and sells lows ("whipsaws").
- Crypto trends are strong but violent; trend systems need wide stops and small positions to survive reversals.

### Mean reversion
Buy when price falls unusually far below its recent average, expecting it to revert; sell when it rises unusually far above.

- Works best in ranging markets. Fails catastrophically in strong trends, where "oversold" keeps getting more oversold.
- Mean reversion in crypto is especially dangerous during crashes: buying every dip in a 70% fall is a fast way to lose everything.

### Arbitrage
Exploit price differences for the same asset across exchanges, or between spot and derivatives markets.

- Conceptually low-risk, but in practice dominated by firms with faster infrastructure, lower fees, and co-located servers. Retail arbitrage edges are usually tiny after fees, and disappear quickly.
- Cross-exchange arbitrage adds transfer times and withdrawal risks: by the time your coins arrive on the cheaper exchange, the price gap may be gone.

### Market making
Place buy and sell orders on both sides of the market, profiting from the spread.

- Earns small, frequent profits in calm markets. Suffers large losses in violent moves, when the bot ends up holding the falling side (adverse selection).
- Requires careful inventory management: a market maker that accumulates too much of a falling asset is just a bag holder with extra steps.

### Momentum and breakout
Buy assets that are rising fastest, or that break above resistance, expecting continuation.

- Simple and popular, but crowded: many bots chase the same breakouts, creating false breakouts designed to trap them.
- Needs strict exit rules, because momentum reversals are sudden.

**Honest summary:** no strategy family works in all markets. Every strategy has a market regime where it loses, and the skill is recognising that regime and stopping — not finding a strategy that never loses.

## 3. Backtesting: how strategies are tested — and fooled

**Backtesting** means running a strategy on historical data to see how it would have performed.

A valid backtest must include:

- **Realistic fees:** exchange fees, spreads, and slippage, on every trade. Ignoring fees turns many losing strategies into apparent winners.
- **Realistic execution:** you cannot assume you always traded at the closing price, or that your orders always filled. Large orders move prices.
- **Out-of-sample testing:** test on data the strategy was not designed on. A strategy tuned to perform perfectly on 2021 data and tested only on 2021 data proves nothing.
- **Multiple market regimes:** bull, bear, sideways, crash, and low-liquidity periods. A strategy that only works in bull markets is not a strategy — it is a bull market.

### Overfitting: the central trap
**Overfitting** means tuning a strategy so precisely to historical data that it captures noise, not real patterns. Symptoms:

- Dozens of parameters, each finely tuned.
- Spectacular backtest returns that collapse on new data.
- A strategy that works on one coin and one period, and nowhere else.

**Defences:** keep parameters few and robust (a strategy that works across a range of parameter values is more trustworthy than one that works only at one exact value), test out-of-sample, test on multiple assets, and distrust any backtest that looks too good.

### Survivorship bias
Testing only on coins that still exist today ignores the coins that died. A strategy that "would have" picked winners among today's survivors looks better than it was, because the losers were removed from the dataset. Include delisted and dead coins in backtests where possible.

### Look-ahead bias
Using information in a backtest that would not have been available at the time of the trade — for example, using a day's closing price to decide a trade supposedly placed at that day's open. Look-ahead bias creates impossible profits and is embarrassingly common in beginner backtests.

## 4. From backtest to live trading: why live results differ

Even a honest backtest overstates live performance, because live trading adds:

- **Slippage:** your orders move the market, especially in thin pairs.
- **Latency:** data arrives, your code decides, your order travels to the exchange, and the market moves in between. In fast markets, milliseconds matter, and retail infrastructure is slower than professional infrastructure.
- **Fees and funding:** derivatives funding, withdrawal fees, and exchange fee tiers change with volume and market conditions.
- **API failures and rate limits:** exchanges throttle API calls, go down during volatility, or reject orders. A bot without handling for these will behave unpredictably exactly when it matters most.
- **Market impact of other bots:** your strategy competes with thousands of similar bots, arbitraging away the same edges.

**Rule:** paper-trade (run the bot on live data without real money) for at least a month, through varied market conditions, before risking real funds. Paper trading still overstates results (no slippage, no emotion, no API failures under real stakes), but it catches the most obvious failures cheaply.

## 5. Infrastructure and risk controls

A production bot needs more than a strategy:

- **API keys with least privilege:** trading-only keys, with withdrawals disabled, IP restrictions where the exchange supports them, and keys stored securely (never in code repositories or chat messages).
- **Kill switch:** a manual and automatic way to stop all trading instantly — on maximum daily loss, on API errors, on unusual positions, or on your command.
- **Position and exposure limits:** maximum per trade, per asset, and total, enforced by code, not by intention.
- **Logging and alerts:** every order, fill, error, and decision logged with timestamps, and alerts for anomalies (large losses, failed orders, unexpected positions).
- **Redundancy and monitoring:** what happens if your server crashes, your internet drops, or the exchange goes down mid-position? A bot that cannot handle downtime is a liability, not an asset.
- **Separate funds:** bot funds in a separate exchange sub-account, with only the amount the bot is allowed to risk. Never give a bot access to your main holdings.

## 6. Common beginner mistakes

| Mistake | Better approach |
|---|---|
| Trusting a backtest without out-of-sample testing | Out-of-sample, multi-asset, multi-regime testing, with fees and slippage |
| Overfitting with many parameters | Few, robust parameters; distrust spectacular backtests |
| Running a bot with full account access | Trading-only API keys, withdrawals disabled, separate sub-account |
| No kill switch or daily loss limit | Hard-coded limits, tested by triggering them deliberately |
| Copying a "profitable bot" from the internet | A bot you did not understand, test, and risk-manage is someone else's risk, not your strategy |
| Ignoring fees, funding, and taxes | Count all costs; derivatives funding and tax on frequent trades can erase apparent profits |
| Letting a bot run unattended for months | Regular reviews, strategy health checks, and regime awareness — bots need supervision, not abandonment |

## 7. A realistic learning path

1. **Learn manual trading first.** If you cannot trade profitably (or at least deliberately) by hand, automating your decisions automates your mistakes faster.
2. **Learn a programming language and data analysis** (Python is the standard choice, with libraries for data, backtesting, and exchange APIs).
3. **Build a backtester, and break it.** Deliberately introduce look-ahead bias, ignore fees, and overfit — then see how spectacular and false the results look. This lesson is worth more than any profitable backtest.
4. **Backtest honestly**, with fees, slippage, out-of-sample data, and multiple regimes.
5. **Paper-trade for a month**, with full logging and risk controls.
6. **Trade tiny real amounts**, with a kill switch and daily loss limit, and compare live results to backtest and paper results. Expect live to be worse. If it is much worse, stop and diagnose before scaling.
7. **Scale only on evidence**, not on hope: consistent, disciplined execution over months, not a lucky week.

## 8. Frequently asked questions

**Can I buy a profitable trading bot?**
Almost certainly not. A genuinely profitable bot is a valuable asset its owner would run themselves, not sell for a one-time fee. Bots sold online are usually backtest-optimised illusions, or outright scams that steal API keys or funds.

**Do I need fast hardware or a VPS?**
For most retail strategies (hourly or daily signals), a normal computer or a cheap VPS is enough. For high-frequency strategies competing on milliseconds, retail hardware cannot compete with professional infrastructure, and you should not attempt those strategies.

**Is algorithmic trading illegal or against exchange rules?**
Algorithmic trading via official APIs is allowed by most exchanges, within rate limits and market rules. Wash trading, spoofing, and market manipulation via bots are illegal in many jurisdictions and banned by exchanges. Automating a strategy does not exempt it from market abuse rules.

**How much capital do I need to start algorithmic trading?**
Enough that fees do not dominate returns, but small enough that a total loss is survivable. For learning, paper trading costs nothing and teaches most of the lessons. For live learning, the smallest amount your exchange allows, with strict risk limits.

> Educational content only. This is not financial, investment, or tax advice.
