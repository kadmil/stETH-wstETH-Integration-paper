# stETH wstETH Integration Guide

[![Supported by LEGO](https://img.shields.io/badge/Supported%20by-LEGO-%2300A3FF)](https://www.notion.so/LEGO-Lido-Ecosystem-Grants-Organisation-d7f0bf0182d44348b6173639d2e8363d)   

### What stETH is

Normally users need to run own node and require at least 32ETH to participate in ETH 2.0. Funds will be locked until it is released. But with LIDO which allows users to stake their ETH into the ETH2.0 PoS network, anyone can stake  ETH and get back stETH + the x% staking reward from ETH2.0. The benefit of this is that stETH can be swapped for ETH at any time so users are no longer locked until ETH2.0 is released. 

The stETH token is a tokenized version of staked ether. When a user sends ether into the Lido liquid staking smart contract, the user receives the corresponding amount of stETH tokens. The stETH token represents Lido user’s deposits and the corresponding staking rewards and slashing penalties. It is also is a liquid alternative for the staked ether: it could be transferred, traded, or used in DeFi applications. The stETH token balance is be calculated based on the total amount of staked ether, plus rewards and minus any slashing penalties.

#

![stETH infrastructure](./02.png)


Lido makes the stETH token balance track a balance of corresponding balance of beacon chain ether. A user’s balance of stETH tokens corresponds 1 to 1 to an amount of ether a user could receive if withdrawals were enabled and instant.

The DAO selects node operators, which also validate transactions on the beacon chain and adds their addresses to the `NodeOperatorsRegistry` contract. Authorized node operators have to generate a set of keys for the validation and also provide them with the smart contract. As ether is received from users, it is distributed in chunks of 32 Ether between all active node operators. The staking pool contract contains a list of node operators, their keys, and the logic for distributing rewards between them.

#

![stETH infrastructure](./01.png)

StETH is a rebasable token. It receives reports from the Oracle contract `pushBeacon method` with the state of the protocol's ETH2 validators balances, and updates all the balances of stETH holders distributing the protocol's total staking rewards and penalties. The protocol employs distributed Oracle reporting: there are five Oracle daemons running by the Lido Node operators, and the Oracle smart contract formats beacon report on the consensus of three of five daemon reports. On top of the consensus mechanics, there are sanity checks for reports with sudden drops in total ETH2 balance or rewards with higher-than-possible APY. 

stETH is very good collateral for people who are long Ethereum, as it lets them at once margin long ETH and compounds it with staking rewards. Staking rewards alone can pay off stability fee if it's sufficiently low. The protocol applies a 10% fee (this can be changed by the DAO) on staking rewards that are split between node operators, the DAO, and a slashing insurance fund.

Node operators also validate transactions on the beacon chain. The DAO selects node operators and adds their addresses to the NodeOperatorsRegistry contract. Authorized node operators have to generate a set of keys for the validation and also provide them with the smart contract. As ether is received from users, it is distributed in chunks of 32 Ether between all active node operators. The staking pool contract contains a list of node operators, their keys, and the logic for distributing rewards between them.

**Note:** As stETH is a rebasable token and integration of this asset requires a custom `GemJoin` contract. An easier and less risky way is to integrate wstETH, a trustless fixed-balance wrapper ([https://docs.lido.fi/contracts/wsteth](https://docs.lido.fi/contracts/wsteth)), and use the standard `GemJoin` contract. StETH token is the upgradable contract behind `AppProxyUpgradeable` proxy at [Etherscan](https://etherscan.io/address/0xae7ab96520de3a18e5e111b5eaab095312d7fe84). 


## Tokenomics

Beacon chain's withdrawals are scheduled after merge which should happen around Q1-Q2 2022. Once these features are deployed, the Lido DAO will upgrade Lido to allow the users to burn stETH tokens in exchange for ether. 

[wstETH](https://docs.lido.fi/contracts/wsteth) is a constant-balance wrapper ERC20 token that converts stETH balances which undergo periodical rebases to the underlying shares balances. As Lido oracles report beacon chain rewards, penalties, or slashings, wstETH token balances remain unchanged: instead, the amount of stETH corresponding to one wstETH (and thus wstETH price) changes. Anyone can convert stETH to wstETH and vice versa on-chain through [wstETH.wrap](https://docs.lido.fi/contracts/wsteth#wrap) and [wstETH.unwrap](https://docs.lido.fi/contracts/wsteth#unwrap) functions.

Since wstETH and stETH represent the same underlying asset (Ether staked with Lido plus the rewards accrued) and can be mutually converted at any given time, one can consider the combination of wstETH and stETH liquidity on all exchanges as contributing to a common liquidity pool. stETH price can be converted to wstETH price by multiplying the former by a coefficient obtained from either stETH or wstETH contract.

## Mechanics of stETH

stETH token balances update once a day when the oracle reports changes in Eth2 deposits and changes in ETH rewards from users who stake via Lido. This occurs once a day at 12PM UTC. Because the rewards are embodied through a balance rebase, users who hold stETH will not see a transaction sent to their wallet. Rather, users should see their stETH balance automatically change without an accompanying transaction taking place.

This rebase works across integrated DeFi platforms like Curve and Yearn. This means that if you are to stake your stETH across these protocols to earn additional yields, you will continue to benefit from daily stETH staking rewards as well. UniSwap, 1inch and SushiSwap are not designed for rebasable tokens and as a result you risk losing out on a portion of your daily staking rewards through providing stETH as liquidity across these platforms.

When a user deposits ETH via Lido, that ETH is split between node operators which is then sent to their respective validators.

## The stETH Reward Rate

Users who stake their ETH with Lido will receive daily rewards - in the form of stETH balance rebases - from day one. This is possible because staking rewards with Lido are socialised across all stakers. Rebases affect all holders of stETH regardless of whether their ETH has actually been deposited into the queue as of yet.
This mechanism is the reason why the stETH reward rate is currently lower than that of Ethereum.

Only a portion of Lido validators have made it through the queue, from which all existing stETH holders are accruing their rewards – including the new depositors. This results in an initially lower reward rate because the amount of rewards being accrued from the minority of already accepted validators is being split proportionally towards all stETH holders.

As more of Lido’s validators are activated, the stETH reward rate will grow correspondingly and gravitate towards the full Ethereum staking rate.
To track the Eth validator queue, visit eth2-validator-queue.web.app.

A dashboard to view these validators (and their time in queue alongside their estimated finalizing date) is currently being developed. This dashboard will also display related information such as: number of current validators, rewards being paid out to stETH holders, total amount of ETH staked via Lido, the active number of stakers, Lido APY, Eth2 APY.

# Rebases & beacon chain Oracle

## Lido

- [Source code](https://github.com/lidofinance/lido-dao/blob/master/contracts/0.4.24/Lido.sol)
- [Deployed contract](https://etherscan.io/address/0xae7ab96520de3a18e5e111b5eaab095312d7fe84)

Lido is the core contract which acts as a liquid staking pool. The contract is responsible for Ether deposits and withdrawals, minting and burning liquid tokens, delegating funds to node operators, applying fees, and accepting updates from the oracle contract. Node Operators' logic is extracted to a separate contract, NodeOperatorsRegistry.

Lido also acts as an ERC20 token which represents staked ether, stETH. Tokens are minted upon deposit and burned when redeemed. stETH tokens are pegged 1:1 to the Ethers that are held by Lido. stETH token’s balances are updated when the oracle reports change in total stake every day.

## Rebasing

When a rebase occurs the supply of the token is increased or decreased algorithmically, based on the staking rewards(or slashing penalties) in the Eth2 chain. Rebase happens when oracles report beacon stats.

Rebasing mechanism implemented via "shares". Instead of storing map with account balances, Lido stores which share owned by account in the total amount of Ether controlled by the protocol.

Balance of account calculated next way:

```
balanceOf(account) = shares[account] * totalPooledEther / totalShares
```

- `shares` - map of user account shares. Every time user deposit ether, it converted to shares and added to current user shares.

- `totalShares` sum of shares of all account in `shares` map

- `totalPooledEther` is a sum of three types of ether owned by protocol:

  - buffered balance - ether stored on contract and haven't deposited to official Deposit contract yet.
  - transient balance - ether submitted to the official Deposit contract but not yet visible in the beacon state.
  - beacon balance - total amount of ether on validator accounts. This value reported by oracles and makes strongest impact to stETH total supply change.

For example, assume that we have:

```
totalShares = 500
totalPooledEther = 10 ETH
sharesOf(Alice) -> 100
sharesOf(Bob) -> 400
```

Therefore:

```
balanceOf(Alice) -> 2 tokens which corresponds 2 ETH
balanceOf(Bob) -> 8 tokens which corresponds 8 ETH
```

## Beacon Stats Reporting

One of the most important parts of protocol, it's precise and steady reported data about current balances of validators. Such reports happen once at defined period of time, called frame. Frame duration set by DAO, current value is 24 hours.

To update stats on main Lido contract oracle demands quorum to be reached.
Quorum - is a necessary amount of reports with equal stats from offchain oracle daemons run by protocol participants.
Quorum size and members controlled by DAO.
If quorum wasn't reached next report can happen only at the first epoch of next frame (after 24 hours).

Report consists of count of validators participated in protocol - beacon validators and total amount of ether on validator accounts - beacon balance. Typically beacon balance growth from report to report, but in exceptional cases it also can drops, because of slashing.

- When beacon balance grown between reports, protocol register profit and distribute reward of fresh minting stETH tokens between stETH holders, node operators, insurance fund and treasury. Fee distribution for node operators, insurance fund and treasury can be set by DAO.
- When frame was ended with slashing and new beacon balance less than previous one total supply of stETH becomes less than in previous report and no rewards distributed.


# LidoOracle

- [Source code](https://github.com/lidofinance/lido-dao/blob/master/contracts/0.4.24/oracle/LidoOracle.sol)
- [Deployed contract](https://etherscan.io/address/0x442af784A788A5bd6F42A01Ebe9F287a871243fb)

LidoOracle is a contract where oracles send addresses' balances controlled by the DAO on the ETH
2.0 side. The balances can go up because of reward accumulation and can go down due to slashing and
staking penalties. Oracles are assigned by the DAO.

Oracle daemons push their reports every frame (225 epochs currently, equal to one day) and when the
number of the same reports reaches the ['quorum'](#getquorum) value, the report is pushed to the
[Lido contract][6].

The following mechanisms are also worth mentioning.

## Store the collected reports as an array

The report variant is a report with a counter - how many times this report was pushed by
oracles. This strongly simplified logic of `_getQuorumReport`, because in the majority of cases, we
only have 1 variant of the report so we just make sure that its counter exceeded the quorum value.


**Note:** The important note here is that when we remove an oracle (with `removeOracleMember`), we
also need to remove her report from the currently accepted reports. As of now, we do not keep a
mapping between members and their reports, we just clean all existing reports and wait for the
remaining oracles to push the same epoch again.


## Sanity checks the oracles reports by configurable values

In order to limit the misbehaving oracles impact, we want to limit oracles report change by 10% APR
increase in stake and 5% decrease in stake. Both values are configurable by the governance in case
of extremely unusual circumstances.


**Note:** Note that the change is evaluated after the quorum of oracles reports is reached, and not
on the individual report.


And the logic of reporting to the [Lido contract][6] got a call to `_reportSanityChecks` that does
the following. It compares the `preTotalPooledEther` and `postTotalPooledEther` (see above) and

- if there is a profit or same, calculates the [APR][1], compares it with the upper bound. If was
  above, reverts the transaction with `ALLOWED_BEACON_BALANCE_INCREASE` code.
- if there is a loss, calculates relative decrease and compares it with the lower bound. If was
  below, reverts the transaction with `ALLOWED_BEACON_BALANCE_DECREASE` code.


## StableSwapStateOracle

- [Source Code](https://github.com/lidofinance/curve-merkle-oracle/blob/main/contracts/StableSwapStateOracle.sol)
- [Deployed Contract](https://etherscan.io/address/0x3a6bd15abf19581e411621d669b6a2bbe741ffd6)

A trustless oracle for the ETH/stETH Curve pool using Merkle Patricia proofs of Ethereum state.

Contract receives and verifies the report from the offchain code,
and persists the verified state along with its timestamp.

The oracle assumes that the pool's `fee` and `A` (amplification coefficient) values don't
change between the time of proof generation and submission.

## StEthPriceFeed

- [Source Code](https://github.com/lidofinance/steth-price-feed/blob/main/contracts/StEthPriceFeed.vy)
- [Deployed Contract](https://etherscan.io/address/0xab55bf4dfbf469ebfe082b7872557d1f87692fe6)

Lido intends to provide secure and reliable price feed for stETH for protocols that intend to integrate it. Unfortunately, Chainlik is not available for stETH and Uniswap TWAP is not feasible at the moment: we'd want deep liquidity on stETH/ETH pair for this price, but Uni v2 doesn't allow tight curves for similaraly-priced coins.

stETH has deep liquidity in the Curve pool but it doesn't have a TWAP capability, so that's out, too. In the moment Curve price is flashloanable, if not easily. We decided that in a pinch we can provide a "price anchor" that would attest that "stETH/ETH price on Curve used to be around in recent past" (implemented using the [StableSwapStateOracle](./stable-swap-state-oracle)) and a price feed that could provide a reasonably safe estimation of current stETH/ETH price.

## Vocabulary

- **Current price**—current price of stETH on Curve pool. Flashloanable.
- **Historical price**—the price of stETH on Curve pool that was at least 15 blocks ago. May be older than 15 blocks: in that case, the pool price that was 15 blocks ago differs from the "historical price" by no more than `N`%.
- **Safe price range**—the range from `historical price - N%` to `min(historical price + N%, 1)`.
- **Safe price**—the price that's within the safe price range.

The parameter `N` is configured by the price feed admin; we're planning to initially set it to `5%`.

## stETH price feed specification

The feed is used to fetch stETH/ETH pair price in a safe manner. By "safe" we mean that the price should be expensive to significantly manipulate in any direction, e.g. using flash loans or sandwich attacks.

The feed interfaces with two contracts:

1. Curve stETH/ETH pool: [source](https://github.com/curvefi/curve-contract/blob/c6df0cf/contracts/pools/steth/StableSwapSTETH.vy), [deployed contract](https://etherscan.io/address/0xdc24316b9ae028f1497c275eb9192a3ea0f67022).
2. [StableSwapStateOracle](./stable-swap-state-oracle)

The pool is used as the main price source, and the oracle provides time-shifted price from the same pool used to establish a safe price range.

The price is defined as the amount of ETH wei needed to buy 1 stETH. For example, a price equal to `10**18` would mean that stETH is pegged 1:1 to ETH.

The safe price is defined as the one that satisfies all of the following conditions:

- The absolute value of percentage difference between the safe price and the time-shifted price fetched from the Merkle oracle is at most `max_safe_price_difference`.
- The safe price is at most `10**18`, meaning that stETH cannot be more expensive than ETH.

## Fail conditions

Price feed can give incorrect data in, as far as we can tell, three situations:

- stETH/ETH price moving suddenly and very quickly. There is at least 15 blocks delay between price drop and offchain oracle feed providers submitting a new historical price, and likely more bc tx are not mined instanteously. That should not happen normally: while stETH/ETH is volatile, it's not 5%-in-four-minutes volatile.
- oracle feed going stale because feed providers go offline. This is mitigated by the fact it's operated by several very experienced professionals (all of which, e.g., are Chainlink operators too) - and we only need one operational provider to maintain the feed. The only realistic scenario where this feed goes offline is deprecating the oracle alltogether.
- Multi-block flashloan attack. An block producer who is able to reliably get 2 blocks in a row can treat two blocks as an atomic transaction, leading to what is essentially a multiblock flashloan attack to manipulate price. That can lead to a short period of time (a few blocks) where stETH/ETH price feed is artificially manipulated. This attack is not mitigated, but in our opinion, not very realistic. It's very hard to pull off.


## Oracle contract upgrade to v2

The following changes are to be made to the first mainnet version of the oracle contract. [More info](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-2.md)

# wstETH

- [Source Code](https://github.com/lidofinance/lido-dao/blob/develop/contracts/0.6.12/WstETH.sol)
- [Deployed Contract](https://etherscan.io/token/0x7f39c581f595b53c5cb19bd0b3f8da6c935e2ca0)

It's an ERC20 token that represents the account's share of the total
supply of stETH tokens. WstETH token's balance only changes on transfers,
unlike StETH that is also changed when oracles report staking rewards and
penalties. It's a "power user" token for DeFi protocols which don't
support rebasable tokens.

The contract is also a trustless wrapper that accepts stETH tokens and mints
wstETH in return. Then the user unwraps, the contract burns user's wstETH
and sends user locked stETH in return.

The contract provides the staking shortcut: user can send ETH with regular
transfer and get wstETH in return. The contract will send ETH to Lido submit
method, staking it and wrapping the received stETH.


## Contract Logic Summary

- [wstETH](https://docs.lido.fi/contracts/wsteth) is a permissionless constant-balance wrapper token for the rebasing [stETH](https://docs.lido.fi/contracts/lido), the main Ethereum liquid staking token by Lido.

- The amount of stETH an address holds corresponds to the amount of staked ETH plus the Beacon chain rewards accrued and/or penalties inflicted. After withdrawals are enabled in the Merge network, one would be able to convert stETH to ETH 1:1 using the Lido protocol.

- To reflect Beacon chain rewards and/or penalties, balances of all stETH holders are adjusted daily according to the Beacon validators state reported by a quorum of Lido Oracles. In contrast, wstETH token balances remain unchanged: instead, the amount of stETH corresponding to one wstETH changes. Anyone can convert stETH to wstETH and vice versa on-chain through [`wstETH.wrap`](https://docs.lido.fi/contracts/wsteth#wrap) and [`wstETH.unwrap`](https://docs.lido.fi/contracts/wsteth#unwrap) functions.

## Administrative Addresses

No administrative access is built into wstETH.

Note that stETH contract has admin functions that could affect wstETH market value:

- stETH is an upgradable contract, with the upgrade requiring the a DAO vote performed among LDO holders. Voting app is: https://etherscan.io/address/0x2e59a20f205bb85a89c53f1936454680651e618e.
- The Lido protocol can be stopped by a DAO vote performed among LDO holders. This won't affect wstETH contract in any way except wrapping stETH to wstETH and unwrapping wstETH to stETH won't be possible. Voting app is: https://etherscan.io/address/0x2e59a20f205bb85a89c53f1936454680651e618e. 
- stETH balances are adjusted upon Lido Oracle reports. This operation doesn't affect wstETH balances in any way. Oracle contract is: https://etherscan.io/address/0x442af784A788A5bd6F42A01Ebe9F287a871243fb

## Inheritance Structure

The token contract inherits from the OpenZeppelin's `ERC20Permit`.

## Other

wstETH implements [EIP-2612 Permit](https://eips.ethereum.org/EIPS/eip-2612) standard for `secp256k1`-signed approvals.


## Contracts Description Table


|  Contract  |         Type        |       Bases      |                  |                 |
|:----------:|:-------------------:|:----------------:|:----------------:|:---------------:|
|     └      |  **Function Name**  |  **Visibility**  |  **Mutability**  |  **Modifiers**  |
||||||
| **WstETH** | Implementation | ERC20Permit |||
| └ | <Constructor> | Public ❗️ | 🛑  | ERC20Permit ERC20 |
| └ | wrap | External ❗️ | 🛑  |NO❗️ |
| └ | unwrap | External ❗️ | 🛑  |NO❗️ |
| └ | <Receive Ether> | External ❗️ |  💵 |NO❗️ |
| └ | getWstETHByStETH | External ❗️ |   |NO❗️ |
| └ | getStETHByWstETH | External ❗️ |   |NO❗️ |
| └ | stEthPerToken | External ❗️ |   |NO❗️ |
| └ | tokensPerStEth | External ❗️ |   |NO❗️ |


## Legend


|  Symbol  |  Meaning  |
|:--------:|-----------|
|    🛑    | Function can modify state |
|    💵    | Function is payable |



# Use cases in DeFi

## Liquidity Pools

Liquidity pools are collections of liquidity which consist of tokens which can be seamlessly exchanged with one another through the use of AMMs (automated market makers). Popular examples of platforms that use liquidity pool AMMs are Uniswap, Curve, and SushiSwap.
 
stETH can be pooled together with vanilla ETH in a liquidity pool. This in turn allows users to indirectly unstake their ETH and receive their initial ETH deposit back via pool swaps, bypassing the time required to wait for transactions on Eth2 to be enabled if a user decides that they would like to unstake.
 
![stETH ETH pool on Curve Finance](./03.png)
 
Due to stETH’s relationship with vanilla ETH (users will be able to redeem an equivalent amount of ETH for stETH once transactions are enabled), we hope this will result in less impermanent loss for liquidity providers compared to other conventional liquidity pools - allowing for liquidity providers to gain trading fees without engaging in too much risk - as well as to help hold the peg between the two assets.
 
Liquidity pools are a primary structure in Lido’s ecosystem that helps maintain stETH as the liquid equivalent of vanilla ETH. Without liquidity pools, users will not be able to unstake until transactions are enabled, breaking a core aspect of Lido’s manifesto.
 
There are ongoing liquidity mining programs that are occurring with stETH and LDO. You can read more information about them here:

- stETH-ETH Curve pool
- LDO-ETH Onsen pool
- Curve's stETH-ETH liquidity pool allows liquidity providers to simultaneously accrue trading fees, liquidity mining rewards (CRV, LDO), as well as Lido’s Eth2 reward rate.
- SushiSwaps LDO-ETH Onsen pool allows liquidity providers to simultaneously accrue trading fees, liquidity mining rewards (SUSHI), as well as Lido’s Eth2 reward rate.
 
## Lending
Lending protocols can adopt stETH to allow users to borrow assets while simultaneously accruing Eth2 rewards while still being staked as collateral.
 
To do so, stETH must first be wrapped to be applicable as collateral. This opens up an extra layer of efficiency and composability when it comes to the DeFi ecosystem in relation to yield farming and borrowing. Some examples of lending protocols that may adopt stETH are: Aave, Maker, Compound, Cream, Alpha).
 
The addition of stETH as collateral in lending protocols allows for advanced composable yield farming strategies. A user will be able to deposit stETH as collateral and take out an ETH loan to be further swapped back to stETH to add to their existing loan as a sort of leveraged position. Alternatively, ETH borrowed can be put to other purposes, such as: staking in Lido, depositing into other strategies, providing it as liquidity in a liquidity pool, etc.
 
Price fluctuations of the underlying collateral asset still play a large role in determining a user's health ratio and liquidation risk. Although, this theoretically allows users - who stake stETH as collateral while borrowing a position - to constantly improve their health ratio whilst constantly diminishing the possibilities of any unwanted liquidations.
 
Lending protocols may also allow for the borrowing of stETH in the form of a loan. This would allow users to take out a loan that is, in essence, constantly paying off itself. If there are many people that would like to borrow stETH, suppliers can also earn yield on their stETH as well, earning the Eth2 rate simultaneously with the variable lending rate.
 
With the recent advent and hysteria of undercollateralized loans (credit delegation or protocol-to-protocol), lending protocols may support stETH loans to other protocols (specifically yield aggregators) without needing to supply collateral beforehand. This is particularly efficient with the use of stETH, as the rebasing factor of stETH helps the borrower pay off their debt easier simply by holding it.
 
Proposals have been shared on the governance forums of certain lending protocols. Users have discussed the possibilities of adding stETH as collateral. You can read more information about it here:

- Maker stETH onboarding proposal
- Collateral onboarding call with Maker
- Aave stETH onboarding proposal

Although stETH is a freely tradable asset it does have issues when being used as collateral. Most DEXs and AAVE do not support rebasing of assets locked into the protocol. So depositing stETH into AAVE will lead user to losing rewards for the time that it is locked. LIDO has already solved this with creating wrapped stETH (Wrapped stETH (wstETH) | Lido: Help). 

In a nutshell, user locks stETH into a contract that returns wstETH to him. This wstETH can be locked into any DEX or lending platform while the stETH user wrapped accrues staking rewards in the background. When user unwraps wstETH he will receive stETH deposited + rewards. This is done in a similar way to how aUST can usually be traded for more UST since aUST represents deposited UST with interest earned.
 
## Strategies/Aggregators

Aggregators can use stETH in their yield farming strategies as an additional yield layer on top of their already existing yield farming. Strategies are flexible and can maximize the highest yield possible for their users. Popular examples of yield aggregators include: Yearn, Harvest, Badger.
 
These strategies can utilize a variety of other protocols/initiatives to generate this high yield, such as: farming through liquidity mining incentives, earning yield through lending protocols (as discussed prior), earning yield through native protocol staking, and so on.

![stETH on Harvest Finance](./04.png)


An example of an existing strategy is the st. Ether-ETH pool, which uses the liquidity mining rewards earned from providing liquidity in the Curve stETH-ETH pool to automatically compound into stETH/ETH which is used to stake back into the pool.
 
With the addition of lending protocols adopting stETH as collateral, new strategies can be implemented utilizing the borrowed assets taken as a loan. Users can provide their stETH as collateral, borrow a position, then use those borrowed assets for a variety of yield farming opportunities, such as providing liquidity into the Curve pool, providing liquidity into the Onsen pool, borrowing another position, and more.
 
## Derivatives

The derivatives sector is extremely vast and can expand to all types of subsectors. For this reason, specific details may have slight discrepancies for each derivatives platform since each of them have their own unique intricacies. The examples below illustrate an idea of possible use cases of stETH. Keep in mind that these are currently only hypothetical ideas, and quite possibly some ideas are incompatible.
 
Synthetic-issuing protocols may allow stETH to be used as collateral to mint synths, similar to how lending protocols function. In this case, users will mint a synthetic asset that can be used/traded to track the performance of another asset that can even be unrelated to DeFi assets (eg. Gold, Silver, TSLA). Since synthetics have infinite liquidity, synthetic ETH can be pooled with stETH to allow for low slippage trades between the two assets. Synthetic swaps would allow for cross-asset trades between stETH to any supported synthetic asset.
 
Insurance is another derivative. Although stETH may not have a use case for insurance, insurance can be bought to insure against any smart contract errors or validator slashings. Unslashed Finance recently partnered with Lido to insure against a 5% validator slashing. Future partnerships with other insurance protocols can further mitigate risks any stakers may have.
 
In the future, stETH may also expand to put/call options.

## Brief FAQ:


**1.** What purpose does wstETH serve? I assume mostly as a replacement for the stETH token in LPs that don't work correctly with elastic supply tokens, which is essentially stETH. Are there other reasons for wrapping stETH in wstETH?

- Right, wstETH can be used instead of stETH in integrations with protocols that do not support rebasable tokens. AMM, bridges between blockchains and money markets (marker, aave etc) are the most important.


**2.** How much can the price of wstETH change after unwrap relative to the original stETH? What circumstances can cause this?

- The amount of stETH obtained after unwrap corresponds to the amount of stETH that would have been in the wallet even without wrapping the asset in wstETH.

**2.1** What if others occur between the wrap and unwrap operations, how will they affect the value obtained after unwrap, compared to the original? 

- Wrap 10 stheth->hold wsteth->unwrap is identical to hold 10 steth. Deposit/withdraw does not affect the amount after unwrap, reward/slash/burn has the same effect as for the corresponding amount of stETH. Holding wstETH is absolutely identical to holding the same amount of wstETH.

**3.** The wrap function wstETH fixes a value in the contract that is essentially the numerator of the fraction share/totalShare. Conventionally, this is the value of share of stETH; After wrap the data of stETH token can change - the denominator (totalShare) can decrease e.g. because of the slash. If afterwards you execute unwrap, the numerator will have the same value of share of stETH; the ratio share/totalShare may be much higher than the original one.

- TotalShare does not change from slashing, it changes from deposits, withdrawals and burn. As a result, withdrawals and burn share/totalShare may increase.


**3.1** Can this ratio exceed 1 and cause an error?

- The denominator of the fraction share/totalShare is calculated as the sum of all shares, so it cannot be less than the numerator. The ratio can be equal to one in the theoretical case if the address is the only holder of stETH.


**3.2** Does stETH totalShare change downward only in case of slash ? 

- stETH totalShare does not change downwards in case of slash. In case of slash the amount of stETH corresponding to share changes.


**3.3** In the normal course of the process should totalShare only increase ?

- Correct, while there is no way to take out ETH, totalShare should increase as the amount of tainted assets increases. Technically, you could burn stETH to reduce totalShare, but in practice this does not happen. Lido may resort to this mechanism to handle slashing + slashing insurance, or to deal with MEVs, but so far there has been no occasion to do so.


**3.4** When wrap followed by unwrap do we end up with a smaller stETH value compared to the initial one?

- A wrap followed by an unwrap may result in a lower stETH compared to the initial stETH if the amount of ether locked in the protocol has decreased since the wrap (at this point this may be due to slashing or staking penalties).


## Contracts

### Core Protocol

- Lido DAO: [`0xb8FFC3Cd6e7Cf5a098A1c92F48009765B24088Dc`](https://etherscan.io/address/0xb8FFC3Cd6e7Cf5a098A1c92F48009765B24088Dc) (proxy)
- LDO token: [`0x5A98FcBEA516Cf06857215779Fd812CA3beF1B32`](https://etherscan.io/address/0x5A98FcBEA516Cf06857215779Fd812CA3beF1B32)
- Lido and stETH token: [`0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84`](https://etherscan.io/address/0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84) (proxy)
- wstETH token: [`0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84`](https://etherscan.io/address/0x7f39c581f595b53c5cb19bd0b3f8da6c935e2ca0)
- Node Operators registry: [`0x55032650b14df07b85bF18A3a3eC8E0Af2e028d5`](https://etherscan.io/address/0x55032650b14df07b85bF18A3a3eC8E0Af2e028d5) (proxy)
- Oracle: [`0x442af784A788A5bd6F42A01Ebe9F287a871243fb`](https://etherscan.io/address/0x442af784A788A5bd6F42A01Ebe9F287a871243fb) (proxy)
- Stable Swap Oracle [`0x3a6bd15abf19581e411621d669b6a2bbe741ffd6`](https://etherscan.io/address/0x3a6bd15abf19581e411621d669b6a2bbe741ffd6)
- stETH Price Feed [`0xab55bf4dfbf469ebfe082b7872557d1f87692fe6`](https://etherscan.io/address/0xab55bf4dfbf469ebfe082b7872557d1f87692fe6) (proxy)
- Aragon Voting: [`0x2e59A20f205bB85a89C53f1936454680651E618e`](https://etherscan.io/address/0x2e59A20f205bB85a89C53f1936454680651E618e) (proxy)
- Aragon Token Manager: [`0xf73a1260d222f447210581DDf212D915c09a3249`](https://etherscan.io/address/0xf73a1260d222f447210581DDf212D915c09a3249) (proxy)
- Aragon Finance: [`0xB9E5CBB9CA5b0d659238807E84D0176930753d86`](https://etherscan.io/address/0xB9E5CBB9CA5b0d659238807E84D0176930753d86) (proxy)
- Aragon Agent: [`0x3e40D73EB977Dc6a537aF587D48316feE66E9C8c`](https://etherscan.io/address/0x3e40D73EB977Dc6a537aF587D48316feE66E9C8c) (proxy)

### Reward Programs

- Early Stakers Airdrop: [`0x4b3EDb22952Fb4A70140E39FB1adD05A6B49622B`](https://etherscan.io/address/0x4b3EDb22952Fb4A70140E39FB1adD05A6B49622B)

- 1inch Liquidity Farming: [`0xdB46C277dA1599390eAb394327602889E9546296`](https://etherscan.io/address/0xdB46C277dA1599390eAb394327602889E9546296)

- Curve Liquidity Farming:

- Manager Contract: [`0x753D5167C31fBEB5b49624314d74A957Eb271709`](https://etherscan.io/address/0x753D5167C31fBEB5b49624314d74A957Eb271709)
- Reward Contract: [`0x99ac10631f69c753ddb595d074422a0922d9056b`](https://etherscan.io/address/0x99ac10631f69c753ddb595d074422a0922d9056b)
- Pool Contract: [`0xDC24316b9AE028F1497c275EB9192a3Ea0f67022`](https://etherscan.io/address/0xDC24316b9AE028F1497c275EB9192a3Ea0f67022)

- ARCx:

- Manager Contract: [`0x6140182B2536AE7B6Cfcfb2d2bAB0f6Fe0D7b58E`](https://etherscan.io/address/0x6140182B2536AE7B6Cfcfb2d2bAB0f6Fe0D7b58E)
- Reward Contract: [`0x8F1155447Ee97b5Ae147a01a5c420B0FDDF0370D`](https://etherscan.io/address/0x8F1155447Ee97b5Ae147a01a5c420B0FDDF0370D)

## Görli+Prater testnet

### Core Protocol

- Lido DAO: [`0x1dD91b354Ebd706aB3Ac7c727455C7BAA164945A`](https://goerli.etherscan.io/address/0x1dD91b354Ebd706aB3Ac7c727455C7BAA164945A) (proxy)
- LDO token: [`0x56340274fB5a72af1A3C6609061c451De7961Bd4`](https://goerli.etherscan.io/address/0x56340274fB5a72af1A3C6609061c451De7961Bd4)
- Lido and stETH token: [`0x1643E812aE58766192Cf7D2Cf9567dF2C37e9B7F`](https://goerli.etherscan.io/address/0x1643E812aE58766192Cf7D2Cf9567dF2C37e9B7F) (proxy)
- wstETH token: [`0x6320cd32aa674d2898a68ec82e869385fc5f7e2f`](https://goerli.etherscan.io/address/0x6320cd32aa674d2898a68ec82e869385fc5f7e2f)
- Node Operators registry: [`0x9D4AF1Ee19Dad8857db3a45B0374c81c8A1C6320`](https://goerli.etherscan.io/address/0x9D4AF1Ee19Dad8857db3a45B0374c81c8A1C6320) (proxy)
- Oracle: [`0x24d8451BC07e7aF4Ba94F69aCDD9ad3c6579D9FB`](https://goerli.etherscan.io/address/0x24d8451BC07e7aF4Ba94F69aCDD9ad3c6579D9FB) (proxy)
- Stable Swap Oracle [`0x4522dB9A6f804cb837E5fC9F547D320Da3edD49a`](https://goerli.etherscan.io/address/0x4522dB9A6f804cb837E5fC9F547D320Da3edD49a)
- Aragon Voting: [`0xbc0B67b4553f4CF52a913DE9A6eD0057E2E758Db`](https://goerli.etherscan.io/address/0xbc0B67b4553f4CF52a913DE9A6eD0057E2E758Db) (proxy)
- Aragon Token Manager: [`0xDfe76d11b365f5e0023343A367f0b311701B3bc1`](https://goerli.etherscan.io/address/0xDfe76d11b365f5e0023343A367f0b311701B3bc1) (proxy)
- Aragon Finance: [`0x75c7b1D23f1cad7Fb4D60281d7069E46440BC179`](https://goerli.etherscan.io/address/0x75c7b1D23f1cad7Fb4D60281d7069E46440BC179) (proxy)
- Aragon Agent: [`0x4333218072D5d7008546737786663c38B4D561A4`](https://goerli.etherscan.io/address/0x4333218072D5d7008546737786663c38B4D561A4) (proxy)

### Reward Programs

- Curve Liquidity Farming:

- Pool Contract: [`0xCEB67769c63cfFc6C8a6c68e85aBE1Df396B7aDA`](https://goerli.etherscan.io/address/0xCEB67769c63cfFc6C8a6c68e85aBE1Df396B7aDA)

## Görli+Pyrmont testnet

### Core Protocol

- Lido DAO: [`0xE9c991d2c9Ac29b041C8D05484C2104bD00CFF4b`](https://goerli.etherscan.io/address/0xE9c991d2c9Ac29b041C8D05484C2104bD00CFF4b) (proxy)
- LDO token: [`0xF837FBd803Ad6EdA0a89c5acF8785034F5aB33f2`](https://goerli.etherscan.io/address/0xF837FBd803Ad6EdA0a89c5acF8785034F5aB33f2)
- stETH token: [`0xA0cA1c13721BAB3371E0609FFBdB6A6B8e155CC0`](https://goerli.etherscan.io/address/0xA0cA1c13721BAB3371E0609FFBdB6A6B8e155CC0) (proxy)
- Lido and stETH token: [`0xA5d26F68130c989ef3e063c9bdE33BC50a86629D`](https://goerli.etherscan.io/address/0xA5d26F68130c989ef3e063c9bdE33BC50a86629D) (proxy)
- Node Operators registry: [`0xB1e7Fb9E9A71063ab552dDEE87Ea8C6eEc7F5c7A`](https://goerli.etherscan.io/address/0xB1e7Fb9E9A71063ab552dDEE87Ea8C6eEc7F5c7A) (proxy)
- Oracle: [`0x8aA931352fEdC2A5a5b3E20ed3A546414E40D86C`](https://goerli.etherscan.io/address/0x8aA931352fEdC2A5a5b3E20ed3A546414E40D86C) (proxy)
- Stable Swap Oracle [`0x9A066bD669e2795fe2B936FA959FD414eBB004E9`](https://goerli.etherscan.io/address/0x9A066bD669e2795fe2B936FA959FD414eBB004E9)
- Aragon Voting: [`0xA54DBf1B494113fBDA2E593419eE7241EfE8B766`](https://goerli.etherscan.io/address/0xA54DBf1B494113fBDA2E593419eE7241EfE8B766) (proxy)
- Aragon token manager: [`0xB90D5df4aBDf5F69a00088d43E4A0Fa8A8b44244`](https://goerli.etherscan.io/address/0xB90D5df4aBDf5F69a00088d43E4A0Fa8A8b44244) (proxy)
- Aragon finance: [`0xfBfa38921d745FD7bE9fa657FFbcDFecC4Ab7Cd4`](https://goerli.etherscan.io/address/0xfBfa38921d745FD7bE9fa657FFbcDFecC4Ab7Cd4) (proxy)
- Aragon agent: [`0xd616af91a0C3fE5AEeA0c1FaEfC2d73AcA82F0c9`](https://goerli.etherscan.io/address/0xd616af91a0C3fE5AEeA0c1FaEfC2d73AcA82F0c9) (proxy)

### Reward Programs

- Curve Liquidity Farming:

- Pool Contract: [`0x12edd9e2073E480cc546e1E0aD7F1c9D60c0cA1E`](https://goerli.etherscan.io/address/0x12edd9e2073E480cc546e1E0aD7F1c9D60c0cA1E)


**Resources**
#

Onboarding: [https://www.notion.so/stETH-Collateral-Onboarding-Risk-Evaluation-1ea3a6fb4dbf4132b2c5347dbba0c9ec](https://www.notion.so/stETH-Collateral-Onboarding-Risk-Evaluation-1ea3a6fb4dbf4132b2c5347dbba0c9ec)

Lido White Paper: [https://lido.fi/static/Lido:Ethereum-Liquid-Staking.pdf](https://lido.fi/static/Lido:Ethereum-Liquid-Staking.pdf)

Lido Website: [https://lido.fi](https://lido.fi/)

Lido Blog: [http://blog.lido.fi](http://blog.lido.fi) 

CoinMarketCap: [https://coinmarketcap.com/currencies/steth/historical-data/](https://coinmarketcap.com/currencies/steth/historical-data/)

CoinGecko: [https://www.coingecko.com/en/coins/lido-staked-ether/historical_data/eth](https://www.coingecko.com/en/coins/lido-staked-ether/historical_data/eth)

Ether Scan Lido Contract: [https://etherscan.io/token/0xae7ab96520de3a18e5e111b5eaab095312d7fe84](https://www.notion.so/20de3a18e5e111b5eaab095312d7fe84)

Dune Analytics: [https://duneanalytics.com/k06a/lido-finance](https://duneanalytics.com/k06a/lido-finance)

Nansen: [https://pro.nansen.ai/lido](https://pro.nansen.ai/lido)

Concerning stETH liquidity: [https://blog.lido.fi/concerning-steth-liquidity/](https://blog.lido.fi/concerning-steth-liquidity/)

Risks Calculation: [https://drive.google.com/file/d/1n6pdXKMxaRjxQfZi9WBJXnbD96SNYdxH/view](https://drive.google.com/file/d/1n6pdXKMxaRjxQfZi9WBJXnbD96SNYdxH/view)

Volatility Calculations: [https://colab.research.google.com/drive/1qEsWlsAPKqZgJUhazISPB6Ytcd1MeFGV?usp=sharing](https://colab.research.google.com/drive/1qEsWlsAPKqZgJUhazISPB6Ytcd1MeFGV?usp=sharing)

Pictures: [https://morioh.com/p/79d43d9d161a](https://morioh.com/p/79d43d9d161a)

Lido Docs: [docs.lido.fi](docs.lido.fi)

Oracle V2: [https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-2.md](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-2.md)

LEGO Program: [https://www.notion.so/stETH-wstETH-Integration-guide-outline-a9ed150deb3c4c38a9d78aec221d55b5](https://www.notion.so/stETH-wstETH-Integration-guide-outline-a9ed150deb3c4c38a9d78aec221d55b5)

