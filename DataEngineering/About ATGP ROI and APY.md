##<div align="center"> Thinking STGP, ROI and APY <div/>

We want to calculate APY in a timeframe, which equals (1 + ROI) ** (year / FrameTime) - 1.
So, we first calculate ROI in the timeframe, which is STGP * STGpLP, where
   - STGP is STG price in USD (All stable coins are finally backed to USD), which has discrepancy between published and in-house data sets.
  - and STGpLP is the amount of STG dividend earned by a unit of LP staked in the farming in a given timeframe, which we **can** track WITHOUT error.

####1. STGP the likely source of discrepancy
**Definition by example** Distribution of the feature X across pools means the unit vector which has the direction (X value in 1st pool, X value in 2nd pool, ...), all captured at, or representing, the same time interval.
Once we decide to invest in StarGate, the distribution, across pools, of our investment should follow the distribution of STGP * STGpLP, which is the same as the distribution of STGpLP. (This is, of course, not very precise and we will be talking about it on a later stage). STGP is not required here. <br>
STGP is only required when we take into account the distribution, across DeFi protocols like Binance and cBridge, of our investment. When STGP is required, we can collect it from the authoritative firms, like Binance.
<br/> We have two type of operation:

- **Logal Optimization**: We have decided to invest/keep a given amount of fund in Stargate. Then we can further assume STGP = 1. Now predict ROI = STGP * STGpLP. We only need to predict STGpLP.
- **Global Optimization**: We want to decide how much fund to invest/keep in each of multiple pools at the same time. We cannot assume STGP = 1. We can borrow STGP from Binance or othre sources. We still need to predict STGpLP, though.
<br/><br/><br/><br/><br/><br/><br/><br/>

####2. APY. ROI vs. APY

ROI is very intuitive and linear, though it has no time frame. ROI of 0.25, for example, means 25% of profit from investment, but it doesn't imply the time frame during which the profit was earned. This is the place where APY come into play. APY assumes the same ROI is earned serially and compounded. You can fee it here: APY = (1+ROI) ** (APY_timeframe/ROI_timeframe) - 1, where APY_timeframe is one year by definition.
 **But we always use a fixed timeframe, of 15 minutes, so we don't seem to need APY**

Let's see what will happen if we choose ROI_timeframe to be 15 minutes. APY_timeframe/ROI_timeframe = 365 * 24 * 60 / 15 = 35,040. (Very large. How many 15-min timeframes a year has)

Here are some examples:
- (1 + 0.02) ** 35,040 - 1 = 2.238e * 10 ** 301
If our ROI ever happens to be 2% in 15 min, APY becomes a 301-figure number. Easy to be infected with error during calculation.

- (1 + 0.0001) ** 35,040 - 1 = 32.24
(1 + 0.0002) ** 35,040 - 1 = 1103.66
Here, 2X ROI does not give 2X APY. APY is has severe non-linearity. **It distorts reality over million times when ROI is large.**

- If users decide investment based on APY, then 2X ROI may get 1 million billion X liked, rather than 2X. They will get unexpected results. **This is the place where our AI comes into play** to give results consistent with intuition. The true intuition is ROI.
<br/>

**We will follow ROI, while nun-customer users follow APY, when decide our investment distribution.**