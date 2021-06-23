# stETH wstETH Integration Guide

Draft %in progress%

# stETH

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

# Lido

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

## View Methods

### name()

Returns the name of the token

```sol
function name() returns (string)
```

### symbol()

Returns the symbol of the token, usually a shorter version of the name

```sol
function symbol() returns (string)
```

### decimals()

Returns the number of decimals for getting user representation of a token amount.

```sol
function decimals() returns (uint8)
```

### totalSupply()

Returns the amount of tokens in existence.

```sol
function totalSupply() returns (uint256)
```


**Note:** Always equals to `getTotalPooledEther()` since token amount
is pegged to the total amount of Ether controlled by the protocol.


### getTotalPooledEther()

Returns the entire amount of Ether controlled by the protocol

```sol
function getTotalPooledEther() returns (uint256)
```


**Note:** The sum of all ETH balances in the protocol, equals to the total supply of stETH.


### balanceOf()

Returns the amount of tokens owned by the `_account`

```sol
function balanceOf(address _account) returns (uint256)
```


**Note:** Balances are dynamic and equal the `_account`'s share in the amount of the
total Ether controlled by the protocol. See `sharesOf`.


### getTotalShares()

Returns the total amount of shares in existence.

```sol
function getTotalShares() returns (uint256)
```

### sharesOf()

Returns the amount of shares owned by `_account`

```sol
function sharesOf(address _account) returns (uint256)
```

### getSharesByPooledEth()

Returns the amount of shares that corresponds to `_ethAmount` protocol-controlled Ether

```sol
function getSharesByPooledEth(uint256 _ethAmount) returns (uint256)
```

### getPooledEthByShares()

Returns the amount of Ether that corresponds to `_sharesAmount` token shares

```sol
function getPooledEthByShares(uint256 _sharesAmount) returns (uint256)
```

### getFee()

Returns staking rewards fee rate

```sol
function getFee() returns (uint16)
```

#### Returns:

Fee in basis points. 10000 BP corresponding to 100%.

### getFeeDistribution()

Returns fee distribution proportion

```sol
function getFeeDistribution() returns (
  uint16 treasuryFeeBasisPoints,
  uint16 insuranceFeeBasisPoints,
  uint16 operatorsFeeBasisPoints
)
```

#### Returns:

| Name                      | Type     | Description                                                                            |
| ------------------------- | -------- | -------------------------------------------------------------------------------------- |
| `treasuryFeeBasisPoints`  | `uint16` | Fee for the treasury. Expressed in basis points, 10000 BP corresponding to 100%.       |
| `insuranceFeeBasisPoints` | `uint16` | Fee for the insurance fund. Expressed in basis points, 10000 BP corresponding to 100%. |
| `operatorsFeeBasisPoints` | `uint16` | Fee for the node operators. Expressed in basis points, 10000 BP corresponding to 100%. |

### getWithdrawalCredentials()

Returns current credentials to withdraw ETH on ETH 2.0 side after the phase 2 is launched

```sol
function getWithdrawalCredentials() returns (bytes32)
```

### getBufferedEther()

Get the amount of Ether temporary buffered on this contract balance


**Note:** Buffered balance is kept on the contract from the moment the funds are received from user
until the moment they are actually sent to the official Deposit contract.



```sol
function getBufferedEther()  returns (uint256)
```

#### Returns:

Amount of buffered funds in wei

### getDepositContract()

Gets deposit contract handle

```sol
function getDepositContract() public view returns (IDepositContract)
```

#### Returns:

Address of deposit contract

### getOracle()

Returns authorized oracle address

```sol
function getOracle() returns (address)
```

### getOperators()

Gets node operators registry interface handle

```sol
function getOperators() returns (INodeOperatorsRegistry)
```

#### Returns:

Address of NodeOperatorsRegistry contract

### getTreasury()

Returns the treasury address

```sol
function getTreasury() returns (address)
```

### getInsuranceFund()

Returns the insurance fund address

