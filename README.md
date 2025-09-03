# TIBacktester
> `tibacktester` is originally a fork of Pybroker, 
> however our goal is to use the utils and libraries from pybroker to build a new backtesting engine,
> to support these main unique features besides the good existing features in pybroker (numba acceleration, machine learning strategy support, etc):

- Option Backtesting
- Portfolio Backtesting / Multi-Asset Backtesting / Pair Trading Backtesting 
- Strategy Optimization (through libraries and algorithms commonly used in machine learning and AI)
- Basic built-in strategies library (to contain a few common strategies to test and use out of the box)


## Why we create this new backtesting library?

Pybroker is a great backtesting library, it is fast and with machine learning strategy support by default, 
it has built-in mechanism to avoid forward looking bias, however, it has some limitations too.

1. When I tried to use pybroker to test a pair trading strategy, which it means we trade Asset A based on Asset B's signal,
 I found it is not possible to do that in pybroker, because pybroker does not support multi asset backtesting at a time, 
 this is a limitation for many backtesting libraries, however, it is common in real world trading to trade multiple assets at the same time.

2. Similar to many existing backtesting libraries, pybroker does not support option backtesting.


## Roadmap of TiBacktester


- [ ] New backtesting engine (to support multi asset backtesting)
- [ ] Charting library integration (to visualize the backtesting result)
  - [ ] Market Value
  - [ ] Drawdown
  - [ ] Trades, Long/Short Positions
  - [ ] Sharpe Ratio line
- [ ] Consider using EagerPy to improve the performance
- [ ] Strategy parameters optimization 
- [ ] Built-in strategies library
- [ ] Option backtesting
  - [ ] Fetch options data 
  - [ ] Option pricing models
  - [ ] Option Greeks calculation
  - [ ] Backtest Option Only Strategies
  - [ ] Backtest Stock + Option Strategies, such as covered call, protective put, etc.

> The idea is to use existing utils and functions from pybroker as much as possible to build this new backtesting engine.



> We haven't created the documentation yet, so please refer to the examples below or pybroker's documentation for now: https://pybroker.readthedocs.io/en/latest/, new documentation will be created when it is ready.


## Algorithmic Trading in Python with Machine Learning

Are you looking to enhance your trading strategies with the power of Python and
machine learning? Then you need to check out **TIBacktester**! This Python framework
is designed for developing algorithmic trading strategies, with a focus on
strategies that use machine learning. With TIBacktester, you can easily create and
fine-tune trading rules, build powerful models, and gain valuable insights into
your strategy’s performance.



## Key Features

- A super-fast backtesting engine built in [NumPy](https://numpy.org/) and accelerated with [Numba](https://numba.pydata.org/).
- The ability to create and execute trading rules and models across multiple instruments with ease.
- Access to historical data from [Alpaca](https://alpaca.markets/), [Yahoo Finance](https://finance.yahoo.com/), [AKShare](https://github.com/akfamily/akshare), or from [your own data provider](https://www.TIBacktester.com/en/latest/notebooks/7.%20Creating%20a%20Custom%20Data%20Source.html).
- The option to train and backtest models using [Walkforward Analysis](https://www.TIBacktester.com/en/latest/notebooks/6.%20Training%20a%20Model.html#Walkforward-Analysis), which simulates how the strategy would perform during actual trading.
- More reliable trading metrics that use randomized [bootstrapping](https://en.wikipedia.org/wiki/Bootstrapping_(statistics)) to provide more accurate results.
- Caching of downloaded data, indicators, and models to speed up your development process.
- Parallelized computations that enable faster performance.

With TIBacktester, you'll have all the tools you need to create winning trading
strategies backed by data and machine learning. Start using TIBacktester today and
take your trading to the next level!

## Installation

TIBacktester supports Python 3.9+ on Windows, Mac, and Linux. You can install
TIBacktester using `pip`:

```bash
   pip install -U tibacktester
```

Or you can clone the Git repository with:

```bash
   git clone https://github.com/Tradeinsight-info/TIBacktester
```

## A Quick Example

Get a glimpse of what backtesting with TIBacktester looks like with these code
snippets:

**Rule-based Strategy**:

```python
from tibacktester import Strategy, YFinance, highest

def exec_fn(ctx):
  # Get the rolling 10 day high.
  high_10d = ctx.indicator('high_10d')
  # Buy on a new 10 day high.
  if not ctx.long_pos() and high_10d[-1] > high_10d[-2]:
     ctx.buy_shares = 100
     # Hold the position for 5 days.
     ctx.hold_bars = 5
     # Set a stop loss of 2%.
     ctx.stop_loss_pct = 2

strategy = Strategy(YFinance(), start_date='1/1/2022', end_date='7/1/2022')
strategy.add_execution(
  exec_fn, ['AAPL', 'MSFT'], indicators=highest('high_10d', 'close', period=10))
# Run the backtest after 20 days have passed.
result = strategy.backtest(warmup=20)
```

**Model-based Strategy**:

```python
from tibacktester import Alpaca, Strategy, model

def train_fn(train_data, test_data, ticker):
  # Train the model using indicators stored in train_data.
  ...
  return trained_model

# Register the model and its training function with TIBacktester.
my_model = model('my_model', train_fn, indicators=[...])

def exec_fn(ctx):
  preds = ctx.preds('my_model')
  # Open a long position given my_model's latest prediction.
  if not ctx.long_pos() and preds[-1] > buy_threshold:
     ctx.buy_shares = 100
  # Close the long position given my_model's latest prediction.
  elif ctx.long_pos() and preds[-1] < sell_threshold:
     ctx.sell_all_shares()

alpaca = Alpaca(api_key=..., api_secret=...)
strategy = Strategy(alpaca, start_date='1/1/2022', end_date='7/1/2022')
strategy.add_execution(exec_fn, ['AAPL', 'MSFT'], models=my_model)
# Run Walkforward Analysis on 1 minute data using 5 windows with 50/50 train/test data.
result = strategy.walkforward(timeframe='1m', windows=5, train_size=0.5)
```


