# Lesson 20 - Lottery

## Lottery contract

* (Review) Design patterns
* Architecture overview
* Lottery structure

### Implementation details

* Implement ownable
* Owner deploy lottery and define betting price and fee
* Owner start lottery
  * Define a block timestamp target
* Players must buy an ERC20 with ETH
* Players pay ERC20 to bet
  * Only possible before block timestamp met
* Anyone can roll the lottery
  * Only after block timestamp target is met
  * Randomness from RANDAO
* Winner receives the pooled ERC20 minus fee
* Owner can withdraw fees and restart lottery
* Players can burn ERC20 tokens and redeem ETH

### References

<https://coinsbench.com/how-to-create-a-lottery-smart-contract-with-solidity-4515ff6f849a>

<https://docs.soliditylang.org/en/latest/contracts.html#creating-contracts>

<https://support.quicknode.com/hc/en-us/articles/14985210780305-Block-timestamp-explained>

<https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/integer-division/>

## Coding the contract

* Implementing the randomness source from `prevrandao`
* Implementing the time lock using block timestamp
* Block time estimation
* Implementing the fee
  * (Review) Dealing with decimals
* Withdrawing from pool and redeeming eth
* (Bonus) "Normal" factory pattern and clone pattern overview

### Code references

Contract code for `LotteryToken`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";

contract LotteryToken is ERC20, ERC20Burnable, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor(string memory name, string memory symbol) ERC20(name, symbol) {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }
}
```

Contract code reference for `Lottery`:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;

import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";
import {LotteryToken} from "./LotteryToken.sol";

/// @title A very simple lottery contract
/// @author Matheus Pagani
/// @notice You can use this contract for running a very simple lottery
/// @dev This contract implements a relatively weak randomness source, since there is no cliff period between the randao reveal and the actual usage in this contract
/// @custom:teaching This is a contract meant for teaching only
contract Lottery is Ownable {
    /// @notice Address of the token used as payment for the bets
    LotteryToken public paymentToken;
    /// @notice Amount of tokens given per ETH paid
    uint256 public purchaseRatio;
    /// @notice Amount of tokens required for placing a bet that goes for the prize pool
    uint256 public betPrice;
    /// @notice Amount of tokens required for placing a bet that goes for the owner pool
    uint256 public betFee;

    /// @notice Constructor function
    /// @param tokenName Name of the token used for payment
    /// @param tokenSymbol Symbol of the token used for payment
    /// @param _purchaseRatio Amount of tokens given per ETH paid
    /// @param _betPrice Amount of tokens required for placing a bet that goes for the prize pool
    /// @param _betFee Amount of tokens required for placing a bet that goes for the owner pool
    constructor(
        string memory tokenName,
        string memory tokenSymbol,
        uint256 _purchaseRatio,
        uint256 _betPrice,
        uint256 _betFee
    ) Ownable(msg.sender) {
        paymentToken = new LotteryToken(tokenName, tokenSymbol);
        purchaseRatio = _purchaseRatio;
        betPrice = _betPrice;
        betFee = _betFee;
    }

    /// @notice Opens the lottery for receiving bets
    function openBets() external {
        // TODO
    }

    /// @notice Gives tokens based on the amount of ETH sent and the purchase ratio
    /// @dev This implementation is prone to rounding problems
    function purchaseTokens() external {
        // TODO
    }

    /// @notice Charges the bet price and creates a new bet slot with the sender's address
    function bet() public {
        // TODO
    }

    /// @notice Calls the bet function `times` times
    function betMany(uint256 times) external {
        require(times > 0);
        while (times > 0) {
            bet();
            times--;
        }
        // TODO (Bonus): optimize this
    }

    /// @notice Closes the lottery and calculates the prize, if any
    /// @dev Anyone can call this function at any time after the closing time
    function closeLottery() external {
        // TODO
    }

    /// @notice Returns a random number calculated from the previous block randao
    /// @dev This only works after The Merge
    function getRandomNumber() external {
        // TODO
    }

    /// @notice Withdraws `amount` from that accounts's prize pool
    function prizeWithdraw(uint256 amount) external {
        // TODO
    }

    /// @notice Withdraws `amount` from the owner's pool
    function ownerWithdraw(uint256 amount) external {
        // TODO
    }

    /// @notice Burns `amount` tokens and give the equivalent ETH back to user
    function returnTokens(uint256 amount) external {
        // TODO
    }
}
```

Script code reference:

