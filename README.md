# stETH wstETH Integration Guide

Draft %in progress%

## stETH

### What stETH is

Normally users need to run own node and require at least 32ETH to participate in ETH 2.0. Funds will be locked until it is released. But with LIDO which allows users to stake their ETH into the ETH2.0 PoS network, anyone can stake  ETH and get back stETH + the x% staking reward from ETH2.0. The benefit of this is that stETH can be swapped for ETH at any time so users are no longer locked until ETH2.0 is released. 

The stETH token is a tokenized version of staked ether. When a user sends ether into the Lido liquid staking smart contract, the user receives the corresponding amount of stETH tokens. The stETH token represents Lido user’s deposits and the corresponding staking rewards and slashing penalties. It is also is a liquid alternative for the staked ether: it could be transferred, traded, or used in DeFi applications. The stETH token balance is be calculated based on the total amount of staked ether, plus rewards and minus any slashing penalties.

Lido makes the stETH token balance track a balance of corresponding balance of beacon chain ether. A user’s balance of stETH tokens corresponds 1 to 1 to an amount of ether a user could receive if withdrawals were enabled and instant.

The DAO selects node operators, which also validate transactions on the beacon chain and adds their addresses to the `NodeOperatorsRegistry` contract. Authorized node operators have to generate a set of keys for the validation and also provide them with the smart contract. As ether is received from users, it is distributed in chunks of 32 Ether between all active node operators. The staking pool contract contains a list of node operators, their keys, and the logic for distributing rewards between them.

StETH is a rebasable token. It receives reports from the Oracle contract `pushBeacon method` with the state of the protocol's ETH2 validators balances, and updates all the balances of stETH holders distributing the protocol's total staking rewards and penalties. The protocol employs distributed Oracle reporting: there are five Oracle daemons running by the Lido Node operators, and the Oracle smart contract formats beacon report on the consensus of three of five daemon reports. On top of the consensus mechanics, there are sanity checks for reports with sudden drops in total ETH2 balance or rewards with higher-than-possible APY. 

stETH is very good collateral for people who are long Ethereum, as it lets them at once margin long ETH and compounds it with staking rewards. Staking rewards alone can pay off stability fee if it's sufficiently low. The protocol applies a 10% fee (this can be changed by the DAO) on staking rewards that are split between node operators, the DAO, and a slashing insurance fund.

Node operators also validate transactions on the beacon chain. The DAO selects node operators and adds their addresses to the NodeOperatorsRegistry contract. Authorized node operators have to generate a set of keys for the validation and also provide them with the smart contract. As ether is received from users, it is distributed in chunks of 32 Ether between all active node operators. The staking pool contract contains a list of node operators, their keys, and the logic for distributing rewards between them.

**Note:** As stETH is a rebasable token and integration of this asset requires a custom `GemJoin` contract. An easier and less risky way is to integrate wstETH, a trustless fixed-balance wrapper ([https://docs.lido.fi/contracts/wsteth](https://docs.lido.fi/contracts/wsteth)), and use the standard `GemJoin` contract.

1. The stETH is a token representing the share of the ETH staked with Lido on the beacon chain.


2. How the shares math works.
3. stETH & the ERC20 standard

StETH token is the upgradable contract behind `AppProxyUpgradeable` proxy at [Etherscan](https://etherscan.io/address/0xae7ab96520de3a18e5e111b5eaab095312d7fe84). 


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
