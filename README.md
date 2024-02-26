# Beanstalk Codehawks
<p align="center">
<img src="https://res.cloudinary.com/droqoz7lg/image/upload/q_90/dpr_2.0/c_fill,g_auto,h_320,w_320/f_auto/v1/company/fsv4gpiuvkthl27oygeh?_a=BATAUVAA0" width="500" alt="Beanstalk">
</p>

# Contest Details

### Prize Pool

- Total Pool - $100,000
- H/M - $95,000
- Low - $5,000
- Starts: Monday, February 26, 2024
- Ends: Monday, March 25, 2024

### Stats

- nSLOC: 5776
- Complexity Score: 3356
- $/nSLOC: $15.58
- $/Complexity: $26.82

## About

Beanstalk is a permissionless fiat stablecoin protocol built on Ethereum. Its primary objective is to incentivize independent market participants to regularly cross the price of 1 Bean over its dollar peg in a sustainable fashion.

Beanstalk does not have any collateral requirements. Beanstalk uses credit instead of collateral to create Bean price stability relative to its value peg of $1. The practicality of using DeFi is currently limited by the lack of decentralized low-volatility assets with competitive carrying costs. Borrowing rates on USD stablecoins have historically been higher than borrowing rates on USD, even when supply increases rapidly. Non-competitive carrying costs are due to collateral requirements.

