# What is Tokemak?
Its a generating sustainable liquidity for the tokenized world. Eliminating inefficiencies and helping LPs to deploy liquidity where it can do the most work.

[Link to contest page](https://audits.sherlock.xyz/contests/101)

# issues i was able to find?

|Severity| Title| Link|
|--------|------|-----|
| Medium| LMPVaultRegistry.removeVault doesn’t remove vault from _vaultsByType mapping| [Link](https://github.com/sherlock-audit/2023-06-tokemak-judging/issues/68)|

# Issue: LMPVaultRegistry.removeVault doesn’t remove vault from _vaultsByType mapping

## Summary
LMPVaultRegistry.removeVault doesn’t remove vault from _vaultsByType mapping

## Vulnerability Detail
when vaults are added via LMPVaultRegistry.addVault(), the vaults are added to 3 mappings

_vaults mapping, see here
_vaultsByAsset mapping, see here
_vaultsByType mapping, see here
But when the vaults are being removed via LMPVaultRegistry.removeVault(), LMPVaultRegistry.removeVault() fails to remove the vault from _vaultsByType mapping.

it removes the vault from _vaults mapping here and _vaultsByAsset mapping here
But it fails to remove the vault from _vaultsByType mapping.

Therefore a removed vault will still exist in _vaultsByType mapping

## Impact
A removed vault will still exist in _vaultsByType mapping
And if there be any need to re-add the vault via LMPVaultRegistry.addVault() there will be a revert because of this line
function listVaultsForType() will still list the removed vault as one of the available vaults for a particular type, see [here](https://github.com/sherlock-audit/2023-06-tokemak/blob/5d8e902ce33981a6506b1b5fb979a084602c6c9a/v2-core-audit-2023-07-14/src/vault/LMPVaultRegistry.sol#L102-L103)
## Code Snippet
https://github.com/sherlock-audit/2023-06-tokemak/blob/main/v2-core-audit-2023-07-14/src/vault/LMPVaultRegistry.sol#L64

## Tool used
Manual Review

## Recommendation
remove the vault from _vaultsByType mapping too in the LMPVaultRegistry.removeVault() function
