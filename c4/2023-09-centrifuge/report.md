
| Severity | Title | 
|:--:|:---|
| [M-01](#m-01-you-can-deposit-for-other-users-really-small-amount-to-dos-them) | You can deposit for other users really small amount to DoS them |



# [M-01] You can deposit for other users really small amount to DoS them

## Impact
Deposit and mint under [**LiquidityPool**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L141-L152) lack access control, which enables any user to **proceed** the  mint/deposit for another user. Attacker can deposit (this does not require tokens) some wai before users TX to DoS the deposit.

## Proof of Concept
[deposit](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L141-L144) and [mint](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L148-L152) do [processDeposit](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L427-L441)/[processMint](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L451-L465) which are the secondary functions to the requests. These function do not take any value in the form of tokens, but only send shares to the receivers. This means they can be called for free. 

With this an attacker who wants to DoS a user, can wait him to make the request to deposit and on the next epoch front run him by calling  [deposit](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L141-L144) with something small like 1 wei. Afterwards when the user calls `deposit`, his TX will inevitable revert, as he will not have enough balance for the full deposit.  

## Tools Used
Manual review.

## Recommended Mitigation Steps
Have some access control modifiers like [**withApproval**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L97-L100) used also in [redeem](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L200-L208).

```diff
-    function deposit(uint256 assets, address receiver) public returns (uint256 shares)  {
+    function deposit(uint256 assets, address receiver) public returns (uint256 shares) withApproval(receiver) {
        shares = investmentManager.processDeposit(receiver, assets);
        emit Deposit(address(this), receiver, assets, shares);
     }

-    function mint(uint256 shares, address receiver) public returns (uint256 assets) {
+    function mint(uint256 shares, address receiver) public returns (uint256 assets) withApproval(receiver) {
        assets = investmentManager.processMint(receiver, shares);
        emit Deposit(address(this), receiver, assets, shares);
     }
```