# Opal contest result

# 1 High and 4 mediums (of which 2 are solos) vulnerabilities found


# Med-solo-1: _withdrawFromAuraPool() will revert if BPT isn't much, this shouldn't be so as it still try to withdraw some BPT to make up [issue #46]

AuditorPraise
AuditorPraise

created on Feb 14, 2024 at 20:41
•

edited by AuditorPraise on Feb 14, 2024 at 20:44

### Description:
In Omnipool._withdrawFromAuraPool() there will always be a revert whenever the balance of the BPT to withdraw is < _bptAmountOut. This is wrong since Omnipool._withdrawFromAuraPool() still tries to withdraw some BPT to make up the expected _bptAmountOut.

i believe this check (require(balance >= _bptAmountOut, "not enough balance");) should have been after _withdrawFromAuraPool() tries to withdraw some BPT from the auraPool to make up the expected _bptAmountOut, that way if the BPT balance is still lower than the expected _bptAmountOutfor whatever reason it can now revert.

### Recommendation:

change the order of the code from this:

```solidity
 // Make sure we have enough BPT to withdraw
        uint256 balance = auraPool.balanceOf(address(this));
        require(balance >= _bptAmountOut, "not enough balance");//@audit-issue this renders the below line of code useless 
        auraPool.withdrawAndUnwrap(_bptAmountOut, true);
```
```solidity
to this:

 auraPool.withdrawAndUnwrap(_bptAmountOut, true);// @audit-ok withdraw first from auraPool
 uint256 balance = auraPool.balanceOf(address(this));
 require(balance >= _bptAmountOut, "not enough balance")
```

AuditorPraise
AuditorPraise

changed the description

Feb 14, 2024 at 20:44

AuditorPraise
AuditorPraise

linked file src/pools/Omnipool.sol

Feb 15, 2024 at 12:41

AuditorPraise
AuditorPraise

unlinked file src/pools/Omnipool.sol

Feb 15, 2024 at 12:42

AuditorPraise
AuditorPraise

linked file src/pools/Omnipool.sol

Feb 15, 2024 at 12:43



hrishibhat
hrishibhat

commented on Feb 25, 2024 at 03:08
Public
```
I don't see a problem here. The bpt balance of the aura pool is already checked for, not sure how withdrawAndUnwrap would add something extra. also in the suggestion the balance would be reduced after withdrawAndUnwrap, so the balance returned will be less. I don't think that is correct.
```
Hrishikesh Bhat
hrishibhat

changed

Severity: Medium
Severity: Informational
Feb 25, 2024 at 03:08

AuditorPraise
AuditorPraise

commented on Feb 29, 2024 at 08:51
Public
```
The bpt balance of the aura pool gotten in the current sequence may not == _bptAmountOut The issue here is that auraPool.withdrawAndUnwrap(_bptAmountOut, true) is called to withdraw more bpt to increase the bpt balance of the aura pool in the omnipool.

so checking that the bpt balance of the aura pool is < than _bptAmountOut when auraPool.withdrawAndUnwrap(_bptAmountOut, true) will still be called to withdraw more bpt seems like mistimed logic, because even if the bpt balance of the aura pool in the omnipool isn't sufficient for _bptAmountOut why revert the tx when you'll still try to withdraw more bpt from the aurapool to increase the bpt balance of the aura pool in the omnipool ?
```
AuditorPraise
AuditorPraise

commented on Feb 29, 2024 at 09:00
Public
```
Here's the logic of withdrawAndUnwrap(), it withdraw Bpt and sends it to the omnipool contract. This will increase the bpt balance of the aurapool

  function withdrawAndUnwrap(uint256 amount, bool claim) public updateReward(msg.sender) returns(bool){

        //also withdraw from linked rewards
        for(uint i=0; i < extraRewards.length; i++){
            IRewards(extraRewards[i]).withdraw(msg.sender, amount);
        }
        
        _totalSupply = _totalSupply.sub(amount);
        _balances[msg.sender] = _balances[msg.sender].sub(amount);

        //tell operator to withdraw from here directly to user
        IDeposit(operator).withdrawTo(pid,amount,msg.sender);
        emit Withdrawn(msg.sender, amount);

        //get rewards too
        if(claim){
            getReward(msg.sender,true);
        }
        return true;
    }
```
Hrishikesh Bhat
hrishibhat

changed

Severity: Informational
Severity: Medium
Mar 11, 2024 at 09:20

hrishibhat
hrishibhat

commented on Mar 11, 2024 at 09:21
Public
```
Agreed. Thanks for the additional clarity. Considering this valid issue
```
Hrishikesh Bhat
hrishibhat

changed

Status: New
Status: Confirmed
Mar 11, 2024 at 09:25


# Med-solo-2: Loss of GEM token incentive for a depositedFor user, whenever a user deposits for another user via depositFor() [issue #34]

AuditorPraise

created on Feb 14, 2024 at 13:10
•

edited by AuditorPraise on Feb 16, 2024 at 04:41

### Description:

whenever a user deposits for someone else via depositFor() in the omniPool, the "depositedFor" user is supposed to get the GEM Tokens as incentive whenever rebalancingRewardActive == true. But there's an issue in the depositedFor() which causes the GEM Tokens to be minted to the msg.sender instead of the _depositFor (The address of the user for whom to deposit).

_handleRebalancingRewards() is called inside depositFor() at Ln276 with msg.sender as account instead of _depositFor.

in _distributeRebalancingRewards() gem will be transferred to the msg.sender instead of _depositFor.

 `IERC20(gem).safeTransferFrom(incentivesMs, account, amount);`
 
### Recommendation:

when calling _handleRebalancingRewards() in depositFor() at Ln 276 use _depositFor as account instead of msg.sender.


AuditorPraise
AuditorPraise

linked file src/pools/Omnipool.sol

Feb 14, 2024 at 13:13

AuditorPraise
AuditorPraise

linked file src/tokenomics/GemMinterRebalancingReward.sol

Feb 14, 2024 at 13:15

AuditorPraise
AuditorPraise

changed the description

Feb 16, 2024 at 04:41

hrishibhat
hrishibhat

commented on Feb 27, 2024 at 03:10
Public
```
Valid but considering this a medium since this only impact when someone uses depositFor is done on behalf of the user and the exact impact of this is unclear since it depends on how the depositFor is intended to be used
```
Hrishikesh Bhat
hrishibhat

changed

Severity: High
Severity: Medium
Feb 27, 2024 at 03:10

Hrishikesh Bhat
hrishibhat

changed

Status: New


![opal contest real](https://github.com/AuditorPraise/Portfolio/assets/141132434/873e7f7e-6052-4f0f-89e0-45cb9595953b)


