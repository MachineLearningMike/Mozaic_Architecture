
##<div align="center">Data Scheme for StarGate</div>

- This document **only defines** the farming performance gauge data of the StarGate bridge.
- This document does not specify optimized algorithms that collect and compute the data.

#### 1. Snapshots at a given time moment
- **STGpB~c~**: STG-per-Block on the chain c.
- **ALLOC~c,p~**: AllocPoint for the pool p on the chain c.
- **T_ALLOC~c~**: Total AllocPoint on the chain c.
- **USDpLP~c,p~**: USD-per-LP for the pool p on the chain c.
- **STK~c,p~**: Staked LP Tokens for the pool p on the chain c.
- **STGP**: STG price quote in USD
<br/>
- **STG_ACC~c,p~**: ACC reward that would have been accumulated for one unit of LP since the beginning of farming if the unit had been kept staked throughout, on the pool p on the chain c.
<br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/>


#### 2. Block-level data
- **B-Average** of feature X in a block is defined to be $1/n * \sum_{i=0}^{n-1} x_i$, where $\{x_i | i = 0, 1, ..., n-1\}$ are values that are held by X sequentially in the block and $x_k \neq x_{k+1}$ for all $k$. **Example**: If Balance has a constant value 30 in the block, then the B-Average of Balance is 30. If Balance has two values 30 and 40 in total sequentially in the block, then the B-Average of Balance is $1/2 * (30+40) = 35$
- **STGpB^b^~c~**, **ALLOC^b^~c,p~**, **T_ALLOC^b^~c~**,**USDpLP^b^~c,p~**, **STK^b^~c,p~**, and **STGP^b^**: The B-Average of relevant snapshot feature on a given block.
  <br/>
- **STGpLP^b^~c,p~**: STG-rewards-per-LP on a given block, equal to the increment of STG_ACC~c,p~ from that block's opening time to the next block's.
- **ROI^b^~c,p~**: Return-On-Investment on a given block, equal to the increment of USDpLP~c,p~ * STG_ACC~c,p~ from that block's opening time to the next block's.
- **APY^b^~c,p~**: Annual Profit Yield on a give block, equal to (1 + ROI^b^~c,p~) ** (year / AvgBlockTime~c~) - 1, where AvgBlockTime is the average time elapse that a block is minted.
<br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/>

#### 3. Timeframe-level data

- **Timeframe**: time interval $[T_a, T_b)$, where $T_a$ and $T_b$ are the Unix timestamp of an integer second. Note $T_b$ does *not* belong to the interval.
- **A block belongs to a timeframe** if and only if the timestamp of the block belongs to the timeframe.
- **BlockValue(X, B)** denotes the value of a feature X on the block B. **An examples** of BlockValue(X, B) is the B-Average.
- Some derived features around a feature X on a timeframe are defined as follow: <br/>**#-Sum** = $\sum_{i=0}^{n-1} BlockValue(X, B_i)$, <br/>**#-Product** = $\prod_{i=0}^{n-1} BlockValue(X, B_i)$, <br />**#-Average** = $1/n * \sum_{i=0}^{n-1} BlockValue(X, B_i)$, <br/>where $\{B_i | i = 0, 1, ..., n-1\}$ are all blocks that belong to the timeframe.
- **STGpB^a^~c~**, **ALLOC^a^~c,p~**, **T_ALLOC^a^~c~**, **T_ALLOC^a^~c~**, **SLP^a^~c,p~**, **USDpLP^a^~c,p~**, **STGpLP^a^~c,p~**, **STK^a^~c,p~**, and **STGP^a^**: The #-Average of relevant block-level feature on a timeframe.
<br/>
- **STGpLP^s^~c,p~**: The Sum of STGpLP^b^~c,p~, on a timeframe, for the pool p on the chain c.
- **ROI^s^~c,p~**: The Sum of ROI^b^~c,p~ on the timeframe for the pool p on the chain c.
- **APY^s^~c,p~**:= (1 + ROI^s^~c,p~) ** (year / FrameTime) - 1, where FrameTime is the length of timeframe.