```typescript
import { viem } from "hardhat";
import { parseEther, formatEther, Address } from "viem";
import * as readline from "readline";

const MAXUINT256 =
  115792089237316195423570985008687907853269984665640564039457584007913129639935n;

let contractAddress: Address;
let tokenAddress: Address;

const BET_PRICE = "1";
const BET_FEE = "0.2";
const TOKEN_RATIO = 1n;

async function main() {
  await initContracts();
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  mainMenu(rl);
}

async function initAccounts() {
  // TODO
}

async function initContracts() {
  // TODO
}

async function mainMenu(rl: readline.Interface) {
  menuOptions(rl);
}

function menuOptions(rl: readline.Interface) {
  rl.question(
    "Select operation: \n Options: \n [0]: Exit \n [1]: Check state \n [2]: Open bets \n [3]: Top up account tokens \n [4]: Bet with account \n [5]: Close bets \n [6]: Check player prize \n [7]: Withdraw \n [8]: Burn tokens \n",
    async (answer: string) => {
      console.log(`Selected: ${answer}\n`);
      const option = Number(answer);
      switch (option) {
        case 0:
          rl.close();
          return;
        case 1:
          await checkState();
          mainMenu(rl);
          break;
        case 2:
          rl.question("Input duration (in seconds)\n", async (duration) => {
            try {
              await openBets(duration);
            } catch (error) {
              console.log("error\n");
              console.log({ error });
            }
            mainMenu(rl);
          });
          break;
        case 3:
          rl.question("What account (index) to use?\n", async (index) => {
            await displayBalance(index);
            rl.question("Buy how many tokens?\n", async (amount) => {
              try {
                await buyTokens(index, amount);
                await displayBalance(index);
                await displayTokenBalance(index);
              } catch (error) {
                console.log("error\n");
                console.log({ error });
              }
              mainMenu(rl);
            });
          });
          break;
        case 4:
          rl.question("What account (index) to use?\n", async (index) => {
            await displayTokenBalance(index);
            rl.question("Bet how many times?\n", async (amount) => {
              try {
                await bet(index, amount);
                await displayTokenBalance(index);
              } catch (error) {
                console.log("error\n");
                console.log({ error });
              }
              mainMenu(rl);
            });
          });
          break;
        case 5:
          try {
            await closeLottery();
          } catch (error) {
            console.log("error\n");
            console.log({ error });
          }
          mainMenu(rl);
          break;
        case 6:
          rl.question("What account (index) to use?\n", async (index) => {
            const prize = await displayPrize(index);
            if (Number(prize) > 0) {
              rl.question(
                "Do you want to claim your prize? [Y/N]\n",
                async (answer) => {
                  if (answer.toLowerCase() === "y") {
                    try {
                      await claimPrize(index, prize);
                    } catch (error) {
                      console.log("error\n");
                      console.log({ error });
                    }
                  }
                  mainMenu(rl);
                }
              );
            } else {
              mainMenu(rl);
            }
          });
          break;
        case 7:
          await displayTokenBalance("0");
          await displayOwnerPool();
          rl.question("Withdraw how many tokens?\n", async (amount) => {
            try {
              await withdrawTokens(amount);
            } catch (error) {
              console.log("error\n");
              console.log({ error });
            }
            mainMenu(rl);
          });
          break;
        case 8:
          rl.question("What account (index) to use?\n", async (index) => {
            await displayTokenBalance(index);
            rl.question("Burn how many tokens?\n", async (amount) => {
              try {
                await burnTokens(index, amount);
                await displayBalance(index);
                await displayTokenBalance(index);
              } catch (error) {
                console.log("error\n");
                console.log({ error });
              }
              mainMenu(rl);
            });
          });
          break;
        default:
          throw new Error("Invalid option");
      }
    }
  );
}

async function checkState() {
  // TODO
}

async function openBets(duration: string) {
  // TODO
}

async function displayBalance(index: string) {
  // TODO
}

async function buyTokens(index: string, amount: string) {
  // TODO
}

async function displayTokenBalance(index: string) {
  // TODO
}

async function bet(index: string, amount: string) {
  // TODO
}

async function closeLottery() {
  // TODO
}

async function displayPrize(index: string) {
  // TODO
  return "TODO";
}

async function claimPrize(index: string, amount: string) {
  // TODO
}

async function displayOwnerPool() {
  // TODO
}

async function withdrawTokens(amount: string) {
  // TODO
}

async function burnTokens(index: string, amount: string) {
  // TODO
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

## Homework

* Create Github Issues with your questions about this lesson
* Read the references

## Weekend Project

This is a group activity for at least 3 students:

* Build a frontend for the Lottery dApp
  * Use any framework of your preference
* Implement the blockchain integration in the frontend
  * Buy/Return tokens
  * View/Place bets
  * Run lottery
  * Check lottery state
  * View/Claim prizes
  * Lottery admin
* Bonus: Organize, document and optimize the smart contract
* Submit your weekend project by filling the form provided in Discord
* Share your code in a github repo in the submission form
