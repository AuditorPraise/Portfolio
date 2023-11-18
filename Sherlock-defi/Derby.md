# What is Derby Finance?
Derby Finance is a community powered yield optimizer that diversifies its exposure over a wide variety of DeFi yield opportunities on different EVM chains and layer 2s.

[Link to contest page](https://audits.sherlock.xyz/contests/13)

# Issues i was able to find

|Severity | Title| Link|
|---------|------|-----|
| Medium | The exchange rate in MainVault.sol is statically updated.| [Link](https://github.com/sherlock-audit/2023-01-derby-judging/issues/173)|
| Medium | missing deadline and deadline checker in xTransfer() and swapStableCoins() | [Link](https://github.com/sherlock-audit/2023-01-derby-judging/issues/111)|

# 1. The exchange rate in MainVault.sol is statically updated.

## Summary

The exchange rate in MainVault.sol is not dynamically adjusted based on market conditions.

## Vulnerability Detail

The exchange rate is static as it is only updated once in 2 weeks. Within 2 weeks the price of Lp tokens will change a lot and the exchange rate will become stale.
If the exchange rate is statically adjusted in a vault, it means that the rate at which assets are exchanged or swapped within the vault won't be all that correct as the exchangeRate is meant to be adjusted based on market conditions. This can have several implications for the vault and its users, like:
Lack of flexibility of the vault, price risk, arbitrage Opportunities.

## Impact

The exchangeRate state variable was updated in the constructor.
```solidity
  constructor(
    string memory _name,
    string memory _symbol,
    uint8 _decimals,
    uint256 _vaultNumber,
    address _dao,
    address _game,
    address _controller,
    address _vaultCurrency,
    uint256 _uScale
  )
    VaultToken(_name, _symbol, _decimals)
    Vault(_vaultNumber, _dao, _controller, _vaultCurrency, _uScale)
  {
    **exchangeRate = _uScale;**
    game = _game;
    governanceFee = 0;
    maxDivergenceWithdraws = 1_000_000;
  }
```
And constructors only run once, looking through the code i see some functions that updates the exchangeRate but this is statically done (i.e the update wasn't created to sync with realTime market conditions).
```solidity
  function setXChainAllocationInt(
    uint256 _amountToSend,
    uint256 _exchangeRate,
    bool _receivingFunds
  ) internal {
    amountToSendXChain = _amountToSend;
    **exchangeRate = _exchangeRate;**


    if (_amountToSend == 0 && !_receivingFunds) settleReservedFunds();
    else if (_amountToSend == 0 && _receivingFunds) state = State.WaitingForFunds;
    else state = State.SendingFundsXChain;
  }
```
This could lead to the following:

1.) Lack of Flexibility: Statically updating the exchange rate in a vault can limit the flexibility of the vault, making it less adaptable to changes in market conditions. For example, if the exchange rate is statically updated, it may not be possible to adjust the rate to account for changes in supply and demand, leading to potential inefficiencies in the market.

2.) Price Risk: Users of the vault may be exposed to higher price risks if the exchange rate is statically updated. For example,
in the withdraw function, value is gotten by multiplying '_amount ' by the exchangeRate and dividing by '10**decimals()'

    value = (_amount * exchangeRate) / (10 ** decimals());
Now since the exchangeRate is statically updated the outcome of "(_amount * exchangeRate) / (10 ** decimals())" could be a much lower value than real market price of the Lp tokens or a much higher value than real market price of the Lp tokens, as the exchangeRate of markets are always changing (i.e the actual price of the Lp token is constantly changing.)

## Code Snippet

https://github.com/sherlock-audit/2023-01-derby/blob/main/derby-yield-optimiser/contracts/MainVault.sol#L296

https://github.com/sherlock-audit/2023-01-derby/blob/main/derby-yield-optimiser/contracts/MainVault.sol#L64

## Tool used

Manual Review

## Recommendation

build the exchangeRate to sync with realTime market conditions (i.e the realTime price movement of the Lp token).


# 2.  missing deadline and deadline checker in xTransfer() and swapStableCoins() 

## Summary
the transactions can potentially remain pending indefinitely, if there are network issues or other problems.

## Vulnerability Detail

These functions should have a deadline and deadline checker implemented to ensure that the transaction is completed within a reasonable amount of time since it transfers funds from one chain to another like the xTransfer()
the transaction can potentially remain pending indefinitely, if there are network issues or other problems.

i think it's the same case for swapStableCoin()

## Impact
The xTransfer() function appears to be transferring tokens using the Connext protocol, which likely involves exchanging tokens across different blockchains. In this case, it may be beneficial to include a deadline and deadline checker to ensure that the transaction is completed within a reasonable amount of time, because the transaction can potentially remain pending indefinitely, if there are network issues or other problems.

Same for the swapStableCoins() function which swaps stable coins on curve.
Without a deadline and deadline checker, there is a risk that the transaction may take longer than expected

## Code Snippet
https://github.com/sherlock-audit/2023-01-derby/blob/main/derby-yield-optimiser/contracts/XProvider.sol#L133-L161

https://github.com/sherlock-audit/2023-01-derby/blob/main/derby-yield-optimiser/contracts/libraries/Swap.sol#L36-L58

## Tool used
Manual Review

## Recommendation
implement a deadline and deadline checker so that users can have control over the duration of the tx, because users can be helpless in situations where the tx is stuck in the mempool due to network issues or other problems.





