<br><br><br><br><br><br><br><br>

# <p style="text-align: center;">Architectural Decisions</p>


<div style="page-break-after: always;"></div>
<br><br><br><br>

**Table of Contents**
<br><br>

<!-- vscode-markdown-toc -->
* 1. [Overall state transition](#Overallstatetransition)
	* 1.1. [1.1 Definition](#Definition)
	* 1.2. [Design decisions](#Designdecisions)
	* 1.3. [Visual description](#Visualdescription)
* 2. [Local vaults](#Localvaults)
	* 2.1. [Vaults as smart contracts](#Vaultsassmartcontracts)
	* 2.2. [Limited responsibility](#Limitedresponsibility)
	* 2.3. [Local transportation to/from wallets](#Localtransportationtofromwallets)
	* 2.4. [Local moves of Staking stock](#LocalmovesofStakingstock)
* 3. [Omnichain staking](#Omnichainstaking)
	* 3.1. [Definition](#Definition-1)
	* 3.2. [Regular asset move plans](#Regularassetmoveplans)
	* 3.3. [Design decisions](#Designdecisions-1)
	* 3.4. [Upgrading staking](#Upgradingstaking)
* 4. [Omnichain LP token](#OmnichainLPtoken)
	* 4.1. [Requirements analysis](#Requirementsanalysis)
	* 4.2. [Design decisions](#Designdecisions-1)
* 5. [Logical components layout](#Logicalcomponentslayout)
* 6. [Omnichain vault](#Omnichainvault)
* 7. [Staking planner](#Stakingplanner)
	* 7.1. [Task definition](#Taskdefinition)
	* 7.2. [Definition](#Definition-1)
	* 7.3. [Formula](#Formula)
* 8. [Inter-chain transportation](#Inter-chaintransportation)
	* 8.1. [Considerations](#Considerations)
	* 8.2. [ Decentralized operations required](#Decentralizedoperationsrequired)
	* 8.3. [Employ the LayerZero service](#EmploytheLayerZeroservice)
	* 8.4. [Operations exemptible of decentralization](#Operationsexemptibleofdecentralization)
	* 8.5. [Design recommendations](#Designrecommendations)
	* 8.6. [An off-chain detour for inter-chain transportation](#Anoff-chaindetourforinter-chaintransportation)
* 9. [Miscellaneous](#Miscellaneous)
	* 9.1. [Compounding](#Compounding)
		* 9.1.1. [Considerations](#Considerations-1)
	* 9.2. [Design recommendations](#Designrecommendations-1)
	* 9.3. [Gas supply](#Gassupply)
		* 9.3.1. [Considerations](#Considerations-1)
		* 9.3.2. [Design recommendations](#Designrecommendations-1)
	* 9.4. [Auxiliary descriptions of the architecture](#Auxiliarydescriptionsofthearchitecture)
		* 9.4.1. [Deposit / Withdraw - deposits and rewards mixed 1:1](#DepositWithdraw-depositsandrewardsmixed1:1)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

<div style="page-break-after: always;"></div>
<br><br>

# <p style="text-align: center;">Architectural Decisions</p>

- Definition
    - Mozaic, the system, and Mozaic system refers to the software system that this project is going to develop, launch, and operate.
- Goal
    - This document describes architectural decisions that implement the system requirements.
    - This document serves as additional technical terminology of the project.
<br>


##  1. <a name='Overallstatetransition'></a>Overall state transition
<br>

###  1.1. <a name='Definition'></a>1.1 Definition
<br>

- **Asset state** is the state of what amounts of what tokens, including LP tokens, on what chains are, at the given moment of time, 

    - deposited and pending staking
    - staked
    - rewarded and pending harvesting
    - staying in system treasuries
    - pending withdrawal
    - pending LP payment

- **Request state** is the state of what deposit/withdrawal requests are accepted by the system and waiting for final processing, at the given moment of time.

- If the system does **Optimize asset/request state**, it changes the states to earn optimal staking reward.
<br><br>

###  1.2. <a name='Designdecisions'></a>Design decisions

We choose the Sleeping-then_Optimizing model for the overall system behavior. 
- The system will not always be optimizing, but sleeping most of the time accepting deposit/withdrawal requests from users.
- If the system **accept**s a deposit request, it 
    - collects the asset the user wants to deposit, (on the asset's chain)
    - tells the user to wait until the next optimization round, when the system will send LP tokens to the user in return for the asset,
    - and book the request with the system for later processing.
- If the system **accept**s a withdraw request, it
    - collects the LP tokens the user wants to return, (on the LP's chain)
    - tells the user to wait until the next optimization round, when the system will send some asset of the requested token type,
    - and book the request with the system for later processing.
- The system will **optimize asset/request state** at regular or irregular intervals. The frequency of optimization rounds will be optimized, as frequent moves of asset may incur more costs while infrequent optimization rounds will hinder from quick maneuver of staking.
- When entering the **Optimization process**, or the **Optimizing** state, the system will take a snapshot of the asset/request state. Then the system transforms/changes the states to optimal states for maximum reward, while continuing to **accept** deposit/withdrawal requests, which will be handled at the next round of optimization.
<br><br>

###  1.3. <a name='Visualdescription'></a>Visual description
<br>
The Sleeping-then_Optimizing model of behavior can be expressed in a UML State Machine diagram shown below:
<br><br>

<p align="center">
  <img src=".\High-leve state machine 1.0.PNG" width="1280" title="high-level use cases" style="page-break-after: avoid;">
</p>

<div style="page-break-after: auto;"></div>
<br><br>

##  2. <a name='Localvaults'></a>Local vaults
<br>
We need a module that implements the use case **Control asset move** identified in the requirements specification, solely, and completely. We call the module the vault, for the following reasons.

<br>

###  2.1. <a name='Vaultsassmartcontracts'></a>Vaults as smart contracts
According to the requirements, vaults are responsible to, exclusively and at the decentralization level,
- keep track of all changes to asset/request state, defined in the previous section
- execute all changes to asset/request state,
- log all changes to asset/request state.
<br>

The only way is to deploy smart contracts on chains that cooperate with each other to form an abstract vault module. We call them vaults or vault contracts.
<br><br>

###  2.2. <a name='Limitedresponsibility'></a>Limited responsibility

According to the requirements, vaults do *not* have to 
- find an optimal asset state to **Optimize asset/request state** to
- execute asset move requests, like transitionPlan identified in requirements, strictly, because there will not be negative profit.
<br><br>

###  2.3. <a name='Localtransportationtofromwallets'></a>Local transportation to/from wallets
- Pull asset from the user if a deposit involves a local token of asset to collect
- Push LP to the user if a deposit involves the local LP token to return
- Pull LP tokens from the user if a withdrawal involves the local LP token to collect
- Push asset to the user if a withdrawal involves a local token of asset to return
- Swap at local Dex pools
<br><br>

###  2.4. <a name='LocalmovesofStakingstock'></a>Local moves of Staking stock
- Stake assets to local staking pools
- Un-stake whole or partial assets from local staking pools
- Collect rewards from local staking pools
- Swap at local Dex pools
<br><br>

<div style="page-break-after: auto;"></div>

##  3. <a name='Omnichainstaking'></a>Omnichain staking
page-break

###  3.1. <a name='Definition-1'></a>Definition
<br>
Omnichain staking requires that:

- Assets can be deposited in any listed token format on any listed chain, *at users' request*.
- Deposited assets can be swapped/transferred, and staked in any staking pool on any listed chain, *guided by the system's optimization plan*.
- Staked assets and rewards can be withdrawn in any listed token format on any listed chian, *at users' request*.
- Rewards collected can be swapped/transferred, and staked in any staking pool on any listed chain, *guided by the system's optimization plan.*
<br><br>

**For a given chain, we define the followings:**
<br>

$ChainAssetPlaces = \{ChainVaultWallet\} \cup ChainStakingPools \cup ChainWithdrawalWallets$

- $ChainVaultWallet$: the vault's wallet on the given chain. They have 
    - deposited assets that are pending staking
- $ChainStakingPools$: all staking pools on the given chain. They have
    - staked assets
    - pending rewards
- $ChainWithdrawalWallets$: wallets of all users whose withdrawal requests are booked and who will receive withdrawal assets on the given chain. They have
    - negative undefined asset to send to the users - the system has to calculate the amount based on the LP token amount redeemed from the user.
<br><br>

$AllVaultWallets$: the set of $ChainVaultWallet$ of all listed chains.

$AllStakingPools$: the sum of $ChainStakingPools$ of all listed chains.

$AllWithdrawalWallets$: the sum of $ChainWithdrawalWallets$ of all listed chains.
<br><br>
$AllAssetPlaces = AllVaultWallets \cup AllStakingPools \cup AllWithdrawalWallets$
<br><br>
We know an asset move is the move of asset, whether it changes the token type or not, between any two of $AllAssetPlaces$.
<br><br>

We classify asset moves into the following two groups:
- **Inter-Chain Moves** of a given chain: asset moves that take place between any two of $Chian Asset Place$ of the given chain
- **Intra-Chain Moves**: asset moves that take place between one of a chain's $Chain Asset Places$ and another of another chain's $Chain Asset Places$
<br>

<div style="page-break-after: auto;"></div>

###  3.2. <a name='Regularassetmoveplans'></a>Regular asset move plans
<br>
An asset move plan is a set of elementary asset move instructions. We need to eliminate redundant value flows from asset move plans to save the cost of executing the plan.
<br> 

A regular asset move plan as a plan that has no redundant value flows. **For any asset move plan, there exists a regular equivalent of the original plan. It should be unique and easy to find.**

Below comes two example of asset move plan: an irregular plan and its regular equivalet:

<p align="center">
  <img src=".\Irregular asset moves.PNG" width="1280" title="high-level use cases" style="page-break-after: avoid;">
</p>

<p align="center">
  <img src=".\Regular asset moves.PNG" width="1280" title="high-level use cases" style="page-break-before: avoid;">
</p>

<div style="page-break-after: auto;"></div>

###  3.3. <a name='Designdecisions-1'></a>Design decisions
<br>

We deduce the following design decisions:
- Asst moves will be regularized. We believe regularization will reduce the total cost of asset moves and the number of inter-chain swap/transfers.
- We believe that ChainWallet will be the best place for regular **Inter-Chain Moves**. It means **Inter-Chain Moves** will be between ChainWallets.
- We need one special vault that oversees the cooperation between vaults, including itself, at decentralization level.
    - **Master vault**: the special vault
    - **Master chain**: the chain that hosts the master vault
<br><br>

###  3.4. <a name='Upgradingstaking'></a>Upgrading staking
<br>
This algorithm will be implemented off-chain, because

<br>
- it will save gas fees
- the requirements don't require decentralization-level of asset move planning
<br>

**This algorithm handles assets**:
- in their USDC-equivalent on the chain, rather than in their own token, when working intra-chain
- in their equivalent of master chains' USDC or the largest giving chains's USDC
- we hope USDC on all chains will have exactly the same price
<br>

**Algorithm**
<br>

- Input: Target staking portfolio. (See the coming sections for optimal staking portfolio)
- Classify asset places into giving places and taking places
    - If the current asset amount is significantly greater than the target asset amount, it is a giving asset place
    - If the current asset amount is significantly less than the target asset amount, it is a giving asset place
    - Else, it is a neutral asset place
    - An asset place has
        - a positive giving amount if it is a giving asset place, else zero
        - a positive taking amount if it is a taking asset place, else zero
        - a zero giving amount and a zero taking amount if it is a neutral asset place.
- Classify chains into giving chains and taking chains
    - If the sum of giving amount of all ChainAsstPlaces is significantly greater than the sum of taking amount, it is a giving chain
    - If the sum of giving amount of all ChainAsstPlaces is significantly less than the sum of taking amount, it is a giving chain
    - Else, it is a neutral chian
    - A chain has
        - a positive giving amount if it is a giving chain, else zero
        - a positive giving amount if it is a taking chain, else zero
        - a zero giving amount and a zero taking amount if it is a neutral chain
- Generate a regular inter-chain asset move plan, AesseMovePlan, for giving chains
    - Collect the local swap prices and fees
    - Sum up surplus asset, which is the target asset less the current asset, of giving asset places to a variable
    - Make a distribution plan that divides the sum of surplus asset to taking asset places
    - Do not care of dust deficit amount. (Leave it to vaults' tunning operation)
    - Now, an irregular asset move plan has been created
    - Regularize the plan, by using graph theories and ad-hoc techniques
    - Conclude with the surplus assets amount of this giving chain
    - **Be aware that the asset move plan is actually asset send plan that doesn't care of price slippage and fees**
    - **Be aware that this plan does/can not take into account the actual price slippage and fees**
- Transfer surplus assets on giving chains to taking chains
    - Sum up surplus assets of giving chains to a variable
    - Make a distribution plan that divides the sum of surplus assets to taking chains
    - Do not care of dust deficit amount. (Leave it to vaults' tunning operation)
    - Now, an irregular asset move plan has been created
    - Regularize the plan, by using graph theories and ad-hoc techniques
    - **Be aware that the asset move plan is actually asset send plan that doesn't care of price slippage and fees**
    - **Be aware that this plan does/can not take into account the actual price slippage and fees**
- Generate a regular inter-chain asset move plan, AeetMovePlan, for taking chains
    (the same as for giving chains)



<div style="page-break-after: auto;"></div>
<br><br>

##  4. <a name='OmnichainLPtoken'></a>Omnichain LP token
<br>

###  4.1. <a name='Requirementsanalysis'></a>Requirements analysis
<br>
Omnichain LP requires that:

- The LP token should exist on all listed chains.
- All LP token versions should always have the save value.
- When the system **stake**s assets that a user **deposit**ed, the system returns LP tokens to the user. The amount of the returned LP token should represent the newly staked asset in the **Staking stock** immediately after the asset is staked.
- A user can **withdraw** assets from the **Staking stock**, in any listed token format on any listed chain, by first returning LP tokens from their wallet to the system wallet.
- The amount of asset that is **withdraw**n is the portion of **Staking stock** that is represented by the returned LP tokens immediately before the asset is **withdraw**n.
<br><br>

Additional optional use cases of LP token may include:

- The LP token can be rebased
- The LP token cannot be rebased and the supply is capped
- The LP token can be staked
- The LP token can be promoted with farming
- LP tokens will open the possibility of becoming an algorithmic stable token by rebasing itself based on oracle price feeds.

<br>

###  4.2. <a name='Designdecisions-1'></a>Design decisions
<br>

LP token contracts should be simple and can/should be highly decentralized.
<br>

- A LP token contract will be deployed on each listed chain
- All LP token contracts will have the same functionality
    - there will not be master-slave or main-secondary relationship
    - this is to make LPs and chains as equal as possible
- LP token contracts will be independent of vaults, except that local vaults should be able to mint and burn local LP tokens 
    - LP token contracts will know be aware of vault contracts
    - LP tokens will be able to continue to work while vaults are in maintenance mode or being upgraded
- LP token contracts will be independent of administration
    - LP tokens will be completely free from administration or DAO 
    - LP tokens will not be upgradeable, for example
- LP tokens will be inter-chain swapped 1:1 at the level of decentralization
    - LZ or other inter-chain transportation service will be used 
    - (**What if the LZ service gets down?** Make it changeable?)
    - Mint-burn, not lock-release, mechanism will be used
- LP token swap will be either completely successful or completely reverted on both the source chain and the destination chain
    - It will employ the same technique as Stargate's swap, **if we find no alternatives**.
- **LP tokens will be promoted with farming**
    - Farming will emit, as reward, portion of the systems profit share
    - This will incentivize more staking
    - Farming will be controlled by administration (Starget's decision models is interesting)
<br>

<div style="page-break-after: always;"></div>
<br><br>

##  5. <a name='Logicalcomponentslayout'></a>Logical components layout
<br>

The overall architectural requiement for vault was/is to **minimize vault as much as possible** leaving most compute to off-chain modules.

The design decisions are as illustrated in the following figure:
<br><br>
<p align="center">
  <img src=".\High-level functional modules 1.0.PNG" width="1280" title="high-level functional modules" style="page-break-after: avoid;">
</p>

Functional modules are described below:
- **Secondary vault contract**: This module is a smart contract and deployed on each chain except the home chain. It participates in the cooperation with its peer **Secondary vault contract**s to implement **Control asset move**, This module has at least the following private operations that work locally on its chain: collect_reward(...) and move_staking_asset(...)
- **Master vault contract**: This module is a smart contract, has global operations, in addition to the local operations inherited from **Secondary vault contract**, and is deployed on one and only one of the chains, called **Master chain**, where it takes the role of **Secondary vault contract**, as well as the the unique role of the master vault operating **LP token contract**.
- **LP token contract**: This module manages the LP token balances of **User**s, which represents their proportional share of the total assets of the system.
- **User wallet**: This is a blockchain wallet and identifies a **User**. **User**'s actions; like deposit, withdraw, and harvest; are authenticated/authorized with this wallet.
- **Vault account**: It is the blockchain account of, and controlled by, **Secondary vault contract** and used to store temporary assets, like funds pending staking.
- **Treasury wallet**: This is a blockchain wallet, and a place to store and retrieve system revenues, like fees. It will be better if it is not owned by a human, but be the account of a smart contract that only obeys vault contracts, for better decentralization. It is deployed on all chains.
- **Staking optimizer**: This is an off-chain module that can invoke **Master vault contract**s. This module is globally unique, calculates optimal **transitionPlan**s, and lets the master vault to execute the plans (in cooperation with secondary vaults). *It is an important design decision that the transitionPlan is calculated off-chain, thus leading to transparency and security debates, for the sake of gas- and time- savings:*
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
- **Pools tracker**: A shared module between **Staking optimizer** and **Trading optimizer**, this module retrieves and tracks all relevant information from chains, like Reward Release Speed, and total Staked LP of each pool. Running this module on-chain would enhance transparency, but would at the same time incur huge gas fees and effectively disable the system.

<br>


<div style="page-break-after: always;"></div>
<br><br>

##  6. <a name='Omnichainvault'></a>Omnichain vault
<br>

We identify vaults through their surrounding modules interacting with them.
The external actors in the following use case diagram, together with their interactions with vaults are already described above. We can now explore the use cases of vault.

<br>

<p align="center">
  <img src=".\Vault use cases 1.0.PNG" width="1280" title="vault use cases" style="page-break-after: avoid;">
</p>
<br>

- **_Deposit**: This is what happens at the level of vault contracts when the **Deposit** use case is invoked at the system level. Invoked by the **User wallet** with fund amount to deposit, this use case coordinates the following two use cases.
- **Book deposit**: This use case 
    - collects the fund from **User wallet**,
    - books the deposit request with the system,
    - and pauses the session of "_Deposit".
- **Finish deposit**: This use case 
    - resume the session of "_Deposit",
    - retrieves the booked deposit request, 
    - mints LP tokens to cover the new fund, 
    - returns the LP tokens to **User wallet**,
    - and helps **Control staking transition** stake the fund.
- **_Withdraw**: This is what happens at the level of vault contracts when the **Withdraw** use case is invoked at the system level. Invoked by **User wallet** with LP tokens returned, this used case coordinates the following two use cases.
- **Book withdraw**: This use case 
    - collects the returned LP tokens,
    - burns the collected LP tokens,
    - books the withdrawal request with the system,
    - and pauses the session of "_Deposit".
- **Finish deposit**: This use case 
    - resume the session of "_Deposit",
    - retrieves the books withdrawal request,
    - subtract fund, as much as covered by the returned LP tokens, from the total system assets, and
    - returns the fund to **User wallet**
- **Control staking transition**: This use case transitions to a new asset state by executing **transitionPlan** provided by **Staking optimizer**. (This is the most challenging part of vault implementation.) It does *collectively*, by using **Move staking asset**,
    - **Collect reward**,
    - Collect staked assets, to cover the fund to **Finish withdraw**,
    - **Finish deposit**,
    - **Finish withdraw**,
    - execute remaining part of **transitionPlan**
- **Control trading**: This use case executes **assetMovePlan** provided by **Trading optimizer**.

<div style="page-break-after: auto;"></div>
<br><br>

##  7. <a name='Stakingplanner'></a>Staking planner
<br>
Note. All errors, like numerical processing rounding and price slippage, are ignored at this stage of architectural design.
<br><br>

###  7.1. <a name='Taskdefinition'></a>Task definition

- Goal: Calculate best staking portfolio, in order to
    - Save vault contracts long calculations of staking optimization, thus to save gas.
    - Keep vault contracts insulated from future algorithm upgrades of staking optimization.
- Consideration
    - Input may not be idealistically consistent inside itself, because an idealistic snapshot of multiple chain states is impossible logically.
    - Output staking portfolio may not be completely/perfectly implemented, because input may have errors and there are unpredictable price slippages sneaked into the calculation.
<br>

- Input
    - Deposit requests currently booked
    - Withdrawal requests currently booked
    - Current pending rewards
    - Current poolsState
<br>

- Output
    - optimal staking portfolio

<br>

###  7.2. <a name='Definition-1'></a>Definition

- **Pools state**

    $PS = \{PS_i | i=1..N\}$,

    where $PS_i$ is the $i$-th pool state: 

    $PS_i = (RewardRate_i, RewardToken_i, TotalStake_i, StakingToken_i, MozaicStake_i, Price_i^R, Price_i^S)$

    , where 
    - $Price_i^R$ is the price of reward token
    - $Price_i^S$ is the pirce of staking token
    
    <br>

    Note: We assume reward on $PS_i$ is calculated as:

    $MozaicReward_i = \frac {\normalsize RewardRate_i}{\normalsize TotakStake_i} \times MozaicStake_i $

    If this assumption restricts the applicable staking pools, we have change the optimization algorithm.
<br>

- **Token vectors**

    - **User tokens vector**

        All listed tokens on all listed chains, known to users.

        $UserTokens = \{UserToken_i | i=1..M\}$

    - **Reward tokens vector**

        $RewardTokens = \{RewardToken_i | i=1..N\}$

    - **Staking tokens vector**

        $StakingTokens = \{StakingToken_i | i=1..N\}$

    - **Tokens vector**

        $Tokens = (UserTokens, RewardTokens, StakingTokens)$
<br>
- **Asset vectors**

    - **Vector of booked deposit requests**, at transition index $t$
    
        $Deposits^t = \{ (D_i^t, \space dLP_i^t) \space | \space D_i^t: deposit \space amount \space denominated \space by \space UserToken_i, \space dLP_i^t: \space LP \space amount, \space all \space at \space transition \space t, . i=1..M \}$

    - **Vector of booked withdrawals requests**, at time index $t$

        $Withdrawals^t = \{ (W_i^t, \space wLP_i^t) \space | \space D_i^t: withdrawal \space amount \space denominated \space by \space UserToken_i, \space wLP_i^t: \space LP \space amount, \space all \space at \space transition \space t,  . i=1..M \}$

    - **Vector of collected rewards**, at transition index $t$

        $Rewards^t = \{R_i^t \space | \space D_i^t: reward \space amount, at \space transition \space t, \space denominated \space by \space RewardToken_i. i=1..M \}$

    - **Vector of staked assets**, at transition index $t$

        $Stakes^t = \{S_i^t \space | \space D_i^t: stake \space amount, at \space transition \space t, \space denominated \space by \space StakingToken_i. i=1..N \}$
<br>
- **Transformations**

    - **Transformation $USDT^{+1}$**

        $USDT^{+1}$ transforms denominated asset vector to USDT-denominated vector, by dividing them with relevant USDT prices.

        Example: $USDT^{+}$ transforms (1, 2, 3) to (1, 2.02, 1300), assuming UserTokens = (USDT, USDC, ETH) and USDT/USDC = 1.01 and USDT/ETH = 1300.

    - **Transformation $USDT^{-1}$**

        $USDT^{-1}$ transforms a USDT-denominated asset vector to Tokens-denominated vector, by multiplying them with relevant USDT prices.

        Example: $USDT^{-1}$ transforms (1, 2.02, 1300) to (1, 2, 3), assuming UserTokens = (USDT, USDC, ETH) and USDT/USDC = 1.01 and USDT/ETH = 1300.

    - **Transformation $FOP$** - the core of the Archimedes algorithm

        $FOP$, standing for Find Optimal Portfolio, finds the **best** USDT-denominated vector of staked assets for a given total USDT amount. In other words, *it finds what amounts of (USDT-denominated) value should be allocated to what staking pools for swap_and_staking, provided that the total (USDT-denominated) value is given*.  **best** is relative and subjective.

    - **Transformation $Sum$**

        $ElementWiseSum$ sums up all elements of a vector.
<br>
- **Mozaic's asset state**, at transition $t$

    - Asset snapshot just before the transition that takes place at transition $t$:

        $MS^t = (Tokens, Deposits^t, - \space Withdrawals^t, Rewards^t, Stakes^t)$

        Or, simply, $MS^t = (T, D^t, - \space W^t, R^t, S^t)$

        Note the "-" sign only applies to the asset amount of each element of the vector.

    - Asset snapshot just after the transition $t$:

        $MS^{t+} = (Tokens, 0Deposits, - \space 0Withdrawals, 0Rewards, optimal \space Stakes^t)$

        Or, simply, $MS^{t+} = (T, 0D, - \space 0W, 0R, optimal \space S^t)$

        , where 0D, 0D, and 0R are a vector of zero valued elements of their respective types.

<br>

###  7.3. <a name='Formula'></a>Formula

If a state transition takes place at time $t$, Mozaic's asset state $MS^t$ changes to $MS^{t+}$ as shown in the following diagram: <br><br>

$$\begin{CD} \space \space  \space \space \space \space MS^t = (T,\space D^t,\space - \space W^t,\space R^t,\space S^t) @> (Resulting \space transition) >> MS^{t+} = (T,\space 0D,\space - \space 0W,\space 0R,\space optimal \space S^t)  \space  \space \space \space \space \space \space \space \space \\ @V USDT^{+1} VV @A USDT^{-1} AA \\  MS_U^t = (T, \space USDT^{+1}(D^t), \space ... ) @>> (Implicit) > MS_U^{t+} = (T,\space 0D,\space - \space 0W,\space 0R,\space FOP(Total \space in \space USDT)) \space \space \space \space \space \space \space \space \space \space\space \\ @V Sum VV @A zeros, \space {FOP} AA \\ Total \space in \space USDT @> Identity >> Total \space in \space USDT \end{CD}$$
<br>

The algorithm for Staking planner $MS^t$ is a chain of transformations:

$optimial \space S^{t} = USDT^{-1} \circ FOP \circ Sum \circ USDT^{+1} (T, \space D^t, \space - \space W^t, \space R^t, \space S^t)$ <br><br>

- $USDT^{+1}$ and $USDT^{-1}$ are obvious, except that we may need systematic methods to find best Dexes and swap paths.
- FOP is solved analytically, demonstrating about 9% of competitive edge over the public.
- ElementWiseSum is trivial.
- $D^t and W^t$ can be retrieved from the booked requests of deposits and withdrawals.
- $S^t$ is found when we "Collect reward" pending rewards.
<br>


**The formula essentially does**:
- collect the quantity numbers of all asset amounts available for the new staking: deposited assets pending staking, less assets to send to withdraw-ers, plus the current staked assets, plus pending rewards,
- transform them to a single USDT value: Total_in_USDT,
- allocate the Total_in_USDT **optimally** across all staking pools, by using the FOP algorithm, Note: **optimally** is relative and subjective.
- transform the allocated USDT amounts back to the native tokens on the staking pools.
<br><br>


<div style="page-break-after: auto;"></div>
<br><br>

##  8. <a name='Inter-chaintransportation'></a>Inter-chain transportation
<br>

###  8.1. <a name='Considerations'></a>Considerations

- Decentralized inter-chain transportation is required for omnichain-ness
- Decentralized inter-chain transportation may lead to bad User Experience, for its inherent long asynchronous operation
- As such, we need to reduce the use of inter-chain transportation as possible
- **Not all inter-chain transportation need to be decentralized** (explained below)
- Layer Zero is the de facto industry standard of decentralized inter-chain transportation service

###  8.2. <a name='Decentralizedoperationsrequired'></a> Decentralized operations required
Some vault operations should be decentralized to meet the requirements. Tracking of asset/LP amount should be executed decentrally, without intermediate off-chain agent or relayer, and with transparent loggs
- Collecting pending rewards from all staking pools to vaults
- Withdrawing from and staking to staking pools to/from vault
- Query for local amounts of assset and LP token
- Cooperation between vaults to calculate and exchange the information of asset/LP amounts and indexes derived therefrom
- Sending assets from users' wallets to vaults, on users' deposit requests
- Calculating LP amount to send to users, in return for their deposited assets
- Sending LP tokens from vaults to users' wallets, on users' deposit requests
- Sending LP from users' wallets to vaults, on users' withdrawal requests
- Calculating asset amount to send to users, in return for their returned LP tokens
- Sending assets from vaults to users' wallets, on users' withdrawal requests
<br><br>

###  8.3. <a name='EmploytheLayerZeroservice'></a>Employ the LayerZero service


<p align="center">
  <img src=".\Mozaic and LayerZero.PNG" width="1280" title="vault use cases" style="page-break-before: avoid;">
</p>
<br>


###  8.4. <a name='Operationsexemptibleofdecentralization'></a>Operations exemptible of decentralization
Vaults cooperation for staking optimization **does not have to be decentralized**, in the meaning that the optimization doesn't have to provide ideal maximum profit nor have to be successful
- Collecting pools information from chains to off-chain modules, could be done by off-chain modules
- Sending asset move plana to chains, could take detour via Mozaic off-chain modules with Admin wallets, at the risk of 
    - the plan could be tempered by (inauditable/unautidited) Mozaic modules or hackers. (But the plan itself is calculated by off-chain modules.)
    - the plan may even fail to be conveyed. (But this type of off-chain failture can also happen when we don't employ off-line detours.)
- Relaying requests between local vaults, during transitioning to a new staking, could take detour via Mozaic off-chain modules with Admin wallets, with the same risks as above
<br>

###  8.5. <a name='Designrecommendations'></a>Design recommendations

- We will choose *decentralized* inter-chain transportation between off-chain modules and vault contracts when finding new optimal staking portfolio and transitioning to the new staking
- Inter-chain messages, once identified as required, will carry as much information as possible.

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

###  8.6. <a name='Anoff-chaindetourforinter-chaintransportation'></a>An off-chain detour for inter-chain transportation
- An off-chain module monitors event logs of a smart contract for a target event happening
- Once detected, the event will be consumed by off-chain modules to produce response
- The produces response will be sent to a proper smart contract

<div style="page-break-after: auto;"></div>
<br>

##  9. <a name='Miscellaneous'></a>Miscellaneous

###  9.1. <a name='Compounding'></a>Compounding

####  9.1.1. <a name='Considerations-1'></a>Considerations
- Rewards should be compounded as frequently as possible, unless the gas fees grows larger than rewards
- Compounding can be executed either:
    - at stocking transition rounds
    - or at its own intervals

###  9.2. <a name='Designrecommendations-1'></a>Design recommendations
- Leave it open

###  9.3. <a name='Gassupply'></a>Gas supply

####  9.3.1. <a name='Considerations-1'></a>Considerations

- Optimization will spend significant amount of gas, although we minimize vaults
- Gas is spent chain-wise, although the profit generation is not necessarily chain-wise

####  9.3.2. <a name='Designrecommendations-1'></a>Design recommendations
- The initial version will be sourcing local gas fees from the local Staking stock. **If the local Staking stock is not sufficient for gas fees, the chain will be set inactive**.
- Future versions will maintain a distributed treasure manager to provide local gas spending.

###  9.4. <a name='Auxiliarydescriptionsofthearchitecture'></a>Auxiliary descriptions of the architecture

####  9.4.1. <a name='DepositWithdraw-depositsandrewardsmixed1:1'></a>Deposit / Withdraw - deposits and rewards mixed 1:1
- We maintain a variable $deposits$ per user.
- Alice's $deposits$ is now 170 stable coins.
- Alice deposits 30 stable coins,
    - Her $deposits$ increases by 30 to become 200.
    - Her total LP token increases by some amount of LP token that is calculated to be the share of 30 in the system's resulting renewed Staking stock.
- When she wants to withdraw with 20 LP tokens
    - We first find she has a total 100 LP tokens
    - Decrease her $deposits$ by 200 * 20 / 100 = 40 stable coins
    - Return the withdrawal assets to her, say 120 stable coins, which is a mix of deposits and rewards
    - We know she withdraws 40 principal deposit, together with 120 - 40 = 80 rewards
- **Now, the performance fees = 80 * 10% = 8 stable coins.**
- This means the withdrawal amount is forced to be a 1:1, which is not numerical but proportional, mix of original/principal deposits and rewards generated by staking/compounding them.
- The system will not serve other mix ratios but 1:1, because a ratio is meaning less as $deposits$ and rewards are all mixed and work together with constant compounding.


