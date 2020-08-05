

## Margin

When a user inputs a specified **price**, 

Although leverage is not explicitly defined in our protocol, the amount of margin a trader uses to collateralize a position is a function of leverage according to the following formula:

`margin = quantity * max(contractPrice / leverage, indexPrice / leverage - NPV)`


For new positions, the funding fee component of the NPV formula can removed since the position has not been created yet, resulting in the following NPV calculations:

- `NPV_long = indexPrice - contractPrice` 
- `NPV_short = contractPrice - indexPrice`



## Liquidation Price





## Implied PNL

