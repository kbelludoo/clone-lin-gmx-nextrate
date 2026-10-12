# clone-lin-gmx-nextrate

Experimental i64-scale LIN clone of [gmx-io/gmx-contracts](https://github.com/gmx-io/gmx-contracts) Vault **getNextFundingRate** (`factor * reserved * intervals / poolAmount`).

- **LIN clone:** [`src/lin_gmx_nextrate.lin`](src/lin_gmx_nextrate.lin)
- **C11 oracle:** [`test/oracles/gmx_nextrate_c11.c`](test/oracles/gmx_nextrate_c11.c)
- **Proofs (results only) live in lin-open:** https://github.com/kbelludoo/lin-open
- **Event receipt:** `docs/events/EVENT_GMX_NEXTRATE_CLONE_LIN.rulel` in lin-open

This repository is the standalone LIN copy. It is **not** a GMX Vault, **not** GLP, and does **not** claim Solidity uint256 / mainnet parity. The kernel is the GMX V1 **time-sliced utilization funding increment**: `factor * reserved * ((now-last)/fundingInterval) / pool`. If `last + interval > now` the increment is 0. Companion `getUtilisation` is `reserved * 1e6 / pool`. Confusing next-rate with raw elapsed seconds overcharges **28800x**. Confusing it with `getFundingFee` `size*rate/1e6` is **100x**. It is **not** GMX `getFundingFee`, **not** GMX `getFeeBasisPoints` USDG tax/rebate, **not** Uniswap V2 `xy`, **not** Solidly cubic `x3y+y3x`, **not** Saddle/Curve `getD`, and **not** Maker `rpow`.

Upstream is MIT. This LIN kernel is an original uint64-scale implementation of the published `getNextFundingRate` formula, not a verbatim Solidity dump. `getNextFundingRate` / `lastFundingTimes` / `stableFundingRateFactor` / `getUtilisation` are rejected by the Solidity transpiler (`REJ_SOLIDITY_GMX_NEXTRATE`).

Pinned upstream: commit `fe55c4ba8911ae2378b50bfb9bece5ee50cf2af8`, `contracts/core/Vault.sol` sha256 `0b49e265fc4c04d3eea74b07d698748050671a2df259f1faa8d05cabf2970f92` blob `4aef3c7dcf40b4657e7dfcef582958dc2d418126`.

Canonical goldens (uint64-scale, 128-bit product, 50% reserved/pool, factor=100, interval=28800):

- 1 interval: nextRate **50**
- elapsed 20000 < 28800: **0** vs ignore-interval **50**
- 24h: intervals=3 rate **150** vs raw elapsed seconds **4320000** (28800x)
- stable factor=10: **5** vs volatile **50** (10x)
- `getUtilisation`: **500000** vs nextRate **50** (10000x)
- confused `getFundingFee` reserved*factor/1e6: **5000** vs **50** (100x)
- uninit last=0 at now=1700000000: **2951350** vs initialized **0**
- naive i64 `(4e9*3e9)/2e9` wraps to **-3223372036**; 128-bit muldiv = **6000000000**
