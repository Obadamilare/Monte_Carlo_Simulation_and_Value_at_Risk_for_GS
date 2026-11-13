# Monte Carlo Simulation and Value at Risk for Goldman Sachs (GS)

A Python project that estimates how much one share of Goldman Sachs stock could lose over the next 30 trading days. It uses 3 years of historical prices to simulate 10,000 possible price paths, then measures Value at Risk (VaR) from the results.

This is an educational project. It is not financial advice.

## The question

Nobody can know what a stock price will be next month, but investors and banks still need to know how much they could lose. This project asks one question: given how GS has behaved in the past, how bad could the next 30 trading days realistically get?

## Simulated price paths



![Simulated GS price paths](simulated_paths.png)



Each line is one possible 30 day journey of the GS share price. The fan widens with time because uncertainty grows the further ahead we look. These are possibilities, not forecasts.

## Data

* Source: Yahoo Finance, downloaded with the yfinance library
* Ticker: GS, daily prices, downloaded with period "3y"
* gs_historical.csv is the raw download: 753 trading days, from 2023-11-06 to 2026-11-05
* gs_clean.csv is the cleaned dataset used for the analysis: 744 trading days, from 2023-11-06 to 2026-10-23

## Method

1. **Load and clean.** The download came with two levels of column labels, so I removed the ticker level to give each column one name. I checked for missing values, duplicate dates and date order, and found none. I did not fill anything, because weekends and holidays are market closures, not missing data. Rows dated on or after 2026-10-26 were removed, including the last row, which was an unfinished trading day. Four sanity checks passed: high is never below low, close sits between low and high, prices are positive, and volume is positive.
2. **Daily log returns.** For each day, the log return is ln(today's close divided by yesterday's close). This gives 743 returns.
3. **Average and volatility.** The average daily log return and its standard deviation (the volatility) are calculated from the 743 returns.
4. **Simulation.** NumPy generates a 30 by 10,000 grid of random numbers from the standard normal curve. Each is scaled by the volatility, and the average return is added. The daily returns are added up along each path and converted to prices with the exponential function, starting from the last closing price.
5. **Value at Risk.** The 30 day change in price is calculated for all 10,000 paths. The 5th percentile of those changes gives the 95% VaR.

## Modelling choices

* Log returns are used because they add up across days.
* Daily returns are drawn from a normal distribution.
* The drift is the average log return used directly. It already contains the volatility drag, so no extra correction is applied.
* The random seed is fixed at 42, so the results can be reproduced.
* Loss convention: one share held for 30 trading days. Loss is the start price minus the final price, reported as a positive number.

## Results so far

* Cleaned data: 744 trading days, 743 daily log returns
* Average daily log return: 0.001429
* Daily volatility: about 0.0186, which is about 29.60% per year
* Worst day: 2025-04-03, a fall of 9.66%
* Best day: 2024-11-06, a rise of 12.31%
* Starting price: $875.68, the close on 2026-10-23
* Simulation: 10,000 paths of 30 trading days, seed 42
* 95% VaR over 30 trading days: $103.60 per share, about 11.8% of the starting price

How to read the VaR: in 95% of the simulated outcomes, the loss on one share after 30 trading days was no more than about $103.60. In the other 5%, the loss was larger.

## Limitations

* The model assumes daily returns follow a normal curve. The worst day in the data was more than 5 standard deviations below the average, which a normal curve says should almost never happen. Real returns have heavier tails than the model assumes.
* The average and volatility are assumed to stay constant, and each day is assumed to be independent of the others. Real markets can break both assumptions.
* The average return comes from a period in which GS rose strongly, from about $303 to about $876. If future growth is weaker, the real risk is higher than the simulation shows.
* Simulated paths show what could happen under the model's assumptions. They do not predict what will happen.
* The results apply to one share. They do not cover a portfolio.

## Project status

Done:
* Data download, cleaning and validation
* Log returns, average return and volatility
* 10,000 path Monte Carlo simulation
* Fan chart of simulated paths
* 95% Value at Risk

In progress:
* Expected Shortfall (the average loss in the worst 5% of outcomes)
* Histogram of final prices with the VaR line
* Comparison of the simulated VaR with real historical 30 day losses

## Possible extensions

* Use a distribution with heavier tails, or a historical simulation, and compare the results
* Let volatility change over time
* Test whether the returns really follow a normal curve
* Count how often real losses exceed the VaR, as a proper backtest
* Extend the project to a portfolio of several stocks
* Try other horizons and a 99% confidence level

## Repository contents

* Monte_Carlo_Simulation_and_Value_at_Risk.ipynb: the full notebook
* gs_historical.csv: the raw Yahoo Finance download
* gs_clean.csv: the cleaned dataset
* images: saved charts

## How to run

Install the libraries:

```
pip install yfinance pandas numpy matplotlib jupyter
```

Open the notebook in Jupyter and run the cells from top to bottom.

The notebook downloads the last 3 years of data from today, so a fresh run will use a slightly different window and give slightly different numbers. To reproduce the results above exactly, load the saved file instead of downloading:

```
df = pd.read_csv("gs_clean.csv", index_col=0, parse_dates=True)
```

## Author

Adekogbon Obadamilare Testimony

GitHub: https://github.com/Obadamilare

## Acknowledgements

Thanks to Coach Timothy and SmartBizCrux Technologies for the guidance and learning opportunity.
