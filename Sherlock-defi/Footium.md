# What is Footium?

Footium is a multiplayer football management game where players own and manage their own digital football club.

[LInk to contest page](https://audits.sherlock.xyz/contests/71)

# Issues i was able to find

|Severity| Title| Link|
|--------|------|-----|
| Medium| Loss of funds due to Unchecked return value of .transfer().| [Link](https://github.com/sherlock-audit/2023-04-footium-judging/issues/9)|

# 1. Loss of funds due to Unchecked return value of .transfer()

## Summary
not checking the return value of transfer() method can lead to loss of funds/ stuck funds

## Vulnerability Detail
The transfer method() returns boolean but the transferERC20() function doesn't check it.

Now when the transfer fails due to unforeseen circumstances the contract will assume the transfer was successful.

## Impact
funds can be stuck in contract because the transferERC20() functions assumes every transfer succeeds.

## Code Snippet
https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L110

Also here,
https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L130

## Tool used
Manual Review

## Recommendation
pls check the return boolean value and make the transferERC20() function revert on failure.
