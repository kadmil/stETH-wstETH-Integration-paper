# stETH wstETH Integration Guide

Draft %in progress%

## stETH

### What stETH is

The section covers stETH basics:

1. The stETH is a token representing the share of the ETH staked with Lido on the beacon chain.
2. How the shares math works.
3. stETH & the ERC20 standard

### Rebases & beacon chain Oracle

1. StETH rebases & beacon chain Oracle reports
    1. "Happy path" scenario (daily report)
    2. Realistic exceptions from the "happy path"
        1. Skipping a day of report
        2. Reporting two days at once
    3. Corner cases (gotchas)
        1. Reporting slashings
        2. Reporting few weeks of no finality on eth2
        3.  Note on the case where the whole balance can't be transferred and 1 wei stays on the sender's account
2. Oracle reporting details
    1. Quorum for the beacon chain Oracle
    2. Sanity checks for reports
    3. Oracle APIs
        1. Beacon chain APR
        2. Hooking up to the Oracle updates
3. Slashing & burning the cover

### Price oracle & price feed for stETH

The section on price oracle ([github.com/lidofinance/curve-merkle-oracle](http://github.com/lidofinance/curve-merkle-oracle)) & the price feed ([github.com/lidofinance/steth-price-feed](http://github.com/lidofinance/steth-price-feed))

1. How the stETH price oracle works: how the price is reported, how the oracle is operated
2. How the stETH price feed works: what data sources are used, what API does it provide
3. What guarantees the price oracle & price feed provide

### Main liquidity venues for stETH

Section covering how to get stETH & how to exit stETH position

1. Staking with Lido: how to get stETH from Lido 1-1 to ETH
2. Curve steth pool [https://curve.fi/steth](https://curve.fi/steth)
    1. Incentives
    2. Peg stability
    3. Link to [https://blog.lido.fi/concerning-steth-liquidity/](https://blog.lido.fi/concerning-steth-liquidity/)

### stETH Farms

Section covering stETH liquidity mining programs

1. Curve pool [https://curve.fi/steth](https://curve.fi/steth)

## wstETH

The section covers wstETH token: the stable-balance stETH wrapper for DeFi integrations

1. What the wstETH is & how it works
2. `wrap`/`unwrap` & rewards accounting
3. How to use `permit`
4. Price feed for wstETH
5. Liquidation issues with steth

## Risks


## Superuser priviliges

[https://docs.lido.fi/token-guides/steth-superuser-functions](https://docs.lido.fi/token-guides/steth-superuser-functions)

## How to Integrate stETH 


## Good Examples


## Common Issues


## RoadMap