In particular, the [Sun](https://docs.bean.money/almanac/farm/sun) and [Silo](https://docs.bean.money/almanac/farm/silo) components of Beanstalk are in scope of this audit. The code in the repository includes the Seed Gauge System upgrade, which upgrades the Silo to autonomously adjust the Grown Stalk issued to holders of various whitelisted tokens. An overview of the Gauge System can be read [here](https://github.com/BeanstalkFarms/Beanstalk/issues/726).

You can read an overview of how Beanstalk works [here](https://docs.bean.money/almanac/introduction/how-beanstalk-works).

* [Docs](https://docs.bean.money/)
* [Whitepaper](https://bean.money/beanstalk.pdf)
* [Website](https://bean.money/)
* [Beanstalk Farms Twitter](https://twitter.com/BeanstalkFarms)
* [Beanstalk Public GitHub Repo](https://github.com/BeanstalkFarms/Beanstalk)

## Actors

* Stalkholder / Silo Member
    * Anyone who Deposits assets on the Deposit Whitelist into the Silo, earning the illiquid Stalk token in doing so. Stalkholders participate in governance and earn Bean seigniorage.
* `gm` caller
    * Anyone who calls the `gm` function to start the next Season.
* Unripe holder
    * Anyone who holds Unripe Beans or Unripe LP. These assets were distributed to holders of BDV (Bean Denominated Value) at the time of the April 2022 governance exploit. Most Unripe holders have their Unripe assets Deposited in the Silo, and thus are also Stalkholders. Somewhat relevant to this audit.
* Fertilizer holder
    * Anyone who holds Fertilizer, the debt asset earned by participating in Beanstalk's recapitalization. Not particularly relevant for the scope of this audit.
* Pod holder
    * Anyone who holds Pods, the Beanstalk-native debt asset. Pods are minting when lending Beans to Beanstalk (Sowing Beans). Not particularly relevant for the scope of this audit.

## Scope

Generally, the audit covers the Silo, the Sun and many of their associated libraries. A couple contracts from the Barn (related to Unripe assets) are also in scope. 

Specifically, only the following contracts are in scope.

```js
protocol/
└── contracts/
    ├── beanstalk/
    │   ├── AppStorage.sol
    │   ├── barn/
    │   │   └── UnripeFacet.sol
    │   ├── init/
    │   │   ├── InitBipSeedGauge.sol
    │   │   └── InitWhitelistStatuses.sol
    │   ├── silo/
    │   │   ├── BDVFacet.sol
    │   │   ├── ConvertFacet.sol
    │   │   ├── EnrootFacet.sol
    │   │   ├── MigrationFacet.sol
    │   │   ├── SiloFacet/
    │   │   │   ├── Silo.sol
    │   │   │   ├── SiloFacet.sol
    │   │   │   ├── SiloGettersFacet.sol
    │   │   │   └── TokenSilo.sol
    │   │   └── WhitelistFacet/
    │   │       ├── WhitelistedFacet.sol
    │   │       └── WhitelistedTokens.sol
    │   └── sun/ 
    │       ├── GaugePointFacet.sol
    │       ├── LiquidityWeightFacet.sol
    │       └── SeasonFacet/
    │           ├── Oracle.sol
    │           ├── SeasonFacet.sol
    │           ├── SeasonGettersFacet.sol
    │           ├── Sun.sol
    │           └── Weather.sol
    ├── libraries/
    │   ├── Convert/ 
    │   │   ├── LibChopConvert.sol
    │   │   ├── LibConvert.sol
    │   │   ├── LibConvertData.sol
    │   │   ├── LibLambdaConvert.sol
    │   │   ├── LibUnripeConvert.sol 
    │   │   └── LibWellConvert.sol 
    │   ├── LibCases.sol
    │   ├── LibChop.sol
    │   ├── LibEvaluate.sol
    │   ├── LibFertilizer.sol
    │   ├── LibGauge.sol 
    │   ├── LibIncentive.sol 
    │   ├── LibLockedUnderlying.sol
    │   ├── LibUnripe.sol
    │   ├── Minting/ 
    │   │   └── LibWellMinting.sol
    │   ├── Oracle/ 
    │   │   ├── LibChainlinkOracle.sol
    │   │   ├── LibEthUsdOracle.sol
    │   │   └── LibUsdOracle.sol
    │   ├── Silo/
    │   │   ├── LibGerminate.sol
    │   │   ├── LibLegacyTokenSilo.sol
    │   │   ├── LibSilo.sol
    │   │   ├── LibTokenSilo.sol
    │   │   ├── LibUnripeSilo.sol
    │   │   ├── LibWhitelist.sol
    │   │   └── LibWhitelistedTokens.sol
    │   └── Well/
    │       └── LibWell.sol
    └── pipeline/
        └── junctions/
            └── UnwrapAndSendETH.sol
    
```

## Compatibilities

Beanstalk implements the [ERC-2535 Diamond standard](https://docs.bean.money/developers/overview/eip-2535-diamond). It supports various whitelists for [Deposits](https://docs.bean.money/almanac/farm/silo#deposit-whitelist), [Minting](https://docs.bean.money/almanac/farm/sun#minting-whitelist), [Converts](https://docs.bean.money/almanac/peg-maintenance/convert#convert-whitelist), etc., particularly for LP tokens from [Basin](https://basin.exchange/).

Blockchains:
* Ethereum

Tokens:
* ERC-20 (all are accepted in Farm balances, a whitelist is accepted on the Deposit Whitelist, etc.)
* ERC-1155 (Fertilizer and Deposits are ERC-1155 tokens)

## Setup

Clone repo: 

```bash
git clone https://github.com/Cyfrin/2024-02-Beanstalk-1
```
Install dependencies: 
```bash
cd Beanstalk/protocol
yarn
```
Add RPC:
```bash
export FORKING_RPC=https://eth-mainnet.g.alchemy.com/v2/{RPC_KEY}
```

Build: 
```bash
npx hardhat compile
```
Test: 
```bash
npx hardhat test
```

## Known Issues

* The `enrootDeposits` functions do not properly emit ERC-1155 events.
    * `enrootDeposits` updates a user's Unripe Deposits' BDV and issues the corresponding Stalk to the user. The single `enrootDeposit` function correctly emits the ERC-1155 events, but the multiple variant incorrectly emits a `transferSingle` event to the 0 address for each Deposit. Given the Beanstalk subgraph does not use these events, and cannot be used to harm the protocol, the fix will be implmented in a separate upgrade to Beanstalk.

* The  `SeasonFacet` contract is known to be too large to deploy on mainnet (due to `LibGerminate`). This will be fixed before the Seed Gauge System is deployed

* All findings in the following audit reports
    * [Cyfrin's initial audit report of v0 of the Gauge System](https://arweave.net/tfK_IQlxz1lABDEq4aefN9gPQaynKZKYFvFyU8seYA8) (the version of the Gauge System in this Codehawks audit is substantially different, hence the need for another audit);
    * [Cyfrin's initial Beanstalk report](https://arweave.net/JQodlB-9fil-OWfWOwYy6Q8eqWITJXtyaN5z_Anq1S0);
    * All Beanstalk audit reports listed in this [repository](https://github.com/BeanstalkFarms/Beanstalk-Audits); and
    * All bug reports from the Immunefi program listed [here](https://community.bean.money/bug-reports).

**Additional Known Issues as outlined here: [Additional Known Issues](https://github.com/Cyfrin/2024-02-Beanstalk-1/issues/1)**
