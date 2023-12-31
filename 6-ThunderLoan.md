### [H-1] Error `ThunderLoan:updateExchangeRate` in the `deposit` function causes protocol to think it has more fees than it really does,  which blocks redemption and incorrectly sets the exchange rate



**Description:**

In the ThunderLoan system, the `exchangeRate` is responsible for calculating the exchange rate between assetTokens and underlying tokens . In a way, It's responsible for keeping track of how many fees to give to liquidity providers .

However, the `deposit` function, updates this rate, without collecting any fees !

```solidity

    function deposit(IERC20 token, uint256 amount) external revertIfZero(amount) revertIfNotAllowedToken(token) {
        AssetToken assetToken = s_tokenToAssetToken[token];
        uint256 exchangeRate = assetToken.getExchangeRate();
        uint256 mintAmount = (amount * assetToken.EXCHANGE_RATE_PRECISION()) / exchangeRate;
        emit Deposit(msg.sender, token, amount);
        assetToken.mint(msg.sender, mintAmount);
@>      uint256 calculatedFee = getCalculatedFee(token, amount);
@>      assetToken.updateExchangeRate(calculatedFee);
        token.safeTransferFrom(msg.sender, address(assetToken), amount);
    }
```



**Impact:** 

There are several impacts to this bug .

1. The `redeem` function is blocked, because the protocol thinks the owned tokens is more than it has .

2. Rewards are incorrectly calculated, leading to liquidity providers potentially getting way more or less than deserved .

   

**Proof of Concept:**

1. LP deposits
2. User takes out a flash loan
3. It is now impossible for LP to redeem 

```solidity

  function testRedeemAfterLoan() public setAllowedToken hasDeposits {
        uint256 amountToBorrow = AMOUNT * 10;
        uint256 calculatedFee = thunderLoan.getCalculatedFee(tokenA, amountToBorrow);

        vm.startPrank(user);
        tokenA.mint(address(mockFlashLoanReceiver), calculatedFee);
        thunderLoan.flashloan(address(mockFlashLoanReceiver), tokenA, amountToBorrow, "");
        vm.stopPrank();

        uint256 amountToRedeem = type(uint256).max;
        vm.startPrank(liquidityProvider);
        //@audit redeem transferUnderlyingTo  1003300900000000000000  = 1003.3009e18
        // deposit  1000e18 , fee 0.3e18
        thunderLoan.redeem(tokenA, amountToRedeem);
        // @audit High People cannot reedeem their own tokens because the wrong calculation fee
        
    }

```



**Recommended Mitigation:** 

Removed the incorrectly updated exchange rate lines from `deposit`, 

```diff

    function deposit(IERC20 token, uint256 amount) external revertIfZero(amount) revertIfNotAllowedToken(token) {
        AssetToken assetToken = s_tokenToAssetToken[token];
        uint256 exchangeRate = assetToken.getExchangeRate();
        uint256 mintAmount = (amount * assetToken.EXCHANGE_RATE_PRECISION()) / exchangeRate;
        emit Deposit(msg.sender, token, amount);
        assetToken.mint(msg.sender, mintAmount);
-       uint256 calculatedFee = getCalculatedFee(token, amount);
-       assetToken.updateExchangeRate(calculatedFee);
        token.safeTransferFrom(msg.sender, address(assetToken), amount);
    }
```





### [M-1]Centralization risk for trusted Owners

**Description:** 

Contracts have owners with previleged rights to perform admin task and need to be trusted to not perform malicious updates or drain funds





**Impact:** 

**Proof of Concept:**

**Recommended Mitigation:** 