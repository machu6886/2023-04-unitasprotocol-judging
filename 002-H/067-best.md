shogoki

high

# Unitas swap function is vulnerable to Sandwich Attack at oracle price update


## Summary

WHen the oracle price is updates, an attacker can sandwich the update adderss with 2 swap transactions to gain a profit and drain the collateral.

## Vulnerability Detail

Inside the `swap` function of the `Unitas` contract the `getLatestPrice` function of the  `XOracle` contract is used to fetch the current price of the USDx token to be swapped to/from.
The price has to be updated periodically because the currencies fluctuate against the USD price.
A user could use these fluctuations to speculate on prices and gain a profit or loss.

However,as the price for the oracle is updated by a transaction that is publicly visible inside the mempool, a malicious user or attacker can see the new price before it is active. As the oracle price is the only thing, which has influence of the token number to mint/burn on a swap call, an attacker can easily exploit a temporarily appreciation of an USDx token agains the USD1 (US Dollar price). 
This can be achieved by "sandwiching" the price update transaction with a transaction to swap USD1 into the relevant USDx token first, and swap it back after the price update.

Example: 
THe price of USD91 (INR) is to be increased from 0.012 to 0.013
Attacker already holds 10,000 of USD1 tokens.

- The Feeder sends the transaction to update the oracle price, and it gets placed in the mempool.
- Attacker sees these transaction, and sends himself 2 transactions.
1. Swap 10,000 possible mount of USD1 to USD91
2. Swap all USD91 to USD1
- The attacker sets the gas to ensure that the first tx gets included before the price update, and the second one after the price update. 
- The executed Transactions in order will be:
1. Attacker swaps 10,000 USD1 to USD91 (should receive: 10,000 / 0.012 = 833,333.333)
2. FEEDER updates oracle price of USD91 to 0.013
3. Attacker swaps all USD91 tokens to USD1 (should receive: 833,333.333 * 0.013 = 10,833.333)

The attacker just made 833 USD1 profit in these 2 transactions, and can redeem the USD1 tokens for USDT, which will deduct the collateral by this amount. 
 

## Impact

Attacker can gain profit and "steal" collateral

## Code Snippet

https://github.com/sherlock-audit/2023-04-unitasprotocol/blob/main/Unitas-Protocol/src/Unitas.sol#L439

https://github.com/sherlock-audit/2023-04-unitasprotocol/blob/main/Unitas-Protocol/src/XOracle.sol#L26-L32

## Tool used

Manual Review

## Recommendation

The 2 way minting/burning mechanism by the orcale price might be dangerous.
However to prevent the specific attack vector, maybe the minting can be paused and unpaused before and after the price update. (in separate transactions) 