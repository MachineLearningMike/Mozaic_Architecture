<br><br><br><br><br>

# <p style="text-align: center;">Architecture</p>
# <p style="text-align: center;">Mozaic System</p>

**<p style="text-align: center;">Development use</p>**

<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
**<p style="text-align: center;">mike@mozaic.finance</p>**

<div style="page-break-after: always;"></div>
<br><br>

**Table of Contents**
<br>

<!-- vscode-markdown-toc -->
* 1. [ Definitions and assumptions](#Definitionsandassumptions)
* 2. [Omnichain mLP token](#OmnichainmLPtoken)
	* 2.1. [Requirements](#Requirements)
	* 2.2. [Design decisions](#Designdecisions)
* 3. [Omnichain vaults](#Omnichainvaults)
	* 3.1. [Omnichain vault as smart contracts coordinated across chains](#Omnichainvaultassmartcontractscoordinatedacrosschains)
	* 3.2. [Limited responsibility](#Limitedresponsibility)
	* 3.3. [Local transportation to/from wallets](#Localtransportationtofromwallets)
	* 3.4. [ Reference source code adding requests](#Referencesourcecodeaddingrequests)
	* 3.5. [Vaults identified](#Vaultsidentified)
	* 3.6. [Logical components](#Logicalcomponents)
* 4. [Omnichain staking, or Cross-chain optimization](#OmnichainstakingorCross-chainoptimization)
	* 4.1. [Definition](#Definition)
	* 4.2. [Staking planner](#Stakingplanner)
		* 4.2.1. [Task definition](#Taskdefinition)
		* 4.2.2. [Notations](#Notations)
		* 4.2.3. [Formula](#Formula)
	* 4.3. [Transition planner](#Transitionplanner)
		* 4.3.1. [Regular asset move plans](#Regularassetmoveplans)
		* 4.3.2. [Definitions and notations](#Definitionsandnotations)
		* 4.3.3. [Design decisions](#Designdecisions-1)
		* 4.3.4. [Algorithm input and output](#Algorithminputandoutput)
		* 4.3.5. [Process](#Process)
	* 4.4. [Optimization rounds](#Optimizationrounds)
	* 4.5. [Protocol drivers](#Protocoldrivers)
		* 4.5.1. [Considerations](#Considerations)
		* 4.5.2. [Design decisions](#Designdecisions-1)
		* 4.5.3. [Symmetry between on-chain and off-chain modules](#Symmetrybetweenon-chainandoff-chainmodules)
		* 4.5.4. [Reference source code](#Referencesourcecode)
* 5. [Cross-chain transportation](#Cross-chaintransportation)
	* 5.1. [Considerations](#Considerations-1)
	* 5.2. [ Decentralized operations required](#Decentralizedoperationsrequired)
	* 5.3. [Employ the LayerZero service](#EmploytheLayerZeroservice)
	* 5.4. [Operations exemptible of decentralization](#Operationsexemptibleofdecentralization)
	* 5.5. [Design recommendations](#Designrecommendations)
	* 5.6. [An off-chain detour for inter-chain transportation](#Anoff-chaindetourforinter-chaintransportation)
* 6. [ Overall state transition](#Overallstatetransition)
	* 6.1. [ Design decisions](#Designdecisions-1)
	* 6.2. [ Visual description](#Visualdescription)
	* 6.3. [Reference source code](#Referencesourcecode-1)
* 7. [Miscellaneous](#Miscellaneous)
	* 7.1. [Compounding](#Compounding)
		* 7.1.1. [Considerations](#Considerations-1)
	* 7.2. [Design recommendations](#Designrecommendations-1)
	* 7.3. [Gas supply](#Gassupply)
		* 7.3.1. [Considerations](#Considerations-1)
		* 7.3.2. [Design recommendations](#Designrecommendations-1)
	* 7.4. [Auxiliary descriptions of the architecture](#Auxiliarydescriptionsofthearchitecture)
		* 7.4.1. [Deposit / Withdraw - deposits and rewards mixed 1:1](#DepositWithdraw-depositsandrewardsmixed1:1)
* 8. [Reference source code](#Referencesourcecode-1)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

<div style="page-break-after: always;"></div>
<br><br>


# <p style="text-align: center;">Architectural Decisions</p>

- Mozaic, the system, and Mozaic system refers to the software system that this project is going to develop, launch, operate, and maintain.
- This document describes architectural decisions that implement the [system requirements](https://github.com/MachineLearningMike/Mozaic_Architecture/blob/main/Architecture/Requiements.pdf).
- This document serves as an additional technical terminology of the project.
- This document is shared and maintained by team members.
<br><br>


##  1. <a name='Definitionsandassumptions'></a> Definitions and assumptions
<br>

- **System Asset Snapshot**, **system asset state**, **system asset/request state**, or simply **asset state**, is the state identified by the followings, at a given time:
<br>

- **Deposits**: The vector of all deposit requests. A deposit request can be modeled by:
```
    struct Deposit {    // sends assets to the system and receives mLP in return
        address user;       // the user who requests the deposit
        address to;         // wallet to receive mLP
        address token;      // the denominator token of the assets
        uint    token_chain // the chain that hosts the denominator token
        uint    amount;     // the amount of the assets in the denominator token
        uint    amountLP;   // the amount of mLP
        uint    lp_chain;   // the chain that hosts the mLP token
        uint    usd_equ;   // the usd-equivalent of the asset amount
    }
```
- **Withdrawals**: The vector of all withdrawal requests. A withdrawal request can be modeled by:
```
    struct Withdraw {   // sends mLP to the system and receives assets in return
        address user;       // the user who requests the withdrawal
        address to;         // wallet to receive assets
        address token;      // the denominator token of the assets
        uint    token_chain // the chain that hosts the denominator token
        uint    amount;     // the amount of the assets in the denominator token
        uint    amountLP;   // the amount of mLP
        uint    lp_chain;   // the chain that hosts the mLP token
        uint    usd_equ;   // the usd-equivalent of the asset amount
    }
```

- **Stakes**: The vector of all staking instances. A staking instance can be modeled by:
```
    struct Stake {   // a stake of asset is staked on a pool
        address token;      // the denominator token of the assets
        uint    token_chain // the chain that hosts the denominator token
        uint    amount;     // the asset amount in the denominator token
        uint    pool_id;    // the local/global pool ID
        uint    usd_equ;   // the usd-equivalent of the asset amount
    }
```

- **Rewards**: The vector of all reward instances. A reward instance can be modeled by:
```
    struct Reward {   // a reward is pending collecting on a pool
        address token;      // the denominator token of the assets
        uint    token_chain // the chain that hosts the denominator token
        uint    amount;     // the asset amount in the denominator token
        uint    pool_id;    // the local/global pool ID
        uint    usd_equ;   // the usd-equivalent of the asset amount
    }
```

- **Treasury**: Assets reserved for system operation, development, and management. Treasury consists of TreasuryItems. A treasury item can be modeled by:
```
    struct TreasuryItem {
        address token;      // the denominator token of the assets
        uint    token_chain // the chain that hosts the denominator token
        uint    amount;     // the amount of the assets in the denominator token
        uint    source;     // the source of treasury item, like performance fee, etc.
        uint    usd_equ;   // the usd-equivalent of the asset amount
    }
```

<br><br>


- **Pools state**

    $PoolState = ( \space PS_i \space | \space i=1..N \space)$,

    where $PS_i$ is the $i$-th pool state: 

    $PS_i = (RewardRate_i, RewardToken_i, TotalStake_i, StakingToken_i, MozaicStake_i, Price_i^R, Price_i^S)$

    , where 
    - $RewardRate_i$ is the amount of reward emitted during a given time frame
    - $RewardToken_i$ is the *user-visible* token type of reward
    - $TotalStake_i$ is the total amount of staked asset in the pool
    - $StakingToken_i$ is the *user-visible* token type of staking asset
    - $MozaicStake_i$ is the amount of asset staked in the name of Mozaic
    - $Price_i^R$ is the *average* price of reward token in the time frame
    - $Price_i^S$ is the *average* price of staking token in the time frame
    
    <br>

- **Staking tokens**
<br>

    $$StakingTokens = ( \space StakingToken_i \space | i=1..N \space)$$


- **Regularity of staking pool**

    We assume reward on $PS_i$ is calculated as:

    $MozaicReward_i = \frac {\normalsize RewardRate_i}{\normalsize TotakStake_i} \times MozaicStake_i $

<br><br>

##  2. <a name='OmnichainmLPtoken'></a>Omnichain mLP token
<br>

###  2.1. <a name='Requirements'></a>Requirements
<br>
Omnichain mLP requires that:

- The mLP token should exist on all listed chains.
- When the system **stake**s assets that a user **deposit**ed, the system returns mLP tokens to the user. The amount of the returned mLP token should represent the newly staked asset in the **Staking Stock** immediately after the asset is staked.
- A user can **withdraw** assets from the **Staking Stock**, in any listed token format on any listed chain, by first returning mLP tokens from their wallet to the system wallet. The amount of asset that is **withdraw**n is the portion of **Staking Stock** that is represented by the returned mLP tokens immediately before the asset is **withdraw**n.
<br><br>

Additional requirements:
- The mLP token cannot have initial supply
<br>This is to ensure that mLP has no features of security.

<br>

###  2.2. <a name='Designdecisions'></a>Design decisions
<br>

- mLP token contracts will be independent of vaults, except that local vaults should be able to mint and burn local mLP tokens
- mLP token contracts will be independent of administration
    - mLP tokens will be completely free from administrator or DAO. 
    - mLP tokens will not be upgradeable and rebased.
- mLP tokens will be cross-chain swapped 1:1
    - A 3rd party cross-chain transportation services will be used
    - Mint-burn mechanism will be used
    - Lock-release will not be used
- mLP token swap will be either completely successful or completely reverted on both the source chain and the destination chain
    - It will employ the same technique as Stargate's swap, **if we find no alternatives**.
<br>

<div style="page-break-after: always;"></div>
<br><br>

##  3. <a name='Omnichainvaults'></a>Omnichain vaults
<br>

We need a consolidated, omnichain module that implements the use case **Control asset move** identified in the requirements specification, solely and completely. We call the module the vault, because from users' perspective,
- the module keeps users' assets, control and logs moves of the assets and profits generated from the assets, and returns the assets with profits.
- no modules other than that module have privilege to carry out these tasks.

<br>

###  3.1. <a name='Omnichainvaultassmartcontractscoordinatedacrosschains'></a>Omnichain vault as smart contracts coordinated across chains
According to the requirements, vaults are exclusively responsible to de-centrally
- make all changes to **system assets**
- log all changes to **system assets**

The only way is to have smart contracts on chains cooperate with each other to form the omnichain omnichain vault module. We call them local vaults or vault contracts individually.
<br><br>

###  3.2. <a name='Limitedresponsibility'></a>Limited responsibility
<br>

According to the requirements, vaults do *not* have to 
- find the *best possible* asset state to **optimize asset/request state** to
- precisely execute asset staking requests coming from off-chain side, like transitionPlan identified in requirements, because there will not be negative profit
<br><br>

###  3.3. <a name='Localtransportationtofromwallets'></a>Local transportation to/from wallets
<br>

Local vaults are responsible to:

- Withdraw pending rewards
- Stake and un-stake on local staking pools
- Send/receive assets to/from other local vaults
<br><br>

- Pull assets from the user if a deposit request involves a home token
- Export a deposit request if it involves an away mLP token
- Import an exported deposit request if it involves a home mLP token
- Calculate and push mLP tokens to the user if a deposit request involves the home mLP 
<br><br>
- Pull mLP to the user if a withdrawal request involves the home mLP token
- Export a withdrawal request if it involves an away token
- Import an exported withdrawal request if it involves a home token
- Calculate and push asset to the user if a withdrawal request involves a home token
<br><br>


###  3.4. <a name='Referencesourcecodeaddingrequests'></a> Reference source code adding requests

```
pragma solidity ^0.8.0;

// imports
import "../libraries/lzApp/NonblockingLzApp.sol";
import "../libraries/stargate/Router.sol";
import "../libraries/stargate/Pool.sol";
import "./MozaicLP.sol";

// libraries
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

abstract contract SecondaryVault is NonblockingLzApp {


    // The caller submits _amount of _token, and wants MLP tokens on _chain.
    function addDepositRequest(address _token, uint _amount, uint _chain) external  {
        require(userTokens[_token] != 0 && _amount > 0, "Wrong token/amount");       
        _safeTransferFrom(_token, msg.sender, address(this), _amount);
        
        if (_chain == thisChain) { // The request is local-token for local-mLP
            pending.ds.push( Deposit(msg.sender, _token, _amount, 0) );
            // 0 for mLP amount to send to the user.
        } else { // The request is local-token for away-mLP
            pending.dsToExport.push(DepositToExport(msg.sender, _usdt(_token, _amount), _chain));
            // Foreign chain _chain will store this like: 
            // pending.dsImported.push(DepositImported(msg.sender, usdEq, 0));
            // 0 for the undefined mLP amount to send to the user
        }
    }

    // The caller submits _amount of mLP, and wants _token tokens on _chain chain.
    function addWithdrawalRequest(uint _amountLP, address _token, uint _chain) external  {
        require(userTokens[_token] != 0 && _amountLP > 0, "Wrong token/amount");       
        _safeTransferFrom(mLP, msg.sender, address(this), _amountLP);
        
        if (_chain == thisChain) { // The request is local-LP for local-token
            pending.ws.push( Withdrawal(msg.sender, _token, 0, _amountLP) );
            // 0 for the undefined amount of token to send to the user.
        } else { // The request is local-LP for away-token
            pending.wsToExport.push(WithdrawalToExport(msg.sender, _token, _amountLP, _chain));
            // Foreign chain _chain will store this like:
            // pending.wsImported.push(WithdrawalImported(msg.sender, _token, 0, _amountLP));
            // 0 for the undefined amount of token to send to the user
        }
    }
}
```

<br><br>

###  3.5. <a name='Vaultsidentified'></a>Vaults identified
<br>

We identify vaults through their surrounding modules interacting with them.
The external actors in the following use case diagram, together with their interactions with vaults are already described. We can now explore the use cases of vault.

<br>

<p align="center">
  <img src=".\Vault use cases 1.0.PNG" width="1280" title="vault use cases" style="page-break-after: avoid;">
</p>
<br>

- **_Deposit**: This happens at vault contracts when the **Deposit** use case is invoked at the system. Invoked by the **User wallet** with assets ready to deposit, this use case coordinates the following two use cases.
- **Book deposit**: This use case 
    - collects the assets from **User wallet**,
    - books the deposit request with the system,
    - and pauses the session of "_Deposit".
- **Finish deposit**: This use case 
    - resume the session of "_Deposit",
    - retrieves the booked deposit request, 
    - mints mLP tokens to cover the new assets, 
    - returns the mLP tokens to **User wallet**,
    - and helps **Control staking transition** stake the assets.
- **_Withdraw**: This is what happens at the level of vault contracts when the **Withdraw** use case is invoked at the system level. Invoked by **User wallet** with mLP tokens returned, this used case coordinates the following two use cases.
- **Book withdraw**: This use case 
    - collects the returned mLP tokens,
    - burns the collected mLP tokens,
    - books the withdrawal request with the system,
    - and pauses the session of "_Deposit".
- **Finish deposit**: This use case 
    - resume the session of "_Deposit",
    - retrieves the books withdrawal request,
    - subtract assets, as much as covered by the returned mLP tokens, from the total system assets, and
    - returns the assets to **User wallet**
- **Control staking transition**: This use case transitions to a new asset state by executing **transitionPlan** provided by **Staking optimizer**. (This is the most challenging part of vault implementation.) It does *collectively*, by using **Move staking asset**,
    - **Collect reward**,
    - Collect staked assets, to cover the assets to **Finish withdraw**,
    - **Finish deposit**,
    - **Finish withdraw**,
    - execute remaining part of **transitionPlan**
- **Control trading**: This use case executes **assetMovePlan** provided by **Trading optimizer**.
<br><br>


###  3.6. <a name='Logicalcomponents'></a>Logical components
<br>

The overall architectural requirements for vault was/is to **minimize vault as much as possible** leaving most compute to off-chain modules.

The design decisions are as illustrated in the following figure:
<br><br>
<p align="center">
  <img src=".\High-level functional modules 1.0.PNG" width="1280" title="high-level functional modules" style="page-break-after: avoid;">
</p>

Functional modules are described below:
- **Secondary vault contract**: This module is a local vault and deployed on each chain except the home chain.
- **Master vault contract**: This module is a special **Secondary vaults contract** and deployed on the home chain.
- **mLP token contract**: This module manages the mLP token balances of **User**s.
- **User wallet**: This is a blockchain wallet and identifies a **User**. **User**'s actions; like deposit and withdraw are authenticated/authorized with this wallet.
- **Treasury**: This is a blockchain wallet or a contract account, and a place to store and retrieve system treasury. It will be better if it is the account of a smart contract that only obeys vault contracts, for better decentralization.
- **Staking optimizer**: This is an off-chain module that can invoke **Master vault contract**s. This module is globally unique, calculates optimal **transitionPlan**s, and lets the master vault to execute the plans (in cooperation with secondary vaults).
    - Transparency debate: **User**s will not be able to track why the system chose particular **transitionPlan**s technically.
     - Security debate: If the calculation of **transitionPlan** is hacked or compromised, then the system will make a less-optimal staking maneuver.
    - Justification: Only the second of the following concerns becomes less transparent, leading to both un-assured best profitability and assured huge gas- and time- savings.
        - how much of what assets from which pool to which pool, is the move about
        - whether all the asset moves are securely and/or reasonably/optimally chosen
        - whether all the asset moves are securely executed and logged
        - whether the move logs are readily available to check later
        - whether the execution of transitionPlan is integrated
- **Trading optimizer**: This off-chain module is similar to **Staking optimizer**, except it relates to trading.
- **Adimin wallet**: This wallet is used to invoke **Master vault contract", in privilege, on behalf of the administrator.
- **Staking planner**: An integral component of **Staking optimizer**, this module predicts the next most profitable **staking_portfolio**, based on **poolsState** provided by **Pools tracker**. Running this module on-chain would enhance transparency, but would at the same time incur huge gas fees and effectively disable the system.
- **Transition planner**: An integral component of **Staking optimizer**, this module predicts the most efficient **transitionPlan**, which is the best procedure of asset move that implements the transitioning to a given **staking_portfolio**, based on the current **poolsState**.
- **Trading optimizer**: This is similar to **Staking optimizer**, except that it relates to trading.
- **Trading planner**: This is similar to **Staking planner**, except that it relates to trading.
- **Pools tracker**: A shared module between **Staking optimizer** and **Trading optimizer**, this module retrieves and tracks all relevant information from chains, like Reward Release Speed, and total Staked mLP of each pool. Running this module on-chain would enhance transparency, but would at the same time incur huge gas fees and effectively disable the system.

<br>


<div style="page-break-after: auto;"></div>

##  4. <a name='OmnichainstakingorCross-chainoptimization'></a>Omnichain staking, or Cross-chain optimization
<br>

###  4.1. <a name='Definition'></a>Definition
<br>
Omnichain staking requires that:

- Assets can be deposited in any listed token format on any listed chain, *all of users' choice*.
- Deposited assets can be swapped/transferred, and staked in any staking pool on any listed chain, *guided by the system's optimization plan*.
- Staked assets and rewards can be withdrawn in any listed token format on any listed chian, *all of users' choice*.
- Rewards collected can be swapped/transferred, and staked in any staking pool on any listed chain, *guided by the system's optimization plan.*
<br><br>

###  4.2. <a name='Stakingplanner'></a>Staking planner
<br>
Note. All errors, like numerical processing rounding and price slippage, are ignored at this stage of architectural design.
<br><br>

####  4.2.1. <a name='Taskdefinition'></a>Task definition

- Goal: Calculate the best staking portfolio off-chain in order to
    - Save vault contracts long calculations of staking optimization, thus to save gas.
    - Keep vault contracts insulated from future algorithm upgrades of staking optimization.
- Consideration
    - Input may not be idealistically consistent, because an idealistic snapshot of multiple chain states is impossible logically.
    - Output staking portfolio may not be completely/perfectly implemented, because input may have errors and there are unpredictable price slippages and change of fees.
<br>

- Input
    - Current **system asset/request state**
    - Current poolsState, defined below
<br>

- Output
    - optimal staking portfolio

<br>

####  4.2.2. <a name='Notations'></a>Notations
<br>

- **Asset vectors**

    For the **system asset/request state** snapshot taken just before the transition $t$, we can deduce the following asset and token vectors through *one-to-one* mapping in the same order.

    - **Deposit assets**, at transition index $t$
    
        $Deposits^t = (\space (D_i^t, \space, T_i) \space | \space D_i^t: \space deposited \space amount. \space T_i: \space token. \space)$

    - **Withdrawals assets**, at transition index $t$

        $Withdrawals^t = ( \space (\space - W_i^t, \space T_i) \space | \space W_i^t: withdrawal \space amount, \space T_i: \space token. \space)$

    - **Vector of collected rewards**, at transition index $t$

        $Rewards^t = ( \space (R_i^t, \space T_i) \space | \space D_i^t: reward \space amount,  \space T_i: \space token. \space)$

    - **Vector of staked assets**, at transition index $t$

        $Stakes^t = ( \space (S_i^t, \space T_i) \space | \space D_i^t: stake \space amount,  \space T_i: \space token. \space)$


- **Transformations**

    - **Transformation $USD^{+1}$**

        $USD^{+1}$ transforms an asset vector to USD-denominated asset vector.

        Example: $USD^{+}$ transforms (2 USDC, 3 ETH) to (2.02, 1300), assuming USD/USDC = 1.01 and USD/ETH = 1300.

    - **Transformation $USD^{-1}$**

        $USD^{-1}_{TK}$ transforms a USD-denominated asset vector and a token vector TK to tokens-denominated vector.

        Example: $USD^{-1}_{(USDC, ETH)}$ transforms (2.02 USD, 1300 USD) to (2 USDC, 3 ETH), assuming USD/USDC = 1.01 and USD/ETH = 1300.

    - **Transformation $FOP$** - the core of the Archimedes algorithm

        $FOP$, standing for Find Optimal Portfolio, finds the *best* USD-denominated **vector of staked assets** for a given total USD amount. In other words, *it finds what amounts of (USD-denominated) value should be allocated to what staking pools, provided that the total (USD-denominated) value is given*.  **best** is relative and subjective.

    - **Transformation $Sum$**

        $Sum$ sums up all elements of a vector when they are denominated by the same token.
<br>
- **Asset snapshot**, at transition $t$

    - Asset snapshot just before the transition $t$:

        $AS^t = (Deposits^t, \space Withdrawals^t, Rewards^t, Stakes^t)$

        Or, simply, $AS^t = (D^t, \space W^t, R^t, S^t)$

    - Asset snapshot just after the transition $t$:

        $AS^{t+} = (0Deposits^t, \space 0Withdrawals^t, 0Rewards^t, optimal \space Stakes^t)$

        Or, simply, $AS^{t+} = (0D^t, \space 0W^t, 0R^t, optimal \space S^t)$

        , where 0D, 0D, and 0R are a vector of zero valued elements of their respective types.

<br>

####  4.2.3. <a name='Formula'></a>Formula

If a state transition $t$ is optimal, the following diagram holds: <br><br>

$$\begin{CD} \space \space  \space \space \space \space AS^t = (D^t,\space W^t,\space R^t,\space S^t) @> (Resulting \space transition) >> AS^{t+} = (0D,\space 0W,\space 0R,\space optimal \space S^t)  \space  \space \space \space \space \space \space \space \space \\ @V USD^{+1} VV @A zeros, \space USD^{-1}_{StakingTokens} AA \\  AS_U^t = (USD^{+1}(D^t), \space USD^{+1}(W^t), ... ) @>> (Implicit) > AS_U^{t+} = (0D,\space 0W,\space 0R,\space FOP(Total \space in \space USD)) \space \space \space \space \space \space \space \space \space \space\space \\ @V Sum VV @A zeros, \space {FOP} AA \\ Total \space in \space USD @> Identity >> Total \space in \space USD \end{CD}$$
<br>

The algorithm for Staking planner $AS^t$ is a chain of transformations:

Optimal Staking Portfolio =
<br>
$optimial \space S^{t} = USD^{-1} \circ FOP \circ Sum \circ USD^{+1} (T, \space D^t, \space - \space W^t, \space R^t, \space S^t)$ <br><br>

- $USD^{+1}$ and $USD^{-1}$ are obvious, except that we may need systematic methods to find best Dexes and swap paths.
- An analytical version of $FOP$ demonstrated 9% of competitive edge over the public. *A Machine Learning version should give higher edge.*
- $Sum$ is trivial.
- $D^t$ and $W^t$ can be retrieved from the booked requests of deposits and withdrawals.
- $S^t$ is found when we "Collect reward" pending rewards.
<br>


**The formula essentially does**:
- Sum up all asset amounts available for the new staking:
    ```
    = the **Staking Stock** (i.e. staked assets plus pending rewards),
    + assets received from deposit users,
    - assets to send to withdrawing users.
    ```
- Find the USD-equivalent of the sum assets: Total_in_USD,
- Allocate the Total_in_USDT **optimally** across all staking pools, by using the $FOP$ algorithm, Note: **optimally** is relative and subjective.
- Transform the allocated USD amounts back to the native tokens on the staking pools.

<br><br>

<div style="page-break-after: auto;"></div>


###  4.3. <a name='Transitionplanner'></a>Transition planner

<br>

####  4.3.1. <a name='Regularassetmoveplans'></a>Regular asset move plans
<br>
An asset move plan is a set of elementary asset move instructions. We need to eliminate redundant value flows from asset move plans to save the cost of executing the plan.
<br> 

A regular asset move plan as a plan that has no redundant value flows. **For any asset move plan, there exists a regular equivalent of the original plan. It should be unique(?) and easy to find if we introduce an external asset place as the hub.**

Below comes two example of asset move plan: an irregular plan and its regular equivalent:

<p align="center">
  <img src=".\Irregular asset moves.PNG" width="1280" title="high-level use cases" style="page-break-after: avoid;">
</p>

<p align="center">
  <img src=".\Regular asset moves.PNG" width="1280" title="high-level use cases" style="page-break-before: avoid;">
</p>

<br><br>

####  4.3.2. <a name='Definitionsandnotations'></a>Definitions and notations
<br>

- **Asset vectors**

    For the **system asset/request state** snapshot taken just before the transition $t$, we can deduce the following asset and token vectors through *one-to-one* mapping in the same order.

    - **Deposit assets on a given chain**, at transition index $t$
    
        $Deposits^t_c = (\space (D_i^t, \space, T_i) \space | \space D_i^t: \space deposited \space amount. \space T_i: \space token \space on \space chain \space c. \space)$

    - **Withdrawals assets on a given chain**, at transition index $t$

        $Withdrawals^t_c = ( \space (- W_i^t, \space T_i) \space | \space W_i^t: withdrawal \space amount, \space T_i: \space token \space on \space chain \space c. \space)$

    - **Vector of collected rewards on a given chain**, at transition index $t$

        $Rewards^t_c = ( \space (R_i^t, \space T_i) \space | \space D_i^t: reward \space amount, \space T_i: \space token \space on \space chain \space c. \space)$

    - **Vector of staked assets on a given chain**, at transition index $t$

        $Stakes^t_c = ( \space (S_i^t, \space T_i) \space | \space D_i^t: stake \space amount,  \space T_i: \space token \space on \space chain \space c. \space)$

    - **Vector of assets on a given chain**, at transition index $t$

        - Asset snapshot just before the transition $t$:

            $VA^t_c = (Deposits^t_c, \space Withdrawals^t_c, Rewards^t_c, Stakes^t_c)$

            Or, simply, $VA^t_c = (D^t_c, \space W^t_c, R^t_c, S^t_c)$

        - Asset snapshot just after the transition $t$:

            $VS^{t+}_c = (0Deposits^t_c, \space 0Withdrawals^t_c, 0Rewards^t_c, optimal \space Stakes^t_c)$

            Or, simply, $VA^{t+}_c = (0D^{t+}_c, \space 0W^{t+}_c, 0R^{t+}_c, optimal \space S^t)$

            , where 0D, 0D, and 0R are a vector of zero valued elements of their respective types.
<br>

####  4.3.3. <a name='Designdecisions-1'></a>Design decisions
<br>

**We deduce the following design decisions:**
<br>
- Asset moves will be **regularized**. We believe regularization will reduce the total cost of asset moves and the number of inter-chain swap/transfers.
- **ChainValutWallet** will be the hub for regular **Intra-Chain Moves**. This means an **Intra-Chain Move** will be between the **ChainValutWallet** and another of **ChainAssetPlaces**. We wil *not* allow direct asset moves between **ChainAssetPlaces** that are not **ChainVaultWallet**.
- We need one special vault that oversees the cooperation between vaults, including itself, de-centrally.
    - **Master vault**: the special vault
    - **Master chain**: the chain that hosts the master vault
- The **Master vault** will be the hub for regular **Inter-Chain Moves**. This means an **Inter-Chain Move** will between the **Master vault** and another of local vaults. We wil *not* allow direct asset moves between **ChainAssetPlaces** of different chains.
- This algorithm will be executed off-chain to produce a transition plan, because
<br>
    - it will save huge gas fees that the algorithm would spend if it ran on-chain
    - the requirements don't require decentralization-level of asset move planning
<br>

**Note**: The off-chain execution of this algorithm raises the concerns of decentralization.

<br>

####  4.3.4. <a name='Algorithminputandoutput'></a>Algorithm input and output
<br>

- Input: 
    - Target staking portfolio, given in the extended form of 
    <br>
    $$VA^{t+}_c = (0D^{t+}_c, \space 0W^{t+}_c, 0R^{t+}_c, optimal \space S^t)$$
    - Current staking portfolio, given in the extended form of 
    <br>
    $$VA^t_c = (D^t_c, \space W^t_c, R^t_c, S^t_c)$$
<br>
- Output: 
    - A transition plan generated
    - The transition plan executed

####  4.3.5. <a name='Process'></a>Process

- Find 
    $$Asset \space Surplus_c = VA^t_c - VA^{t+}_c$$
    or, in other words, $$Asset \space Surplus_c \space = \space Current \space amounts \space - \space Target \space amounts$$
    for all chian c.
    <br>
    - Asset surplus element is, therefore, defined for each of deposit, withdrawal, staking, and reward places on all chian.
    - The elements are called an **asset surplus** or simply a **surplus**.
    - An asset place is a giving place if it has a positive surplus, and a taking place if it has a negative surplus.
    - **The goal of transitioning is to take surplus amounts from giving places, and divide them to taking places.**
    - A deposit, as an asset place, has always a positive surplus and is a giving place, because its current amount is positive and the target amount is zero (i.e. we have to empty the place). The source will be the local vault when the request is pending processing.
    - A withdrawal, as an asset place, has always a negative surplus and is a taking place, because its current amount is negative (debt) (when the request is pending processing and the debt amount has been calculated) and the target amount is zero (i.e. we have to settle the debt). It has the destination of assets. The destination is be the "to" wallet of the request.
    - A reward, as an asset place, has always a positive surplus and is a giving place, because its current amount is positive and the target amount is zero (i.e. we have to empty the place). The source will be the local vault when the request has been collected.
    - A stake, as an asset place, has either a positive, zero, or negative surplus, because the target amount may be less, equal, or greater than the current amount. The source is the staking pool combined with the staking/un-staking method.

- Classify all asset places on al chains into giving and taking places, again.
    - If the asset surplus is significantly greater than zero, it is a **giving surplus** and it is the giving amount.
    - If the asset surplus is significantly less than zero, it is a **taking asset surplus** and its absolute value is the taking amount.
    - Else, it is a neutral asset place.
    - An asset place has has either
        - a positive **giving amount** and a zero taking amount,
        - a positive **taking amount** and a zero giving amount,
        - or, a zero giving amount and a zero taking amount.

- Classify chains into giving chains and taking chains
    - If the sum of giving amounts on a given chain is significantly greater than the sum of taking amount, it is a **giving chain**, and the absolute difference is called the **giving amount**, or surplus, of the giving chain.
    - If the sum of taking amounts on a given chain is significantly greater than the sum of giving amount, it is a **taking chain** and the absolute difference is called the **taking amount**, or deficit, of the taking chain.
    - Else, it is a neutral chian
    - A chain has either
        - a positive giving amount and a zero taking amount,
        - a positive taking amount and a zero giving amount,
        - or a zero giving amount and a zero taking amount.
        
- Generate a regular **intra-chain asset move plan** for giving chains
    - Collect the local swap prices and fees
    - Collect giving amounts of all giving asset places to the vault.
    - Swap, divide, and send the collected amount to fill the taking amounts of taking asset places. *Put priority on instances in $W^t_c$ and make sure to fill in them.*
    - Regularize the plan. (See previous sections)

- Execute the regular intra-chain asset move plans for giving chains.
    - The giving, surplus amount of giving chains should remain in their vault.

- Generate a regular **inter-chain asset move plan** that divides giving amount of assets of giving chains to taking chains.
    - Collect giving amounts of giving chains from their vaults to the master vault.
    - Swap, divide, and send the collected amount to fill in the taking amounts of taking chains at their vaults.
    - Regularize the plan.

- Execute the regular inter-chain asset move plan.
    - There should not remain assets in the master vault, except dusts.

- Generate a regular intra-chain asset move plan for taking chains
    - Collect the local swap prices and fees.
    - Note: the vault has already got some assets that came from giving chains.
    - Collect giving amounts of all giving asset places to the vault.
    - Swap, divide, and send the collected amount to fill the taking amounts of taking asset places. *Put priority on instances in $W^t_c$ and make sure to fill in them.*
    - Regularize the plan. (See previous sections)

- Execute regular intra-chain asset move plans for taking chains.
    - There should not remain assets in the their vaults, except dusts.

<br>

Note: This algorithm should be tweaked to cope with changing price slippages and fees, and numerical dusts, in implementation phases.
<br>

**The algorithm is illustrated below:**
<br>
<p align="center">
  <img src=".\Transition algorithm.PNG" width="1280" title="high-level functional modules" style="page-break-after: avoid;">
</p>

###  4.4. <a name='Optimizationrounds'></a>Optimization rounds

- On-chain: Take a snapshot of **system asset/request state**
- Off-chain: Calculate and send mLP tokens to users who requested deposit
- Off-chain: Calculate the amount of tokens to send to users who requested withdrawal
- Off-chain: Call Staking Planner to generate Optimal Staking Portfolio
- Off-chain: Call Transition Planner to generate Optimal Transition Plan
- On-chain: Execute the Optimal Transition Plan

###  4.5. <a name='Protocoldrivers'></a>Protocol drivers

####  4.5.1. <a name='Considerations'></a>Considerations


####  4.5.2. <a name='Designdecisions-1'></a>Design decisions

<br>

<p align="center">
  <img src=".\Driver scheme.PNG" width="1280" title="vault use cases" style="page-break-after: avoid;">
</p>
<br>

####  4.5.3. <a name='Symmetrybetweenon-chainandoff-chainmodules'></a>Symmetry between on-chain and off-chain modules

TransitionPlanner will have the correspondent layer structure:
- Layer3: correspondent of vaults, that don't know about action type.
- Layer2: correspondent of drivers' Exexute(.) function, which doesn't know action parameter structure.
- Layer1: correspondent of drivers' internal action functions, like _Swap(.), which know action parameter structure.
<br>

<p align="center">
  <img src=".\Protocol Scheme Symmetry.PNG" width="1280" title="vault use cases" style="page-break-after: avoid;">
</p>
<br>


####  4.5.4. <a name='Referencesourcecode'></a>Reference source code

**ProtocolDriver**

```
abstract contract ProtocolDriver is Ownable {
    address public vault;

    function SetVault(address _vault) external virtual onlyOwner {
        require(vault != address(0), "Wrong vault address");
        vault = _vault;
    }

    modifier delegatedByVault() {
        require(address(this) == vault, "Wrong vault");  // this: Assuming delegatecall.
        _;
    }

    function Execute(bytes calldata action) external virtual delegatedByVault {
    }
}
```

**Vaults**
```
    ... ... ...
    mapping (uint => address) public protocolDrivers;

    function ChangeProtocolDriver(uint protocol, address driver) external onlyOwner {
        protocolDrivers[protocol] = driver;
    }

    function ExecuteActions(bytes[] calldata protocolActions) external onlyOwner {
        for (uint i = 0; i < protocolActions.length ; i++) {
            (uint protocol, bytes memory action) = abi.decode(protocolActions[i], (uint, bytes));
            (bool success, bytes memory data) = protocolDrivers[protocol]
            .delegatecall(abi.encodeWithSignature(("Execute(bytes)"), action));
            require(success, "");
        }
    }
    ... ... ...
```

**Driver example**
```
contract StargateDriver is ProtocolDriver {
    ... ... ...

    function Execute(bytes calldata action) external virtual override delegatedByVault {
        (ActionType actionType, bytes memory params) = abi.decode(action, (ActionType, bytes));

        if(actionType == ActionType.Stake) {
            _Stake(params);
        } else if(actionType == ActionType.Unstake) {
            _Unstake(params);
        }
    }

    function _Stake(bytes calldata params) internal virtual {
        (address token, uint pool, uint special_for_stagate_staking) = abi.decode(params, (address, uint, uint));

        // do whatever ...
        // You are free to introduce any (protocol x action)-specific param, like special_for_stagate_staking
        // because you have StargateDriver correspondent on the off-chain side (TransitionPlanner)
    }

    function _Unstake(bytes calldata params) internal virtual {
    }

    ... ... ...
}
```


<div style="page-break-after: auto;"></div>
<br><br>

##  5. <a name='Cross-chaintransportation'></a>Cross-chain transportation
<br>

###  5.1. <a name='Considerations-1'></a>Considerations

- Decentralized inter-chain transportation is required for omnichain-ness
- Decentralized inter-chain transportation may lead to bad User Experience, for its inherent long asynchronous operation
- As such, we need to reduce the use of inter-chain transportation as possible
- Layer Zero is the de facto industry standard of decentralized inter-chain transportation service
- There are total 6 cross-chain calls between the master vault and a (local) vault when the system carries out a round of optimization. (See below diagrams.)
    - If the system is deployed on 10 chains and optimizes 24 times a day, we will have **1,440 cross-chain calls a day**.
    - The 6 cascaded cross-chain calls may pose significant risks to integrity/consistency and User Experience, like runtime responsiveness and coding/maintenance complexity.
- If we compromise on the integrity/consistency of optimization (not on asset moves), by adopting off-chain version of executing transition plans and thus exposing the system to rarely feasible hacking/attacks, then cross-chain calls will be cut down 50%, in return. (See below diagrams.)
- **Not all inter-chain transportation need to be decentralized** (explained below)
<br>

###  5.2. <a name='Decentralizedoperationsrequired'></a> Decentralized operations required
Some vault operations should be decentralized to meet the requirements. Tracking of asset/mLP amount should be executed decentrally, without intermediate off-chain agent or relayer, and with transparent loggs
- Collecting pending rewards from all staking pools to vaults
- Withdrawing from and staking to staking pools to/from vault
- Query for local amounts of assset and mLP token
- Cooperation between vaults to calculate and exchange the information of asset/mLP amounts and indexes derived therefrom
- Sending assets from users' wallets to vaults, on users' deposit requests
- Calculating mLP amount to send to users, in return for their deposited assets
- Sending mLP tokens from vaults to users' wallets, on users' deposit requests
- Sending mLP from users' wallets to vaults, on users' withdrawal requests
- Calculating asset amount to send to users, in return for their returned mLP tokens
- Sending assets from vaults to users' wallets, on users' withdrawal requests
<br><br>

###  5.3. <a name='EmploytheLayerZeroservice'></a>Employ the LayerZero service

Transparent cross-chain transportation is the fundamental basis of omnichain operations. We choose LayerZero service for our cross-chain transportation.

<p align="center">
  <img src=".\Mozaic and LayerZero.PNG" width="1280" title="vault use cases" style="page-break-before: avoid;">
</p>
<br>

###  5.4. <a name='Operationsexemptibleofdecentralization'></a>Operations exemptible of decentralization
Vaults cooperation for staking optimization **does not have to be decentralized**, in the meaning that the optimization doesn't have to provide ideal maximum profit nor have to be successful
- Collecting pools information from chains to off-chain modules, could be done by off-chain modules
- Sending asset move plana to chains, could take detour via Mozaic off-chain modules with Admin wallets, at the risk of 
    - the plan could be tempered by (inauditable/unautidited) Mozaic modules or hackers. (But the plan itself is calculated by off-chain modules.)
    - the plan may even fail to be conveyed. (But this type of off-chain failture can also happen when we don't employ off-line detours.)
- Relaying requests between local vaults, during transitioning to a new staking, could take detour via Mozaic off-chain modules with Admin wallets, with the same risks as above
<br>

###  5.5. <a name='Designrecommendations'></a>Design recommendations

- We will choose *decentralized* inter-chain transportation between off-chain modules and vault contracts when finding new optimal staking portfolio and transitioning to the new staking
- Inter-chain messages, once identified as required, will carry as much information as possible.
- Read the section "Reference source code" for more.

<p align="center">
  <img src=".\Get_asset_state Sequence.PNG" width="1280" title="vault use cases" style="page-break-before: avoid;">
</p>
<br>

<p align="center">
  <img src=".\Generate optimal transition plan.PNG" width="1280" title="vault use cases" style="page-break-before: avoid;">
</p>
<br>

<p align="center">
  <img src=".\Execute staking transition plan.PNG" width="1280" title="vault use cases" style="page-break-before: avoid;">
</p>
<br>

###  5.6. <a name='Anoff-chaindetourforinter-chaintransportation'></a>An off-chain detour for inter-chain transportation
- An off-chain module monitors event logs of a smart contract for a target event happening
- Once detected, the event will be consumed by off-chain modules to produce response
- The produces response will be sent to a proper smart contract

<div style="page-break-after: auto;"></div>
<br>

##  6. <a name='Overallstatetransition'></a> Overall state transition
<br>

###  6.1. <a name='Designdecisions-1'></a> Design decisions

We choose the **Toggle-Between-Optimize-and_Stay** model for the overall system behavior. 
- The system will not always be transitioning, but staying most of the time accepting deposit/withdrawal requests from users.
- If the system **accept**s a deposit request, it 
    - collects the asset the user wants to deposit, on the asset's chain,
    - tells the user to wait until the next optimization round, when the system will send some mLP tokens to the user in return for the asset,
    - and book the request with the system for later processing.
- If the system **accept**s a withdrawal request, it
    - collects the mLP tokens the user wants to return, on the mLP's chain,
    - tells the user to wait until the next optimization round, when the system will send some assets of the requested token type in return for the mLP tokens,
    - and book the request with the system for later processing.
- The system will **optimize system asset/request state**, or **transition to new staking** at regular or irregular intervals. The frequency of optimization rounds will be optimized, as frequent moves of asset may incur more costs while infrequent optimization rounds will hinder from quick maneuver of staking.
- When an optimization round is requested, the system leaves the **Staying** state and enters the **Optimizing** state.
- When entering the **Optimizing** state, or an **Optimization round**, the system takes a **system asset/request snapshot**. Then the system transforms/changes the state to an optimal **system asset state** for more rewards, while continuing to **accept** deposit/withdrawal requests, which will be handled at the next round of optimization.
- When an optimization round is finished, the system leaves the **Optimizing** state and enters the **Staying** state.
- In the **Staying** state, the system does nothing but continues to **accept** deposit/withdrawal requests, which will be handled at the next round of optimization.

<br><br>

###  6.2. <a name='Visualdescription'></a> Visual description
<br>

The **Toggle-Between-Optimize-and_Stay** model of behavior can be expressed in a UML State Machine diagram shown below:
<br><br>

<p align="center">
  <img src=".\High-leve state machine 1.0.PNG" width="1280" title="high-level use cases" style="page-break-after: avoid;">
</p>

###  6.3. <a name='Referencesourcecode-1'></a>Reference source code

```
abstract contract SecondaryVault is NonblockingLzApp {
    ... ... ...

    struct Deposit {
        address user;
        address token;
        uint    amount;
        uint    amountLP;   // undefined initially
    }

    struct DepositImported {
        address user;
        uint    usdEq;
        uint    amountLP;   // undefined initially
    }

    struct DepositToExport {
        address user;
        uint    usdEq;
        uint    chainId;
    }

    struct Withdrawal {
        address user;
        address token;
        uint    amount;    // undefined initially
        uint    amountLP;
    }

    struct WithdrawalImported {
        address user;
        address token;
        uint    amount;    // undefined initially
        uint    amountLP;
    }

    struct WithdrawalToExport {
        address user;
        address token;
        uint    amountLP;
        uint    chainId;
    }

    struct Workspace {
        Deposit[] ds;
        DepositToExport[] dsToExport;
        DepositImported[] dsImported;
        Withdrawal[] ws;
        WithdrawalToExport[] wsToExport;
        WithdrawalImported[] wsImported;
    }

    Workspace private pending;
    Workspace private staged;

    uint public thisChain;
    address public mLP;

    function _safeTransferFrom(
        address _token,
        address _from,
        address _to,
        uint256 _value
    ) private {
        // bytes4(keccak256(bytes('transferFrom(address,address,uint256)')));
        (bool success, bytes memory data) = _token.call(abi.encodeWithSelector(0x23b872dd, _from, _to, _value));
        require(success && (data.length == 0 || abi.decode(data, (bool))), "transfer failed");
    }

    function _clean(Workspace storage ws) internal {

    }

    // This call begins transitioning.
    function takeSnapshot() external {
        Workspace storage temp = staged;
        staged = pending;
        pending = temp;
        _clean(pending);

        // do whatever with staged

    }
}
```


<div style="page-break-after: auto;"></div>
<br>


##  7. <a name='Miscellaneous'></a>Miscellaneous

###  7.1. <a name='Compounding'></a>Compounding

####  7.1.1. <a name='Considerations-1'></a>Considerations
- Rewards should be compounded as frequently as possible, unless the gas fees grows larger than rewards
- Compounding can be executed either:
    - at stocking transition rounds
    - or at its own intervals

###  7.2. <a name='Designrecommendations-1'></a>Design recommendations
- Leave it open

###  7.3. <a name='Gassupply'></a>Gas supply

####  7.3.1. <a name='Considerations-1'></a>Considerations

- Optimization will spend significant amount of gas, although we minimize vaults
- Gas is spent chain-wise, although the profit generation is not necessarily chain-wise

####  7.3.2. <a name='Designrecommendations-1'></a>Design recommendations
- The initial version will be sourcing local gas fees from the local Staking Stock. **If the local Staking Stock is not sufficient for gas fees, the chain will be set inactive**.
- Future versions will maintain a distributed treasure manager to provide local gas spending.

###  7.4. <a name='Auxiliarydescriptionsofthearchitecture'></a>Auxiliary descriptions of the architecture

####  7.4.1. <a name='DepositWithdraw-depositsandrewardsmixed1:1'></a>Deposit / Withdraw - deposits and rewards mixed 1:1
- We maintain a variable $deposits$ per user.
- Alice's $deposits$ is now 170 stable coins.
- Alice deposits 30 stable coins,
    - Her $deposits$ increases by 30 to become 200.
    - Her total mLP token increases by some amount of mLP token that is calculated to be the share of 30 in the system's resulting renewed Staking Stock.
- When she wants to withdraw with 20 mLP tokens
    - We first find she has a total 100 mLP tokens
    - Decrease her $deposits$ by 200 * 20 / 100 = 40 stable coins
    - Return the withdrawal assets to her, say 120 stable coins, which is a mix of deposits and rewards
    - We know she withdraws 40 principal deposit, together with 120 - 40 = 80 rewards
- **Now, the performance fees = 80 * 10% = 8 stable coins.**
- This means the withdrawal amount is forced to be a 1:1, which is not numerical but proportional, mix of original/principal deposits and rewards generated by staking/compounding them.
- The system will not serve other mix ratios but 1:1, because a ratio is meaning less as $deposits$ and rewards are all mixed and work together with constant compounding.


##  8. <a name='Referencesourcecode-1'></a>Reference source code

Below comes reference code that sketches and/or decides the code architecture.

```
pragma solidity ^0.8.0;

// imports
import "../libraries/lzApp/NonblockingLzApp.sol";
import "../libraries/stargate/Router.sol";
import "../libraries/stargate/Pool.sol";
import "./MozaicLP.sol";
import "./ProtocolDriver.sol";

// libraries
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

abstract contract ProtocolDriver is Ownable {
    address public vault;

    function SetVault(address _vault) external virtual onlyOwner {
        require(vault != address(0), "Wrong vault address");
        vault = _vault;
    }

    modifier delegatedByVault() {
        require(this == vault, "Wrong vault");  // this: Assuming delegatecall.
        _;
    }

    function Execute(bytes calldata action) external virtual delegatedByVault {
    }
}

abstract contract SecondaryVault is NonblockingLzApp {
    function _usdt(address _token, uint _amount) internal returns (uint usdEq) {
        // use whatever source of price to get usdt-equivalent of 
        // the _amount amount of _token token.
    }

    mapping(address => uint) public userTokens;

    function addUserToken(address _token) external {
        require( _token != address(0), "");
        userTokens[_token] = 1;
    }
    function removeUserToken(address _token) external {
        require( _token != address(0), "");
        userTokens[_token] = 0;
    }

    struct Deposit {
        address user;
        address token;
        uint    amount;
        uint    amountLP;   // undefined initially
    }

    struct DepositImported {
        address user;
        uint    usdEq;
        uint    amountLP;   // undefined initially
    }

    struct DepositToExport {
        address user;
        uint    usdEq;
        uint    chainId;
    }

    struct Withdrawal {
        address user;
        address token;
        uint    amount;    // undefined initially
        uint    amountLP;
    }

    struct WithdrawalImported {
        address user;
        address token;
        uint    amount;    // undefined initially
        uint    amountLP;
    }

    struct WithdrawalToExport {
        address user;
        address token;
        uint    amountLP;
        uint    chainId;
    }

    struct Workspace {
        Deposit[] ds;
        DepositToExport[] dsToExport;
        DepositImported[] dsImported;
        Withdrawal[] ws;
        WithdrawalToExport[] wsToExport;
        WithdrawalImported[] wsImported;
    }

    Workspace private pending;
    Workspace private staged;

    uint public thisChain;
    address public mLP;

    function _safeTransferFrom(
        address _token,
        address _from,
        address _to,
        uint256 _value
    ) private {
        // bytes4(keccak256(bytes('transferFrom(address,address,uint256)')));
        (bool success, bytes memory data) = _token.call(abi.encodeWithSelector(0x23b872dd, _from, _to, _value));
        require(success && (data.length == 0 || abi.decode(data, (bool))), "Stargate: TRANSFER_FROM_FAILED");
    }

    function _clean(Workspace storage ws) internal {

    }

    // This call begins transitioning.
    function takeSnapshot() external {
        Workspace storage temp = staged;
        staged = pending;
        pending = temp;
        _clean(pending);

        // do whatever with staged

    }

    // The caller submits _amount of _token, and wants MLP tokens on _chain.
    function addDepositRequest(address _token, uint _amount, uint _chain) external  {
        require(userTokens[_token] != 0 && _amount > 0, "Wrong token/amount");       
        _safeTransferFrom(_token, msg.sender, address(this), _amount);
        
        if (_chain == thisChain) { // The request is local-token for local-mLP
            pending.ds.push( Deposit(msg.sender, _token, _amount, 0) );
            // 0 for mLP amount to send to the user.
        } else { // The request is local-token for away-mLP
            pending.dsToExport.push(DepositToExport(msg.sender, _usdt(_token, _amount), _chain));
            // Foreign chain _chain will store this like: 
            // pending.dsImported.push(DepositImported(msg.sender, usdEq, 0));
            // 0 for the undefined mLP amount to send to the user
        }
    }

    // The caller submits _amount of mLP, and wants _token tokens on _chain chain.
    function addWithdrawalRequest(uint _amountLP, address _token, uint _chain) external  {
        require(userTokens[_token] != 0 && _amountLP > 0, "Wrong token/amount");       
        _safeTransferFrom(mLP, msg.sender, address(this), _amountLP);
        
        if (_chain == thisChain) { // The request is local-LP for local-token
            pending.ws.push( Withdrawal(msg.sender, _token, 0, _amountLP) );
            // 0 for the undefined amount of token to send to the user.
        } else { // The request is local-LP for away-token
            pending.wsToExport.push(WithdrawalToExport(msg.sender, _token, _amountLP, _chain));
            // Foreign chain _chain will store this like:
            // pending.wsImported.push(WithdrawalImported(msg.sender, _token, 0, _amountLP));
            // 0 for the undefined amount of token to send to the user
        }
    }



    mapping (uint => address) public protocolDrivers;

    function ChangeProtocolDriver(uint protocol, address driver) external onlyOwner {
        protocolDrivers[protocol] = driver;
    }

    function ExecuteActions(bytes[] calldata protocolActions) external onlyOwner {
        for (uint i = 0; i < protocolActions.length ; i++) {
            (uint protocol, bytes memory action) = abi.decode(protocolActions[i], (uint, bytes));
            (bool success, bytes memory data) = protocolDrivers[protocol]
            .delegatecall(abi.encodeWithSignature(("Execute(bytes)"), action));
            require(success, "");
        }
    }
}


contract StargateDriver is ProtocolDriver {
    function Execute(bytes calldata action) external virtual override delegatedByVault {
        (ActionType actionType, bytes memory params) = abi.decode(action, (ActionType, bytes));

        if(actionType == ActionType.Stake) {
            _Stake(params);
        } else if(actionType == ActionType.Unstake) {
            _Unstake(params);
        }
    }

    function _Stake(bytes calldata params) internal virtual {
        (address token, uint pool, uint special_for_stagate_staking) = abi.decode(params, (address, uint, uint));

        // do whatever ...
        // You are free to introduce any (protocol x action)-specific param, like special_for_stagate_staking
        // because you have StargateDriver correspondent on the off-chain side (TransitionPlanner)
    }

    function _Unstake(bytes calldata params) internal virtual {
    }
}
```

