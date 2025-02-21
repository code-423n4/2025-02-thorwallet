# THORWallet audit details
- Total Prize Pool: &#36;18,000 in USDC
  - HM awards: up to &#36;10,150 USDC 
  - QA awards: &#36;350 in USDC
  - Judge awards: &#36;2,500 in USDC
  - Validator awards: &#36;1,500 USDC 
  - Scout awards: &#36;500 in USDC
  - Mitigation Review: &#36;3,000 USDC
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts February 20, 2025 20:00 UTC 
- Ends February 26, 2025 20:00 UTC 

**Note re: risk level upgrades/downgrades**

Two important notes about judging phase risk adjustments: 
- High- or Medium-risk submissions downgraded to Low-risk (QA) will be ineligible for awards.
- Upgrading a Low-risk finding from a QA report to a Medium- or High-risk finding is not supported.

As such, wardens are encouraged to select the appropriate risk level carefully during the submission phase.

## Automated Findings / Publicly Known Issues

The 4naly3er report can be found [here](https://github.com/code-423n4/2025-02-thorwallet/blob/main/4naly3er-report.md).



_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._

When getting a quote to merge TGT to TITN, the quote might lose precision and leave some decimals out. This is ok as they are very small amount.

The TITN token will be deployed on Arbitrum and Base, on Arbitrum. In the contract, the Arbitrum chain ID is hardcoded. This is ok for our use case.


# Overview

The TITN ecosystem enables users to exchange their `ARB.TGT` for `ARB.TITN`, and subsequently bridge their `ARB.TITN` to `BASE.TITN`.

**Key Features**:

1. Token Transfers on BASE:

- Non-bridged TITN Tokens: Holders can transfer their TITN tokens freely to any address as long as the tokens have not been bridged from ARBITRUM.
- Bridged TITN Tokens: Transfers are restricted to a predefined address (`transferAllowedContract`), set by the admin. Initially, this address will be the staking contract to prevent trading until the `isBridgedTokensTransferLocked` flag is disabled by the admin.

2. Token Transfers on ARBITRUM:

- TITN holders are restricted to transferring their tokens only to the LayerZero endpoint address for bridging to BASE.
- Admin/owner retains the ability to transfer tokens to any address.

**Deployment Details:**

- BASE Network:

  - 1 Billion TITN tokens will be minted upon deployment and allocated to the owner.

- ARBITRUM Network:
  - No TITN tokens are minted initially.
  - The owner is responsible for bridging 173.7 Million BASE.TITN to ARBITRUM and depositing them into the MergeTGT contract.

**Transfer Restrictions**

The contracts include a transfer restriction mechanism controlled by the isBridgedTokensTransferLocked flag. This ensures controlled token movement across networks until the admin deems it appropriate to enable unrestricted transfers.

## Deploy contracts

- `npx hardhat lz:deploy` > select both base and arbitrum > then type `Titn`
- `npx hardhat lz:oapp:wire --oapp-config layerzero.config.ts`
- `npx hardhat lz:deploy` > select only arbitrum > then type `MergeTgt`

## Post Deployment steps

### Setup on BASE

1. Bridge 173700000 TITN to Arbitrum: `npx hardhat run scripts/sendToArb.ts --network base`

### Setup on ARBITRUM

1. Approve, deposit, enable merge...: `npx hardhat run scripts/arbitrumSetup.ts --network arbitrumOne`

## User steps

These are the steps a user would take to merge and bridge tokens (from ARB.TGT to ARB.TITN and then to BASE.TITN)

### Merge steps

1. on MergeTGT call the read function quoteTitn() to see how much TITN one can get
2. `await tgt.approve(MERGE_TGT_ADDRESS, amountToDeposit)`
3. `await tgt.transferAndCall(MERGE_TGT_ADDRESS, amountToDeposit, 0x)`
4. `await mergeTgt.claimTitn(claimableAmount)`

### Bridge to Base

1. run `BRIDGE_AMOUNT=10 TO_ADDRESS=0x5166ef11e5dF6D4Ca213778fFf4756937e469663 npx hardhat run scripts/quote.ts --network arbitrumOne`
2. with those params call the `send()` function in the ARB.TITN contract

## LayerZero Docs

- https://github.com/LayerZero-Labs/devtools/tree/main/examples/oft
- https://docs.layerzero.network/



## Deployed contracts

### Test

- BASE.TITN: `0xf72EC6551A98fE12B53f7c767AABF1aD57bB6DA1` [explorer](https://basescan.org/token/0xf72EC6551A98fE12B53f7c767AABF1aD57bB6DA1#code)
- ARB.TITN: `0x2923b8ea6530FB0c9516f50Cd334e18d122ADAd3` [explorer](https://arbiscan.io/token/0x2923b8ea6530FB0c9516f50Cd334e18d122ADAd3#code)
- ARB.MergeTGT: `0x22EAafe4004225c670C8A8007887DC0a9433bd86` [explorer](https://arbiscan.io/address/0x22EAafe4004225c670C8A8007887DC0a9433bd86#code)
- ARB.TGT: `0x429fed88f10285e61b12bdf00848315fbdfcc341` [explorer](https://arbiscan.io/address/0x429fed88f10285e61b12bdf00848315fbdfcc341#code) 

### Production

- BASE.TITN: 
- ARB.TITN:
- ARB.MergeTGT:
- ARB.TGT: `0x429fed88f10285e61b12bdf00848315fbdfcc341` [explorer](https://arbiscan.io/address/0x429fed88f10285e61b12bdf00848315fbdfcc341#code) 


## Links

- **Previous audits:**  N/A
- **Documentation:** https://github.com/THORWallet/TGT-TITN-merge-contracts/blob/main/README.md
- **Website:** https://www.thorwallet.org/
- **X/Twitter:** https://x.com/thorwallet
- **Discord:** https://discord.com/invite/TArAZHDjCr

---


# Scope

*See [scope.txt](https://github.com/code-423n4/2025-02-thorwallet/blob/main/scope.txt)*

### Files in scope


| File   | Logic Contracts | Interfaces | nSLOC | Purpose | Libraries used |
| ------ | --------------- | ---------- | ----- | -----   | ------------ |
| /contracts/MergeTgt.sol | 1| **** | 132 | |@openzeppelin/contracts/access/Ownable.sol<br>@openzeppelin/contracts/utils/ReentrancyGuard.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/Titn.sol | 1| **** | 66 | |@openzeppelin/contracts/access/Ownable.sol<br>@layerzerolabs/oft-evm/contracts/OFT.sol|
| /contracts/interfaces/IERC677Receiver.sol | ****| 1 | 3 | ||
| /contracts/interfaces/IMerge.sol | ****| 1 | 15 | ||
| **Totals** | **2** | **2** | **216** | | |

### Files out of scope

*See [out_of_scope.txt](https://github.com/code-423n4/2025-02-thorwallet/blob/main/out_of_scope.txt)*

| File         |
| ------------ |
| ./contracts/mocks/Tgt.sol |
| ./test/mocks/ERC20Mock.sol |
| ./test/mocks/OFTComposerMock.sol |
| ./test/mocks/OFTMock.sol |
| Totals: 4 |


## Scoping Q &amp; A

### General questions


| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       [TGT on ARB](https://arbiscan.io/token/0x429fed88f10285e61b12bdf00848315fbdfcc341)             |
| Test coverage                           | 85.19% of statements     |
| ERC721 used  by the protocol            |            None              |
| ERC777 used by the protocol             |           None                |
| ERC1155 used by the protocol            |              None            |
| Chains the protocol will be deployed on | Arbitrum,Base |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- | ------ |
| Enabling/disabling fees (e.g. Blur disables/enables fees) | No   |
| Pausability (e.g. Uniswap pool gets paused)               |  Yes   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   No  |


### EIP compliance checklist
N/A



# Additional context

## Main invariants

Unless enabled (or the user is the admin), users who merge their TGT to TITN should not be able to transfer them to any address other than the LayerZero endpoint or a specified contract address `transferAllowedContract`.


## Attack ideas (where to focus for bugs)
* Will everyone who decides to deposit TGT and leave them in the contract for 12 months be able to get their share of TITN plus any remaining TITN left proportional to their deposit?
* The TITN token on Arbitrum should not allow transfer of tokens to any address other than the LayerZero endpoint (there are exceptions for the owner and a flag that can be set by the owner to enable and disable this feature).


## All trusted roles in the protocol

The owner


## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

After 3 months of the merge contract going live, the quote to merge TGT to TINT will reduce linearly until it reaches 0 (9 months later). 


## Running tests




```bash
git clone https://github.com/code-423n4/2025-02-thorwallet
cd 2025-02-thorwallet
pnpm install
npx hardhat compile
npx hardhat test

# To get test coverage
npx hardhat coverage
```

#### Coverage table

File                   |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-----------------------|----------|----------|----------|----------|----------------|
 contracts/            |    85.19 |       55 |    68.18 |     81.4 |                |
  MergeTgt.sol         |    84.62 |    52.08 |    61.54 |    78.46 |... 118,179,183 |
  Titn.sol             |    86.67 |    66.67 |    77.78 |    90.48 |          39,49 |
 contracts/interfaces/ |      100 |      100 |      100 |      100 |                |
  IERC677Receiver.sol  |      100 |      100 |      100 |      100 |                |
  IMerge.sol           |      100 |      100 |      100 |      100 |                |
 contracts/mocks/      |      100 |       50 |      100 |      100 |                |
  Tgt.sol              |      100 |       50 |      100 |      100 |                |
All files              |    86.89 |    54.84 |       72 |    82.98 |                |



## Miscellaneous
Employees of THORWallet and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.