```sol
function getInsuranceFund() returns (address)
```

### getBeaconStat()

Returns the key values related to Beacon-side

```sol
function getBeaconStat() returns (
  uint256 depositedValidators,
  uint256 beaconValidators,
  uint256 beaconBalance
)
```

#### Returns:

| Name                  | Type      | Description                                                                    |
| --------------------- | --------- | ------------------------------------------------------------------------------ |
| `depositedValidators` | `uint256` | Number of deposited validators                                                 |
| `beaconValidators`    | `uint256` | Number of Lido's validators visible in the Beacon state, reported by oracles   |
| `beaconBalance`       | `uint256` | Total amount of Beacon-side Ether (sum of all the balances of Lido validators) |

## Methods

### transfer()

Moves `_amount` tokens from the caller's account to the `_recipient` account.

```sol
function transfer(address _recipient, uint256 _amount) returns (bool)
```


Requirements:

- `_recipient` cannot be the zero address.
- the caller must have a balance of at least `_amount`.
- the contract must not be paused.


#### Parameters:

| Name         | Type      | Description                  |
| ------------ | --------- | ---------------------------- |
| `_recipient` | `address` | Address of tokens recipient  |
| `_amount`    | `uint256` | Amount of tokens to transfer |

#### Returns:

A boolean value indicating whether the operation succeeded.

### allowance()

Returns the remaining number of tokens that `_spender` is allowed to spend
on behalf of `_owner` through `transferFrom`. This is zero by default.

```sol
function allowance(address _owner, address _spender) returns (uint256)
```


This value changes when `approve` or `transferFrom` is called.


#### Parameters:

| Name       | Type      | Description        |
| ---------- | --------- | ------------------ |
| `_owner`   | `address` | Address of owner   |
| `_spender` | `address` | Address of spender |

### approve()

Sets `_amount` as the allowance of `_spender` over the caller's tokens

```sol
function approve(address _spender, uint256 _amount) returns (bool)
```


Requirements:

- `_spender` cannot be the zero address.
- the contract must not be paused.



#### Parameters:

| Name       | Type      | Description        |
| ---------- | --------- | ------------------ |
| `_spender` | `address` | Address of spender |
| `_amount`  | `uint256` | Amount of tokens   |

#### Returns:

A boolean value indicating whether the operation succeeded

### transferFrom()

Moves `_amount` tokens from `_sender` to `_recipient` using the
allowance mechanism. `_amount` is then deducted from the caller's
allowance.

```sol
function transferFrom(
  address _sender,
  address _recipient,
  uint256 _amount
) returns (bool)
```



Requirements:

- `_sender` and `_recipient` cannot be the zero addresses.
- `_sender` must have a balance of at least `_amount`.
- the caller must have allowance for `_sender`'s tokens of at least `_amount`.
- the contract must not be paused.



#### Parameters:

| Name         | Type      | Description          |
| ------------ | --------- | -------------------- |
| `_sender`    | `address` | Address of spender   |
| `_recipient` | `address` | Address of recipient |
| `_amount`    | `uint256` | Amount of tokens     |

#### Returns:

A boolean value indicating whether the operation succeeded

### increaseAllowance()

Atomically increases the allowance granted to `_spender` by the caller by `_addedValue`

