# Rio Liquidity Algorithm

## Background

Rio operates as a crypto broker, sourcing liquidity from OTC desks and exchanges and selling it to retail customers. Rio initially operated under a real-time order model in which the liquidity would be sourced from an exchange and withdrawn to our company wallet and subsequently to the user's wallet address in real time. This presented many issues, including high network fees (each order must incur two 'hops' to get to the end user, once from the exchange to Rio and another from Rio to the user) and technical limitations (some exchanges do not support high frequency withdraws without manual intervention).

Because of the problems associated with real-time orders and withdraws from exchanges, we have decided to explore the idea of distributing funds to our users in real time from our own liquidity reserves, but batching the withdraws from the exchange to our reserves to save on gas fees. This would involve front-running liquidity into our reserves, which would expose Rio to a price risk for the underlying assets. To mitigate this risk, we are considering opening short positions at the same time and for the same amount of the asset we are buying for our liquidity. This would protect us from variations in price. The position short would be closed throughout the day by placing buy orders for the asset as our users purchase crypto. Any crypto leftover in our reserves by the end of the day would be sent back to close our position and avoid margin fees that start accruing after 24H.

## Algorithm Steps for one asset (ETH)

1. At the start of the day, Rio forecasts daily demand for ETH. Determines its roughly 5 ETH.
2. Rio purchases 5 ETH from liquidity provider (in this case FalconX) and sends it to its reserves (Fireblock wallet)
3. Rio immediately opens a margin position for 5 ETH with FalconX (requires 25% collateral in USD)
4. Rio sells the 5 ETH it took out with margin, creating a short position on ETH
5. Rio now has 5 ETH in liquidity to sell to its customers
6. Customer A places a buy order for 1 ETH at a price lower than what Rio paid to FalconX
7. Rio places a market order on FalconX for 1 ETH
8. Upon order confirmation, Rio distributes 1 ETH to the user's wallet from its reserves on Fireblocks.
9. Throughout the day, steps 6-8 are repeated by another 4 customers
10. At this point, Rio has bought back 5 ETH on FalconX, closing its short position
11. In the case that not all 5 ETH were sold, what is remaining in the reserves will be sent back to FalconX at the end of the day to close the short position and avoid margin fees.
12. This process is repeated at the start of every day and for every asset Rio supports.

## Algorithm pseudocode

The following pseudocode is executed at the start and end of the day. This excludes the code responsible for placing orders on FalconX throughout the day.

```
At the start of every day:

    rioAssets = ['ETH', 'BTC']

    for asset in rioAssets:
        amountNeeded = forecastAmountNeeded(asset)

        falconX.purchaseCrypto(asset, amountNeeded)
        falconX.transfer(asset, amountNeeded, rioReservesAddress)

        falconX.getMargin(asset, amountNeeded)
        falconX.sell(asset,amountNeeded)

At the end of every day:

    rioAssets = ['ETH', 'BTC']

    for asset in rioAssets:
        remainingLiquidity = rioReserves.getLiquidity(asset)

        if (remainingLiquidity > 0):
            rioReserves.send(asset, remainingLiquidity, falconXAddress)

```
