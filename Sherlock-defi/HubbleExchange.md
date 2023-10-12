# What is Hubble Exchange?
Hubble Exchange is the first Multi-collateral / Cross-Margin Perpetual Futures protocol on Avalanche.

[Link to contest oage](https://audits.sherlock.xyz/contests/72)

# Issues i was able to find

|Severity| Title| Link|
|--------|------|-----|
| Medium| Oracle.sol may return stale price| [Link](https://github.com/sherlock-audit/2023-04-hubble-exchange-judging/issues/19)|

# 1.  Oracle.sol may return stale price

## Summary
Oracle should use the updatedAt value from the latestRoundData() function to make sure that the latest answer is recent enough to be used.

## Vulnerability Detail
In the current implementation of Oracle.sol there is no freshness check. This could lead to stale prices being used.

If the market price of the token drops very quickly ("flash crashes"), and Chainlink's feed does not get updated in time, the smart contract will continue to believe the token is worth more than the market value.

Chainlink also advise developers to check for the updatedAt before using the price:

"Your application should track the latestTimestamp variable or use the updatedAt value from the latestRoundData() function to make sure that the latest answer is recent enough for your application to use it. If your application detects that the reported answer is not updated within the heartbeat or within time limits that you determine are acceptable for your application, pause operation or switch to an alternate operation mode while identifying the cause of the delay."

## Impact
Oracle.sol may return stale price

Code Snippet
https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/main/hubble-protocol/contracts/Oracle.sol#L24

https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/main/hubble-protocol/contracts/Oracle.sol#L38

https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/main/hubble-protocol/contracts/Oracle.sol#L107

## Tool used
Manual Review

## Recommendation
Consider adding the missing freshness check for stale price. You can do something like this:

```solidity
require(block.timestamp - updatedAt < validPeriod, "freshness check failed.")
```
just use the current block.timestamp with getLatestRoundData's updatedAt to track against validPeriod.
The validPeriod can be based on the Heartbeat of the feed.
