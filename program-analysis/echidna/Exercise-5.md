# Exercise 5

**Table of contents:**

- [Exercise 5](#exercise-5)
  - [Setup](#setup)
  - [Exercise](#exercise)
  - [Solution](#solution)

Join the team on Slack at: https://empireslacking.herokuapp.com/ #ethereum

## Setup

1. Follow the instructions on the [Damn Vulnerable DeFi CTF][ctf] page, namely:
    - clone the repo, and
    - install the dependencies via `npm install`.
2. To run Echidna on these contracts you must comment out the `dependencyCompiler` section in `hardhat.config.js`. Otherwise, the project will not compile with [`crytic-compile`](https://github.com/crytic/crytic-compile). See the example provided [here](./exercises/exercise5/example.hardhat.config.ts).
3. Create a contract called `UnstoppableEchidna` in the `contract/unstoppable` directory.
4. Analyze the `before` function in `test/unstoppable/unstoppable.challenge.js` to identify what initial setup needs to be done.

Hint: You don't need to make the setup very complex. It is possible to find the bug with just three contracts:
  - `DamnValuableToken`
  - `UnstoppableLender`
  - `ReceiverUnstoppable`

## Goals

- Setup the testing environment with the right contracts and necessary balances.
- Add a property to check whether `UnstoppableLender` can always provide flash loans.
- Create `config.yaml` with the necessary configuration option(s). Use the [`template.yaml`](./exercises/exercise5/template.yaml) file as a starting point. 
Consider if blacklisting specific functions that will definitely revert when called by an EOA will aid in Echidna's exploration.
- Once Echidna finds the bug, fix the issue, and re-try your property with Echidna.

Hint: You might have to use the `--multi-abi` flag in this exercise.
## Solution

This solution can be found in [exercises/exercise5/solution.sol](./exercises/exercise5/solution.sol)

[ctf]: https://www.damnvulnerabledefi.xyz/

<details>
<summary>Solution Explained (spoilers ahead)</summary>

The goal of the unstoppable challenge is to realize that `UnstoppableLender` has two modes of tracking its balance: `poolBalance` and `damnValuableToken.balanceOf(address(this))`.

`poolBalance` is added to when someone calls `depositTokens()`.

However, a user can call `damnValuableToken.transfer()` directly and increase the `balanceOf(address(this))` without increasing `poolBalance`.

Now, the two balance trackers are out-of-sync.

When Echidna calls `pool.flashLoan(10)`, the assertion `assert(poolBalance == balanceBefore)` in `UnstoppableLender` will break and the pool can no longer provide flash loans.

See example output below from Echidna:

```bash
$ npx hardhat clean && npx hardhat compile --force && echidna-test . --contract UnstoppableEchidna --multi-abi --config contracts/unstoppable/config.yaml

...

echidna_testFlashLoan: failed!💥  
  Call sequence:
    transfer(0x62d69f6867a0a084c6d313943dc22023bc263691,1296000)

...
```
</details>