This is an alternative to `approve` that can be used as a mitigation for problems described [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol#L42)

```sol
function increaseAllowance(address _spender, uint256 _addedValue) returns (bool)
```



Requirements:

- `_spender` cannot be the the zero address.
- the contract must not be paused.


#### Parameters:

| Name          | Type      | Description                            |
| ------------- | --------- | -------------------------------------- |
| `_sender`     | `address` | Address of spender                     |
| `_addedValue` | `uint256` | Amount of tokens to increase allowance |

#### Returns:

Returns a boolean value indicating whether the operation succeeded

### decreaseAllowance()

Atomically decreases the allowance granted to `_spender` by the caller by `_subtractedValue`

This is an alternative to `approve` that can be used as a mitigation for
problems described [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol#L42)

```sol
function decreaseAllowance(address _spender, uint256 _subtractedValue) returns (bool)
```


Requirements:

- `_spender` cannot be the zero address.
- `_spender` must have allowance for the caller of at least `_subtractedValue`.
- the contract must not be paused.



#### Parameters:

| Name               | Type      | Description                            |
| ------------------ | --------- | -------------------------------------- |
| `_sender`          | `address` | Address of spender                     |
| `_subtractedValue` | `uint256` | Amount of tokens to decrease allowance |

#### Returns:

Returns a boolean value indicating whether the operation succeeded

### submit()

Send funds to the pool with optional \_referral parameter

```sol
function submit(address _referral) returns (uint256)
```

#### Parameters:

| Name        | Type      | Description               |
| ----------- | --------- | ------------------------- |
| `_referral` | `address` | Optional referral address |

#### Returns:

Amount of StETH shares generated

### depositBufferedEther()

Deposits buffered ethers to the official DepositContract. If `_maxDeposits` provided makes no more than `_maxDeposits` deposit calls

```sol
function depositBufferedEther()
function depositBufferedEther(uint256 _maxDeposits)
```

#### Parameters:

| Name           | Type      | Description                 |
| -------------- | --------- | --------------------------- |
| `_maxDeposits` | `uint256` | Number of max deposit calls |

### burnShares()

Destroys `_sharesAmount` shares from `_account`'s holdings, decreasing the total amount of shares.

```sol
function burnShares(
  address _account,
  uint256 _sharesAmount
) returns (uint256 newTotalShares)
```


**Note:** This doesn't decrease the token total supply.

Requirements:

- `_account` cannot be the zero address.
- `_account` must hold at least `_sharesAmount` shares.
- the contract must not be paused.

:::

#### Parameters

| Name            | Type      | Description                         |
| --------------- | --------- | ----------------------------------- |
| `_account`      | `address` | Address where shares will be burned |
| `_sharesAmount` | `uint256` | Amount of shares to burn            |

#### Returns

Amount of totalShares after tokens burning

### stop()

Stop pool routine operations

```sol
function stop()
```

### resume()

Resume pool routine operations

```sol
function resume()
```

### setFee()

Set fee rate to `_feeBasisPoints` basis points. The fees are accrued when oracles report staking results

```sol
function setFee(uint16 _feeBasisPoints)
```

#### Parameters

| Name              | Type     | Description                                                    |
| ----------------- | -------- | -------------------------------------------------------------- |
| `_feeBasisPoints` | `uint16` | Fee expressed in basis points, 10000 BP corresponding to 100%. |

### setFeeDistribution()

Set fee distribution: `_treasuryFeeBasisPoints` basis points go to the treasury,
`_insuranceFeeBasisPoints` basis points go to the insurance fund,
`_operatorsFeeBasisPoints` basis points go to node operators.
The sum has to be 10 000.

```sol
function setFeeDistribution(
  uint16 _treasuryFeeBasisPoints,
  uint16 _insuranceFeeBasisPoints,
  uint16 _operatorsFeeBasisPoints
)
```

#### Parameters

| Name                       | Type     | Description                                                                            |
| -------------------------- | -------- | -------------------------------------------------------------------------------------- |
| `_treasuryFeeBasisPoints`  | `uint16` | Fee for the treasury. Expressed in basis points, 10000 BP corresponding to 100%.       |
| `_insuranceFeeBasisPoints` | `uint16` | Fee for the insurance fund. Expressed in basis points, 10000 BP corresponding to 100%. |
| `_operatorsFeeBasisPoints` | `uint16` | Fee for the node operators. Expressed in basis points, 10000 BP corresponding to 100%. |

### setOracle()

Set authorized oracle contract address to `_oracle`

```sol
function setOracle(address _oracle)
```

#### Parameters

| Name      | Type      | Description                |
| --------- | --------- | -------------------------- |
| `_oracle` | `address` | Address of oracle contract |

### setTreasury()

Set treasury contract address to `_treasury`. This contract is used to accumulate the protocol treasury fee

```sol
function setTreasury(address _treasury)
```

#### Parameters

| Name        | Type      | Description                                        |
| ----------- | --------- | -------------------------------------------------- |
| `_treasury` | `address` | Address of contract which accumulates treasury fee |

### setInsuranceFund()

Set insuranceFund contract address to `_insuranceFund`.
This contract is used to accumulate the protocol insurance fee

```sol
function setInsuranceFund(address _insuranceFund)
```

#### Parameters

| Name             | Type      | Description                                         |
| ---------------- | --------- | --------------------------------------------------- |
| `_insuranceFund` | `address` | Address of contract which accumulates insurance fee |

### setWithdrawalCredentials()

Set credentials to withdraw ETH on ETH 2.0 side after the phase 2 is launched to `_withdrawalCredentials`

```sol
function setWithdrawalCredentials(bytes32 _withdrawalCredentials)
```


**Note:** Note that `setWithdrawalCredentials` discards all unused signing keys as the signatures are invalidated.


#### Parameters

| Name                     | Type      | Description                                                                                 |
| ------------------------ | --------- | ------------------------------------------------------------------------------------------- |
| `_withdrawalCredentials` | `bytes32` | Hash of withdrawal multisignature key as accepted by the deposit_contract.deposit functione |

### withdraw()

Issues withdrawal request. **Not implemented.**

```sol
function withdraw(uint256 _amount, bytes32 _pubkeyHash)
```


**Note:** Will be upgraded to an actual implementation when withdrawals are enabled (Phase 1.5 or 2 of Eth2 launch, likely late 2021 or 2022). At the moment withdrawals are not possible in the beacon chain and there's no workaround


#### Parameters

| Name          | Type      | Description                 |
| ------------- | --------- | --------------------------- |
| `_amount`     | `uint256` | Amount of StETH to withdraw |
| `_pubkeyHash` | `bytes32` | Receiving address           |

### pushBeacon()

Updates the number of Lido-controlled keys in the beacon validators set and their total balance.

```sol
function pushBeacon(uint256 _beaconValidators, uint256 _beaconBalance)
```

#### Parameters

| Name                | Type      | Description                                       |
| ------------------- | --------- | ------------------------------------------------- |
| `_beaconValidators` | `uint256` | Number of Lido's keys in the beacon state         |
| `_beaconBalance`    | `uint256` | Summarized balance of Lido-controlled keys in wei |

### transferToVault()

Send funds to recovery Vault. Overrides default AragonApp behaviour.

```sol
function transferToVault(address _token)
```

#### Parameters

| Name     | Type      | Description                        |
| -------- | --------- | ---------------------------------- |
| `_token` | `address` | Token to be sent to recovery vault |


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


## Add calculation of staker rewards [APR][1]

To calculate the percentage of rewards for stakers, we store and provide the following data:

- `preTotalPooledEther` - total pooled ether mount, queried right before every report push to the
  [Lido contract][6],
- `postTotalPooledEther` - the same, but queried right after the push,
- `lastCompletedEpochId` - the last epoch that we pushed the report to the Lido,
- `timeElapsed` - the time in seconds between the current epoch of push and the
  `lastCompletedEpochId`. Usually, it should be a frame long: 32 _ 12 _ 225 = 86400, but maybe
  multiples more in case that the previous frame didn't reach the quorum.


**Note:** It is important to note here, that we collect post/pre pair (not current/last), to avoid
the influence of new staking during the epoch.

To calculate the APR, use the following formula:

    APR = (postTotalPooledEther - preTotalPooledEther) * secondsInYear / (preTotalPooledEther * timeElapsed)

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

## Receiver function to be invoked on report pushes

To provide the external contract with updates on report pushes (every time the quorum is reached
among oracle daemons data), we provide the following setter and getter functions. It might be
needed to implement some updates to the external contracts that should happen at the same tx the
[rebase](/contracts/lido#rebasing) happens (e.g. adjusting uniswap v2 pools to reflect the
rebase).

And when the callback is set, the following function will be invoked on every report push.

    interface IBeaconReportReceiver {
        function processLidoOracleReport(uint256 _postTotalPooledEther,
                                         uint256 _preTotalPooledEther,
                                         uint256 _timeElapsed) external;
    }

The arguments provided are the same as described in section [above][3].

## View Methods

### getLido()

Return the [Lido contract][6] address.

```sol
function getLido() returns (ILido)
```

### getQuorum()

Return the number of exactly the same reports needed to finalize the epoch.

```sol
function getQuorum() returns (uint256)
```

### getAllowedBeaconBalanceAnnualRelativeIncrease()

Return the upper bound of the reported balance possible increase in [APR][1]. See above about
[sanity checks][4].

```sol
function getAllowedBeaconBalanceAnnualRelativeIncrease() returns (uint256)
```

### getAllowedBeaconBalanceRelativeDecrease()

Return the lower bound of the reported balance possible decrease. See above about [sanity
checks][4].

```sol
function getAllowedBeaconBalanceRelativeDecrease() returns (uint256)
```

### getBeaconReportReceiver()

Return the receiver contract address to be called when the report is pushed to [Lido][6].

```sol
function getBeaconReportReceiver() returns (address)
```

### getCurrentOraclesReportStatus()

Return the current reporting bitmap, representing oracles who have already pushed their version of
report during the expected epoch.

**Note: Every oracle bit corresponds to the index of the oracle in the current members list

```sol
function getCurrentOraclesReportStatus() returns (uint256)
```

### getCurrentReportVariantsSize()

Return the current reporting variants array size.

```sol
function getCurrentReportVariantsSize() returns (uint256)
```

### getCurrentReportVariant()

Return the current reporting array element with index `_index`.

```sol
function getCurrentReportVariant(uint256 _index)
```

### getExpectedEpochId()

Return epoch that can be reported by oracles.

```sol
function getExpectedEpochId() returns (uint256)
```

### getOracleMembers()

Return the current oracle member committee list.

```sol
function getOracleMembers() returns (address[])
```

### getVersion()

Return the initialized version of this contract starting from 0.

```sol
function getVersion() returns (uint256)
```

### getBeaconSpec()

Return beacon specification data.

```sol
function getBeaconSpec()
    returns (
        uint64 epochsPerFrame,
        uint64 slotsPerEpoch,
        uint64 secondsPerSlot,
        uint64 genesisTime
    )
```

### getCurrentEpochId()

Return the epoch calculated from current timestamp.

```sol
function getCurrentEpochId() returns (uint256)
```

### getCurrentFrame()

Return currently reportable epoch (the first epoch of the current frame) as well as its start and
end times in seconds.

```sol
function getCurrentFrame()
    returns (
        uint256 frameEpochId,
        uint256 frameStartTime,
        uint256 frameEndTime
    )
```

### getLastCompletedEpochId()

Return last completed epoch.

```sol
function getLastCompletedEpochId() returns (uint256)
```

### getLastCompletedReportDelta()

Report beacon balance and its change during the last frame.

```sol
function getLastCompletedReportDelta()
    returns (
        uint256 postTotalPooledEther,
        uint256 preTotalPooledEther,
        uint256 timeElapsed
    )
```

## Methods

### setAllowedBeaconBalanceAnnualRelativeIncrease()

Set the upper bound of the reported balance possible increase in APR to `_value`. See above about
[sanity checks][4].

```sol
function setAllowedBeaconBalanceAnnualRelativeIncrease(uint256 _value) auth(SET_REPORT_BOUNDARIES)
```

### setAllowedBeaconBalanceRelativeDecrease()

Set the lower bound of the reported balance possible decrease to `_value`. See above about [sanity
checks][4].

```sol
function setAllowedBeaconBalanceRelativeDecrease(uint256 _value) auth(SET_REPORT_BOUNDARIES)
```

### setBeaconReportReceiver()

Set the receiver contract address to `_addr` to be called when the report is pushed.

:::note
Specify 0 to disable this functionality.
:::

```sol
function setBeaconReportReceiver(address _addr) auth(SET_BEACON_REPORT_RECEIVER)
```

### setBeaconSpec()

Update beacon specification data.

```sol
function setBeaconSpec(
    uint64 _epochsPerFrame,
    uint64 _slotsPerEpoch,
    uint64 _secondsPerSlot,
    uint64 _genesisTime
    )
    auth(SET_BEACON_SPEC)
```

### initialize_v2()

Initialize the contract v2 data, with sanity check bounds
(`_allowedBeaconBalanceAnnualRelativeIncrease`, `_allowedBeaconBalanceRelativeDecrease`).

:::note
Public function `initialize` was removed from v2 because it is not needed once the contract is
initialized for the first time, that happened in v1. Instead we added `initialize_v2` function that
initializes newly added variables, updates the contract version to 1.
:::

```sol
function initialize_v2(
    uint256 _allowedBeaconBalanceAnnualRelativeIncrease,
    uint256 _allowedBeaconBalanceRelativeDecrease
)
```

### addOracleMember()

Add `_member` to the oracle member committee list.

```sol
function addOracleMember(address _member) auth(MANAGE_MEMBERS)
```

### removeOracleMember()

Remove '\_member` from the oracle member committee list.

```sol
function removeOracleMember(address _member) auth(MANAGE_MEMBERS)
```

### setQuorum()

Set the number of exactly the same reports needed to finalize the epoch to `_quorum`.

```sol
function setQuorum(uint256 _quorum) auth(MANAGE_QUORUM)
```

### reportBeacon()

Accept oracle committee member reports from the ETH 2.0 side. Parameters:

- `_epochId` - beacon chain epoch
- `_beaconBalance` - balance in gwei on the ETH 2.0 side (9-digit denomination)
- `_beaconValidators` - number of validators visible in this epoch

```sol
function reportBeacon(uint256 _epochId, uint64 _beaconBalance, uint32 _beaconValidators)
```
# StableSwapStateOracle

- [Source Code](https://github.com/lidofinance/curve-merkle-oracle/blob/main/contracts/StableSwapStateOracle.sol)
- [Deployed Contract](https://etherscan.io/address/0x3a6bd15abf19581e411621d669b6a2bbe741ffd6)

A trustless oracle for the ETH/stETH Curve pool using Merkle Patricia proofs of Ethereum state.

Contract receives and verifies the report from the offchain code,
and persists the verified state along with its timestamp.

The oracle assumes that the pool's `fee` and `A` (amplification coefficient) values don't
change between the time of proof generation and submission.

## Mechanics

The oracle works by verifying Merkle Patricia proofs of the following Ethereum state:

- Curve stETH/ETH pool contract account and the following slots from its storage trie:

  - `admin_balances[0]`
  - `admin_balances[1]`

- stETH contract account and the following slots from its storage trie:
  - `shares[0xDC24316b9AE028F1497c275EB9192a3Ea0f67022]`
  - `keccak256("lido.StETH.totalShares")`
  - `keccak256("lido.Lido.beaconBalance")`
  - `keccak256("lido.Lido.bufferedEther")`
  - `keccak256("lido.Lido.depositedValidators")`
  - `keccak256("lido.Lido.beaconValidators")`

## View Methods

### getState()

```sol
function getState() external view returns (
  uint256 _timestamp,
  uint256 _etherBalance,
  uint256 _stethBalance,
  uint256 _stethPrice
)
```

Returns current state of oracle

| Return Parameter | Type      | Description                                  |
| ---------------- | --------- | -------------------------------------------- |
| `_timestamp`     | `uint256` | The timestamp of the proven pool state/price |
| `_etherBalance`  | `uint256` | The proven ETH balance of the pool           |
| `_stethBalance`  | `uint256` | The proven stETH balance of the pool         |
| `_stethPrice`    | `uint256` | The proven stETH/ETH price in the pool       |

### getProofParams()

```sol
function getProofParams() external view returns (
  address poolAddress,
  address stethAddress,
  bytes32 poolAdminEtherBalancePos,
  bytes32 poolAdminCoinBalancePos,
  bytes32 stethPoolSharesPos,
  bytes32 stethTotalSharesPos,
  bytes32 stethBeaconBalancePos,
  bytes32 stethBufferedEtherPos,
  bytes32 stethDepositedValidatorsPos,
  bytes32 stethBeaconValidatorsPos,
  uint256 advisedPriceUpdateThreshold
)
```

Returns values of proof params

## Methods

### setAdmin()

```sol
function setAdmin(address _admin) external
```

Passes the right to set the suggested price update threshold to a new address.

| Parameter Name | type      | Description       |
| -------------- | --------- | ----------------- |
| `_admin`       | `address` | New admin address |

### setPriceUpdateThreshold()

```sol
function setPriceUpdateThreshold(uint256 _priceUpdateThreshold) external
```

Sets the suggested price update threshold.

| Parameter Name          | type      | Description                                                                                      |
| ----------------------- | --------- | ------------------------------------------------------------------------------------------------ |
| `_priceUpdateThreshold` | `uint256` | The suggested price update threshold. Expressed in basis points, 10000 BP corresponding to 100%. |

### submitState()

```sol
function submitState(
  bytes memory _blockHeaderRlpBytes,
  bytes memory _proofRlpBytes
) external
```

Used by the offchain clients to submit the proof


**Note:** Reverts unless:

- the block the submitted data corresponds to is in the chain;
- the block is at least `MIN_BLOCK_DELAY` blocks old
- all submitted proofs are valid



| Parameter Name         | type    | Description                                 |
| ---------------------- | ------- | ------------------------------------------- |
| `_blockHeaderRlpBytes` | `bytes` | RLP-encoded block header                    |
| `_proofRlpBytes`       | `bytes` | RLP-encoded list of Merkle Patricia proofs. |

`_proofRlpBytes` contains next encoded variables in exact order:

1. proof of the Curve pool contract account;
2. proof of the stETH contract account;
3. proof of the `admin_balances[0]` slot of the Curve pool contract;
4. proof of the `admin_balances[1]` slot of the Curve pool contract;
5. proof of the `shares[0xDC24316b9AE028F1497c275EB9192a3Ea0f67022]` slot of stETH contract;
6. proof of the `keccak256("lido.StETH.totalShares")` slot of stETH contract;
7. proof of the `keccak256("lido.Lido.beaconBalance")` slot of stETH contract;
8. proof of the `keccak256("lido.Lido.bufferedEther")` slot of stETH contract;
9. proof of the `keccak256("lido.Lido.depositedValidators")` slot of stETH contract;
10. proof of the `keccak256("lido.Lido.beaconValidators")` slot of stETH contract.

# StEthPriceFeed

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

## View Methods

### safe_price()

Returns the cached safe price and its timestamp.

```
safe_price() -> (price: uint256, timestamp: uint256)
```

### current_price()

Returns the current pool price and whether the price is safe.

```
current_price() -> (price: uint256, is_safe: bool)
```

## Methods

### update_safe_price()

Sets the cached safe price to the `max(current pool price, 1)` given that the latter is safe.

```
update_safe_price() -> uint256
```

### fetch_safe_price()

Returns the cached safe price and its timestamp. Calls `update_safe_price()` prior to that if
the cached safe price is older than `max_age` seconds.

```
fetch_safe_price(max_age: uint256) -> (price: uint256, timestamp: uint256)
```

#### Parameters:

| Name      | Type      | Description                                                       |
| --------- | --------- | ----------------------------------------------------------------- |
| `max_age` | `uint256` | Amount of seconds last value of safe price considered to be valid |



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
