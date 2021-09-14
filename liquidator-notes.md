# Mango liquidator

Mango markets has open sourced their liquidator [here](https://gitlab.com/OpinionatedGeek/mango-explorer/-/blob/master/Quickstart.md). 

Build a mango.markets liquidator [here](https://docs.mango.markets/development-resources/liquidator). No flash loans available but negligible gas costs.

Currently just uses spot balance split 3 ways (i.e. 1000 USDT is split between BTC/ETH/USDT). This means that liquidations can only occur when amount to repay < 333~ USD worth of each asset. Issue here is you would like to liquidate accounts w/ a higher balance, netting you a higher $ profit. One way to get around that would be to specialize in one asset (i.e. only do ETH liquidations).
Another way would be to take out a margin loan (5x available through mango.markets), repay the borrow, and then remove margin loan. This would likely require more work but could be far more profitable. Looking now this seems like it won't work, since in order to use mango.markets (and borrow from them) you have to make an account and your funds will be held by them in a margin account. You can't liquidate other margin accounts from a margin account itself.

Current thinking: keep a USDC only liquidator that is rebalanced 100% to USDC (because majority of margin is in USDC based on [this page](https://trade.mango.markets/stats)). *Correction*, the # of deposits

[Solaris](https://solarisprotocol.com/) is looking to build borrowing/flashloans on top of Solana but is in super alpha right now so unusable.

### Questions
- the docs are for USDT, while mango.markets seems to work with USDC. Would we have to change the code to account for this? Similarly, to account for SOL.
	- Dev replied [here](https://twitter.com/OpinionatedGeek/status/1412783704942972929). TL;DR is code has to be slightly adapted for mango.markets v2 but shouldn't be too difficult
- borrows are highest in USDC by far, but deposits in BTC, ETH, and SOL are still high (> $9 mil combined). When looking for rebalancing, are we concerned about total deposits for each asset, or total borrows?
	- liquidation works by paying back borrows in order to receive the collateral at a discount. Therefore would be wise to specialize in USDC since that is where the majority of borrows occur. 

### Code notes
`mango/context.py` stores and manages the groups used.

Line 56 reads: 
```python
default_group_name = os.environ.get("GROUP_NAME") or "BTC_ETH_SOL_SRM_USDC"
```
Which seems like it defaults to the 5 group. This has been confirmed by looking up the commits on this file, one of the commits specifically changes this default group.

Could try just running the program and seeing what happens. There's always --dry-run plus if it defaults to the 5 group not sure what else there is.

Test USDC/USDT context in `/tests/test_context.py`. Also `tests/test_group.py` function `test_group5_parse()`.

`tests/test_walletbalancer.py` has tests for rebalancing into ETH/BTC/USDT on v1 but does not test for the 5 group. Can write tests and **also** make sure that the wallet can actually rebalance to USDC. Again can do this by just testing