# Leverage

Although leverage is not explicitly defined in our protocol, the amount of margin a trader uses to collateralize a position is a function of leverage according to the following formula:

$$
margin = quantity \cdot \max (\frac{P_{contract}}{leverage},\frac{ P_{index} }{leverage} - NPV)
$$

For new positions, the funding fee component of the NPV formula can removed since the position has not been created yet, resulting in the following NPV calculations:

$$
NPV_{perpetual\ long}= P_{index}-P_{contract}
$$

$$
NPV_{perpetual\ short} = P_{contract}-P_{index}
$$

