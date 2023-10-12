# What is DODO V3?

A leveraged market making solution to minimize IL and improve on liquidity management. The solution provides yield for retail LPs, higher profits for expert LPs, and better liquidity for traders.

[](https://audits.sherlock.xyz/contests/89)

# Issues i was able to find

|Severity| Title| Link|
|--------|------|-----|
| Medium| possible precision loss in D3VaultLiquidation.finishLiquidation() function when calculating realDebt because of division before multiplication| [Link](https://github.com/sherlock-audit/2023-06-dodo-judging/issues/45)|
| Medium| D3Oracle.getPrice() and D3Oracle.getOriginalPrice() doesn't check If Arbitrum sequencer is down for Chainlink feeds| [Link](https://github.com/sherlock-audit/2023-06-dodo-judging/issues/62)|
| Medium| possible precision loss when calculating borrows in D3VaultLiquidation.liquidate() because of division before multiplication| [Link](https://github.com/sherlock-audit/2023-06-dodo-judging/issues/43)|
| Medium| return value of ERC20.transferFrom() is not checked, can cause loss of funds/stuck funds in D3VaultLiquidation| [Link](https://github.com/sherlock-audit/2023-06-dodo-judging/issues/42)|
| Medium| heartbeat issues for chainlink| [Link](https://github.com/sherlock-audit/2023-06-dodo-judging/issues/39)|
| Medium| ERC20.approve() doesn't approve to 0 first for tokens like USDT| [Link](https://github.com/sherlock-audit/2023-06-dodo-judging/issues/35)|


# 1. possible precision loss in D3VaultLiquidation.finishLiquidation() function when calculating realDebt because of division before multiplication

## Summary
finishLiquidation() divides before multiplying when calculating realDebt.

## Vulnerability Detail
uint256 realDebt = borrows.div(record.interestIndex == 0 ? 1e18 : record.interestIndex).mul(info.borrowIndex);
There will be precision loss when calculating the realDebt because solidity truncates values when dividing and dividing before multiplying causes precision loss.

Values that suffered from precision loss will be updated here

 info.totalBorrows = info.totalBorrows - realDebt;
 
## Impact
Values that suffered from precision loss will be updated here

 info.totalBorrows = info.totalBorrows - realDebt;
 
## Code Snippet
https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Vault/D3VaultLiquidation.sol#L144

https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Vault/D3VaultLiquidation.sol#L147

## Tool used
Manual Review

## Recommendation
don't divide before multiplying

# 2. D3Oracle.getPrice() and D3Oracle.getOriginalPrice() doesn't check If Arbitrum sequencer is down for Chainlink feeds

## Summary
When utilizing Chainlink in L2 chains like Arbitrum, it's important to ensure that the prices provided are not falsely perceived as fresh, even when the sequencer is down. This vulnerability could potentially be exploited by malicious actors to gain an unfair advantage.

## Vulnerability Detail
There is no check in D3Oracle.getPrice()
```solidity
 function getPrice(address token) public view override returns (uint256) {
        require(priceSources[token].isWhitelisted, "INVALID_TOKEN");
        AggregatorV3Interface priceFeed = AggregatorV3Interface(priceSources[token].oracle);
        (uint80 roundID, int256 price,, uint256 updatedAt, uint80 answeredInRound) = priceFeed.latestRoundData();
        require(price > 0, "Chainlink: Incorrect Price");
        require(block.timestamp - updatedAt < priceSources[token].heartBeat, "Chainlink: Stale Price");
        require(answeredInRound >= roundID, "Chainlink: Stale Price");
        return uint256(price) * 10 ** (36 - priceSources[token].priceDecimal - priceSources[token].tokenDecimal);
    }
```
no check in D3Oracle.getOriginalPrice() too
```solidity
 function getOriginalPrice(address token) public view override returns (uint256, uint8) {
        require(priceSources[token].isWhitelisted, "INVALID_TOKEN");
        AggregatorV3Interface priceFeed = AggregatorV3Interface(priceSources[token].oracle);
        (uint80 roundID, int256 price,, uint256 updatedAt, uint80 answeredInRound) = priceFeed.latestRoundData();
        require(price > 0, "Chainlink: Incorrect Price");
        require(block.timestamp - updatedAt < priceSources[token].heartBeat, "Chainlink: Stale Price");
        require(answeredInRound >= roundID, "Chainlink: Stale Price");
        uint8 priceDecimal = priceSources[token].priceDecimal;
        return (uint256(price), priceDecimal);
    }
```
## Impact
could potentially be exploited by malicious actors to gain an unfair advantage.

## Code Snippet
https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/periphery/D3Oracle.sol#L48

https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/periphery/D3Oracle.sol#L58

## Tool used
Manual Review

## Recommendation
code example of Chainlink:
https://docs.chain.link/data-feeds/l2-sequencer-feeds#example-code

# 3. possible precision loss when calculating borrows in D3VaultLiquidation.liquidate() because of division before multiplication

## Summary
D3VaultLiquidation.liquidate() divides before multiplying when calculating borrows

## Vulnerability Detail
please take a look at the snippet below
```solidity
 uint256 borrows = record.amount.div(record.interestIndex == 0 ? 1e18 : record.interestIndex).mul(info.borrowIndex);
//@audit it divides before multiplying
```
--- record.amount is divided by 1e18 if record.interestIndex == 0 or by record.interestIndex if it's != 0, then it's multiplied by info.borrowIndex

lets say:
record.amount = 5000,
record.interestIndex = 7,
record.borrowIndex = 5,

Now normal maths will have 5000 / 7 which will give 714.2857142857 then multiplying it by 5 = 3571.4285714285

But in solidity when we do 5000/7 we'll have 714 and not 714.285714285 because solidity truncates values when dividing, then multiplying by 5 we'll have 3570 instead of 3571.4285714285.

In the above instance there's a loss of about 1.428571428500163

## Impact
precision loss when calculating borrows in D3VaultLiquidation.liquidate()

it might affect the require statement below it making it to revert in a situation where user puts his exact record.amount as debtToCover when calling D3VaultLiquidation.liquidate()
```solidity
 require(debtToCover <= borrows, Errors.DEBT_TO_COVER_EXCEED);
```
because of the precision loss the borrows calculated will loss precision and be lesser than debtToCover.

incorrect values will be updated here
 record.amount = borrows - debtToCover;
## Code Snippet
https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Vault/D3VaultLiquidation.sol#L53-L54

Also here
https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Vault/D3VaultFunding.sol#L99

## Tool used
Manual Review

## Recommendation
look for a more efficient way to calculate borrows but don't divide before multiplying so as to avoid precision loss

# 4. return value of ERC20.transferFrom() is not checked, can cause loss of funds/stuck funds in D3VaultLiquidation

## Summary
The return value of transferFrom() is not checked

## Vulnerability Detail
The return value of transferFrom() is not checked, transfers can fail unexpectedly maybe due to insufficient allowance or any other reason and this will result in the function assuming the transfer was successful.

## Impact
transferFrom is unsafe and also since the return value isn't checked in the startLiquidation() function and liquidateByDODO() function, these functions may assume the transfer was successful leading to loss of funds / stuck funds

## Code Snippet
https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Vault/D3VaultLiquidation.sol#L55

https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Vault/D3VaultLiquidation.sol#L59

https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Vault/D3VaultLiquidation.sol#L98

## Tool used
Manual Review

## Recommendation
use openzeppelin's safeTransferFrom() function.

# 5. heartbeat issues for chainlink

## Summary
On Arbitrum, as well as pretty much any other network, different token pairs have different heartbeats.

## Vulnerability Detail
using the same heartbeat for all feeds is highly dangerous especially when the feed is a chainlink feed

The issue with this is that the USDC/USD oracle has a 24 hour heartbeat, whereas the average has a heartbeat of 1 hour. Since they use the same heartbeat the heartbeat needs to be slower of the two or else the contract would be nonfunctional most of the time. The issue is that it would allow the consumption of potentially very stale data from the non-USDC feed.

If the oracle gets the latest price for two pairs with different heartbeats, using the same heartbeat variable for validation would cause either one of the following:

Oracle will be down (will revert) most of the time.

Oracle will allow for stale prices

When validating prices for two different token pairs, two different heartbeats must be used.

## Impact
Oracle will be down (will revert) most of the time.

Oracle will allow for stale prices

## Code Snippet
https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/periphery/D3Oracle.sol#L48-L56

https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/periphery/D3Oracle.sol#L58-L67

## Tool used
Manual Review

## Recommendation
When validating prices for two different token pairs, two different heartbeats must be used.

# 6. ERC20.approve() doesn't approve to 0 first for tokens like USDT

## Summary
first reduce allowance to 0 first for tokens like USDT

## Vulnerability Detail
There are numerous instances where the IERC20.approve() function is called only once without setting the allowance to zero. Some tokens, like USDT, require first reducing the address' allowance to zero by calling approve(_spender, 0). Transactions will revert when using tokens like USDT (see the approve() function requirement below at line 201-203).
```solidity
function approve(address _spender, uint _value) public onlyPayloadSize(2 * 32) {

        **// To change the approve amount you first have to reduce the addresses`
        **//  allowance to zero by calling `approve(_spender, 0)` if it is not
        **//  already 0 to mitigate the race condition described here:
        //  https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
        require(!((_value != 0) && (allowed[msg.sender][_spender] != 0)));

        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);
    }
```
## Impact
Transactions will revert when using tokens like USDT

## Code Snippet
https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Pool/D3Funding.sol#L22

https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Pool/D3Funding.sol#L52

https://github.com/sherlock-audit/2023-06-dodo/blob/main/new-dodo-v3/contracts/DODOV3MM/D3Pool/D3Funding.sol#L66

## Tool used
Manual Review

## Recommendation
To change the approve amount you first have to reduce the addresses allowance to zero by calling approve(_spender, 0)
