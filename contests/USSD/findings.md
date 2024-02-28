| Severity          | Title  |
| --------          | -----  |
| :red_circle: High | [ETH/USD priceFeed address is used as BTC/USD](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/199) |
| :red_circle: High | [Zero address is used as StableOracleWETH in StableOracleDAI](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/200) |
| :red_circle: High | [Wrong decimals for DAI/ETH chainlink priceFeed is used](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/204) |
| :red_circle: High | [Wrong address is used as StaticOracle in StableOracleDAI.sol and StableOracleWBGL.sol](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/205) |
| :red_circle: High | [StableOracleDAI values DAI at $2](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/503) |
| :red_circle: High | [No access control in rebalancer functions in USSD.sol](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/623) |
| :red_circle: High | [No slippage control when performing swap in USSD.sol](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/680) |
| :red_circle: High | [Protocol assumes DAI is $1](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/714) |
| :orange_circle: Medium | [BuyUSSDSellCollateral() always sells 0 amount if need to sell part of collateral](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/656) |
| :orange_circle: Medium | [Possible DOS of USSDRebalancer due to price deviation of oracle](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/671) |
| :orange_circle: Medium | [StableOracleWBTC use BTC/USD chainlink oracle to price WBTC which is problematic if WBTC depegs](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/202) |
| :orange_circle: Medium | [StableOracle contracts will return the wrong price for asset if underlying aggregator hits minAnswer](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/197) |
| :orange_circle: Medium | [Chainlink's latestRoundData return stale or incorrect result](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/176) |
