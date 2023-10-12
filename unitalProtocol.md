# What is Unitas Protocol all about?
It's all about Unitized stablecoins serving as units of account representing emerging market currencies. A new currency revolution.

[link to the contest page](https://audits.sherlock.xyz/contests/73)

# Issues i was able to find
|Severity | Title | Link|
|---------|-------|-----|
|Medium   |XOracle.getLatestPrice() can return stale price within _getSwapResult() function| [Link](https://github.com/sherlock-audit/2023-04-unitasprotocol-judging/issues/87)|



# 1.  XOracle.getLatestPrice() can return stale price within _getSwapResult() function

# Summary

getLatestPrice() may not really return latest price

# Vulnerability Detail

there's no check between the prev update timestamp and the current timestamp. The price returned can be a stale price when comparing real world price with what is returned by XOracle.getLatestPrice()

# Impact

XOracle.getLatestPrice() can return stale price within _getSwapResult() function

        price = oracle.getLatestPrice(priceQuoteToken);
# Code Snippet

https://github.com/sherlock-audit/2023-04-unitasprotocol/blob/main/Unitas-Protocol/src/XOracle.sol#L58

# Tool used

Manual Review

# Recommendation

get the prev update timestamp and compare with the current timestamp, ensure a revert on stale prices by comparing the two timestamps within a require statement
