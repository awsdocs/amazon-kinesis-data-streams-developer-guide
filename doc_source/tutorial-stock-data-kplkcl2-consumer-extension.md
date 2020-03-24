# Step 6: \(Optional\) Extending the Consumer<a name="tutorial-stock-data-kplkcl2-consumer-extension"></a>

This optional section shows how you can extend the consumer code for a slightly more elaborate scenario\.

If you want to know about the biggest sell orders each minute, you can modify the `StockStats` class in three places to accommodate this new priority\.

**To extend the consumer**

1. Add new instance variables:

   ```
    // Ticker symbol of the stock that had the largest quantity of shares sold 
    private String largestSellOrderStock;
    // Quantity of shares for the largest sell order trade
    private long largestSellOrderQuantity;
   ```

1. Add the following code to `addStockTrade`:

   ```
   if (type == TradeType.SELL) {
        if (largestSellOrderStock == null || trade.getQuantity() > largestSellOrderQuantity) {
            largestSellOrderStock = trade.getTickerSymbol();
            largestSellOrderQuantity = trade.getQuantity();
        }
    }
   ```

1. Modify the `toString` method to print the additional information:

   ```
    
   public String toString() {
       return String.format(
           "Most popular stock being bought: %s, %d buys.%n" +
           "Most popular stock being sold: %s, %d sells.%n" +
           "Largest sell order: %d shares of %s.",
           getMostPopularStock(TradeType.BUY), getMostPopularStockCount(TradeType.BUY),
           getMostPopularStock(TradeType.SELL), getMostPopularStockCount(TradeType.SELL),
           largestSellOrderQuantity, largestSellOrderStock);
   }
   ```

If you run the consumer now \(remember to run the producer also\), you should see output similar to this:

```
 
  ****** Shard shardId-000000000001 stats for last 1 minute ******
  Most popular stock being bought: WMT, 27 buys.
  Most popular stock being sold: PTR, 14 sells.
  Largest sell order: 996 shares of BUD.
  ****************************************************************
```

## Next Steps<a name="tutorial-stock-data-kplkcl2-consumer-extension-next"></a>

[Step 7: Finishing Up](tutorial-stock-data-kplkcl2-finish.md)