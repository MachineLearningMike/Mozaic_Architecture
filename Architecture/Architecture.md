<br><br><br><br><br><br><br><br>

# <p style="text-align: center;">Architectural Decisions</p>


<div style="page-break-after: always;"></div>
<br><br>

**Table of Contents**
<br>

<!-- vscode-markdown-toc -->
* 1. [Overall state transition](#Overallstatetransition)
	* 1.1. [1.1 Definition](#Definition)
	* 1.2. [Design decisions](#Designdecisions)
	* 1.3. [Visual description](#Visualdescription)
* 2. [Local vaults](#Localvaults)
	* 2.1. [Vaults as smart contracts](#Vaultsassmartcontracts)
	* 2.2. [Limited responsibility](#Limitedresponsibility)
	* 2.3. [Local transportation to/from wallets](#Localtransportationtofromwallets)
	* 2.4. [Local moves of Staking Stock](#LocalmovesofStakingStock)
* 3. [Omnichain staking](#Omnichainstaking)
	* 3.1. [Definition](#Definition-1)
	* 3.2. [Regular asset move plans](#Regularassetmoveplans)
	* 3.3. [Design decisions](#Designdecisions-1)
	* 3.4. [Transitioning staking](#Transitioningstaking)
* 4. [Omnichain mLP token](#OmnichainmLPtoken)
	* 4.1. [Requirements analysis](#Requirementsanalysis)
	* 4.2. [Design decisions](#Designdecisions-1)
* 5. [Logical components](#Logicalcomponents)
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
* 10. [Reference source code](#Referencesourcecode)

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

<br>


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

<br>

The **Toggle-Between-Optimize-and_Stay** model of behavior can be expressed in a UML State Machine diagram shown below:
<br><br>

<p align="center">
  <img src=".\High-leve state machine 1.0.PNG" width="1280" title="high-level use cases" style="page-break-after: avoid;">
</p>

<div style="page-break-after: auto;"></div>
<br><br>

##  2. <a name='Localvaults'></a>Local vaults
<br>

We need a module that implements the use case **Control asset move** identified in the requirements specification, solely and completely. We call the module the vault, because from users' perspective:
- the module keeps users' assets, control and logs moves of the assets and profits generated from the assets, and returns the assets with profits.
- no modules other than that module have privilege to carry out these tasks.

<br>

###  2.1. <a name='Vaultsassmartcontracts'></a>Vaults as smart contracts
According to the requirements, vaults are responsible to, exclusively and at the decentralization level,
- make
- log


The only way is to deploy smart contracts on chains that cooperate with each other to form an omnichain vault module. We call them local vaults or vault contracts individaully, while they collectively form an omnichain vault.
<br><br>

###  2.2. <a name='Limitedresponsibility'></a>Limited responsibility

According to the requirements, vaults do *not* have to 
- find the *best possible* asset state to **optimize asset/request state** to
- execute strictly asset move requests coming from outside, like transitionPlan identified in requirements, because there will not be negative profit.
<br><br>

###  2.3. <a name='Localtransportationtofromwallets'></a>Local transportation to/from wallets
<br>

Local vaults are responsible to:

- Pull assets from the user if a deposit request involves a home token
- Export a deposit request if it involves an away mLP token
- Import an exported deposit request if it involves a home mLP token
- Push mLP tokens to the user if a deposit request involves the home mLP 

- Pull mLP to the user if a withdrawal request involves the home mLP token
- Export a withdrawal request if it involves an away token
- Import an exported withdrawal request if it involves a home token
- Push asset to the user if a withdrawal request involves a home token
- Swap at local Dex pools

<br><br>

###  2.4. <a name='LocalmovesofStakingStock'></a>Local moves of Staking Stock
<br>
Local vaults are responsible to:

- Stake assets to local staking pools
- Un-stake assets from local staking pools
- Collect rewards from local staking pools
- Swap at local Dex pools
<br><br>

<div style="page-break-after: auto;"></div>

##  3. <a name='Omnichainstaking'></a>Omnichain staking
<br>

###  3.1. <a name='Definition-1'></a>Definition
<br>
Omnichain staking requires that:

- Assets can be deposited in any listed token format on any listed chain, *all of users' choice*.
- Deposited assets can be swapped/transferred, and staked in any staking pool on any listed chain, *guided by the system's optimization plan*.
- Staked assets and rewards can be withdrawn in any listed token format on any listed chian, *all of users' choice*.
- Rewards collected can be swapped/transferred, and staked in any staking pool on any listed chain, *guided by the system's optimization plan.*
<br><br>

**For a given chain, we define the followings**:
<br>

$ChainAssetPlaces$
 $\space = \space \{ChainVaultWallet\} \space \cup  \space ChainStakingPools \space \cup \space ChainDepositWallets \space \cup \space ChainWithdrawalWallets$

- $ChainVaultWallet$: the vault's wallet on the given chain. They hold from time to time:
    - **Deposits** assets that are pending staking
    - other assets that are transient during optimization
- $ChainStakingPools$: all staking pools on the given chain. They hold:
    - **Stakes** assets,
    - **Rewards** assets, pending collecting
- $ChainDepositWallets$: wallets of all users whose deposit requests are booked and who will receive mLP tokens on the given chain. 
    - They **will** receive an amount of mLP sent to the users - the system has to calculate the amount based on the mLP token amount redeemed from the user.
    - The amount to receive will be denoted as a *negative number*.
    - The undefined number will be denoted as **undefined**.
- $ChainWithdrawalWallets$: wallets of all users whose withdrawal requests are booked and who will receive withdrawal assets on the given chain. 
    - They **will** receive an amount of asset to sent to the users - the system has to calculate the amount based on the mLP token amount redeemed from the user.
    - The amount to receive will be denoted as a *negative number*.
    - The undefined number will be denoted as **undefined**.
<br>

$AllVaultWallets$: the set of $ChainVaultWallet$ of all listed chains.

$AllStakingPools$: the sum of $ChainStakingPools$ of all listed chains.

$AllDepositWallets$: the sum of $ChainDepositWallets$ of all listed chains.

<br>

$AllAssetPlaces = AllVaultWallets \cup AllStakingPools \cup AllDepositWallets \cup AllWithdrawalWallets$
<br>

**We know**:
- an asset move is the move of assets, whether it changes the token type or not, between any the same or different two of $AllAssetPlaces$.
- all asset places have an amount of assets, whether the amount be positive, negative, or **undefined**.
<br>

We classify asset moves into the following two groups:
- **Inter-Chain Moves** of a given chain: asset moves that take place between any two of $Chian Asset Placse$ of the given chain
- **Intra-Chain Moves**: asset moves that take place between one of a chain's $Chain Asset Places$ and another of another chain's $Chain Asset Places$
<br>


###  3.2. <a name='Regularassetmoveplans'></a>Regular asset move plans
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

<div style="page-break-after: auto;"></div>

###  3.3. <a name='Designdecisions-1'></a>Design decisions
<br>

**We deduce the following design decisions:**
<br>
- Asset moves will be **regularized**. We believe regularization will reduce the total cost of asset moves and the number of inter-chain swap/transfers.
- **ChainValutWallet** will be the hub for regular **Intra-Chain Moves**. This means an **Intra-Chain Move** will be between the **ChainValutWallet** and another of **ChainAssetPlaces**. We wil *not* allow direct asset moves between **ChainAssetPlaces** that are not **ChainVaultWallet**.
- We need one special vault that oversees the cooperation between vaults, including itself, at decentralization level.
    - **Master vault**: the special vault
    - **Master chain**: the chain that hosts the master vault
- The **Master vault** will be the hub for regular **Inter-Chain Moves**. This means an **Inter-Chain Move** will between the **Master vault** and another of local vaults. We wil *not* allow direct asset moves between **ChainAssetPlaces** of different chains.
<br><br>

###  3.4. <a name='Transitioningstaking'></a>Transitioning staking
<br>
This algorithm will be executed off-chain to produce a transition plan, because

<br>
- it will save huge gas fees that the algorithm would spend if it ran on-chain
- the requirements don't require decentralization-level of asset move planning
<br>

**Note**: The off-chain execution of this algorithm raises the concerns of decentralization.

<br>

### Algorithm
<br>


    - Classify chains into giving chains and taking chains
        - If the sum of giving amount of all asset instances in $ChainAsstPlaces^i$ of a given chain is significantly greater than the sum of taking amount, it is a **giving chain**, and the difference is called the **giving amount** of the giving chain.
            - **ChainDepositWallets^i** collectively forms a special giving chain.
            - Be careful not to harm deposits/withdrawals
        - If the sum of taking amount of all asset instances in $ChainAsstPlaces^i$ of a given chain is significantly greater than the sum of giving amount, it is a **taking chain** and the difference is called the **taking amount** of the taking chain.
            - Be careful not to harm deposits/withdrawals
            - **ChainWithdrawalWallets^i** collectively forms a special taking chain.
        - Else, it is a neutral chian
        - A chain has
            - a positive giving amount if it is a giving chain, else zero
            - a positive giving amount if it is a taking chain, else zero
            - a zero giving amount and a zero taking amount if it is a neutral chain
    - Generate a regular **intra-chain asset move plan** for giving chains
        - Collect the local swap prices and fees
        - Collect giving amounts of all giving asset instances to the vault.
        - Swap, divide, and send the collected amount to fill the taking amounts of taking asset instances.
        - Regularize the plan. (See previous sections)
        - The giving amount of this giving chain should remain in the vault.
    - Execute the regular intra-chain asset move plans for giving chains.
    - Generate a regular **inter-chain asset move plan** that divides and transfers the giving amount of assets of giving chains to taking chains.
        - Collect giving amounts of all giving asset chains to the master vault.
        - Swap, divide, and send the collected amount to fill the taking amounts of taking chains.
        - Regularize the plan.
        - There should not remain transient assets in the master vault, except dusts.
    - Execute the regular inter-chain asset move plan.
    - Generate a regular intra-chain asset move plan for taking chains
        (the same as with giving chains)
    - Execute regular intra-chain asset move plans for taking chains.
<br>

Note: This algorithm should be tweaked to cope with changing price slippages and fees, and numerical dusts, in implementation phases.
<br>

**The algorithm is illustrated below:**
<br>
<p align="center">
  <img src=".\Transition algorithm.PNG" width="1280" title="high-level functional modules" style="page-break-after: avoid;">
</p>



##  5. <a name='Logicalcomponents'></a>Logical components
<br>

The overall architectural requiement for vault was/is to **minimize vault as much as possible** leaving most compute to off-chain modules.

<p align="center">
  <img src=".\High-level functional modules 1.0.PNG" width="1280" title="high-level functional modules" style="page-break-after: avoid;">
</p>



<div style="page-break-after: auto;"></div>
<br><br>

##  8. <a name='Inter-chaintransportation'></a>Inter-chain transportation
<br>

###  8.1. <a name='Considerations'></a>Considerations

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

###  8.2. <a name='Decentralizedoperationsrequired'></a> Decentralized operations required
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

###  8.3. <a name='EmploytheLayerZeroservice'></a>Employ the LayerZero service

Transparent cross-chain transportation is the fundamental basis of omnichain operations. We choose LayerZero service for our cross-chain transportation.

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
- The initial version will be sourcing local gas fees from the local Staking Stock. **If the local Staking Stock is not sufficient for gas fees, the chain will be set inactive**.
- Future versions will maintain a distributed treasure manager to provide local gas spending.

###  9.4. <a name='Auxiliarydescriptionsofthearchitecture'></a>Auxiliary descriptions of the architecture

####  9.4.1. <a name='DepositWithdraw-depositsandrewardsmixed1:1'></a>Deposit / Withdraw - deposits and rewards mixed 1:1
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


##  10. <a name='Referencesourcecode'></a>Reference source code

Below comes reference code that sketches and/or decides the code architecture.

```
pragma solidity ^0.8.0;

// imports
import "../libraries/lzApp/NonblockingLzApp.sol";
import "../libraries/stargate/Router.sol";
import "../libraries/stargate/Pool.sol";
import "./OrderTaker.sol";
import "./MozaicLP.sol";

// libraries
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

abstract contract SecondaryVault is OrderTaker, NonblockingLzApp {
    function _usdt(address _token, uint _amount) internal returns (uint usdtEq) {
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
        uint    usdtEq;
        uint    amountLP;   // undefined initially
    }

    struct DepositToExport {
        address user;
        uint    usdtEq;
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

    Workspace private left;
    Workspace private right;
    bool public transitioning;  // bool has 8 bits. Can put more to fill 256 bits.

    function _getPendingWorkspace() internal view returns (Workspace storage) {
        if (transitioning) {
            return left;
        } else {
            return right;
        }
    }

    function _getStagedWorkspace() internal view returns (Workspace storage) {
        if (transitioning) {
            return right;
        } else {
            return left;
        }
    }

    uint public thisChain;

    // The caller submits _amount of _token, and wants MLP tokens on _chain.
    function addDepositRequest(address _token, uint _amount, uint _chain) external  {
        require(userTokens[_token] != 0 && _amount > 0, "Wrong token/amount");       
        _safeTransferFrom(_token, msg.sender, address(this), _amount);
        
        Workspace storage pending = _getPendingWorkspace();
        if (_chain == thisChain) { // The request is local-token for local-mLP
            pending.ds.push( Deposit(msg.sender, _token, _amount, 0) );
            // 0 for mLP amount to send to the user.
        } else { // The request is local-token for away-mLP
            pending.dsToExport.push(DepositToExport(msg.sender, _usdt(_token, _amount), _chain));
            // Foreign chain _chain will store this like: 
            // pending.dsImported.push(DepositImported(msg.sender, usdtEq, 0));
            // 0 for the undefined mLP amount to send to the user
        }
    }

    // The caller submits _amount of mLP, and wants _token tokens on _chain chain.
    function addWithdrawalRequest(uint _amountLP, address _token, uint _chain) external  {
        require(userTokens[_token] != 0 && _amountLP > 0, "Wrong token/amount");       
        _safeTransferFrom(mLP, msg.sender, address(this), _amountLP);
        
        Workspace storage pending = _getPendingWorkspace();
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

