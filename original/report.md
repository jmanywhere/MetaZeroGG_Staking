# Security Findings:

### 1. Duplicate Declaration

Lines: 25

```
    struct Staker {
        uint256 amountStaked;
        uint256 rewardDebt;
        uint256 rewards;
        uint256 unstakeInitTime;
        uint256 unstakeInitTime; // <--
        bool claimedAfterUnstake;
    }
```

### 2. Need Specific Compiler Version

Lines: 1

`pragma solidity ^0.8.13;`

### 3. IERC20 Does Not Consider All ERC20 Tokens

**_Severity:_** Informational
Description: Functions might not return booleans as per IERC20 specification.

#### Recommendation:

- Short term, do not use this contract for USDT on ETH or similar be aware of the tokens you'll be setting here.
- Long term, implement generic transfers and approvals that do not depend on return values:

`(bool success, bytes memory data) = token.call(abi.encodeWithSelector(IERC20.<FUNCTION>, <LIST OF ARGUMENTS>));`

### 4. View and PURE Functions Placement

Severity: Low
Description: Gas optimization, suggested function placement order.

##### Recommendation:

Reorder functions for gas efficiency.

```
Gas efficient structure:

ERRORS
STATE VARIABLES
EVENTS
MODIFIERS
CONSTRUCTOR
EXTERNAL / PUBLIC Functions
INTERNAL / PRIVATE Functions
EXTERNAL / PUBLIC VIEW / PURE Functions
INTERNAL / PRIVATE VIEW / PURE Functions
```

### 5. Remove or Comment Before Submitting to Audits

**_Severity_**: High

**Description:** Potential owner backdoor.

##### Recommendation:

`Immediate removal.`

### 6. GAS OPTIMIZATION: If Statement + Revert with Custom Error

Lines: 109, 110, 117, 118, etc

### 7. GAS OPTIMIZATION: Calculate rewardRate on Set

Lines: 55, 70
**Description:** Eliminate the need for a recalculation on a value that is only set once.

```
constructor (uint _rewardRate){
    rewardRate = _rewardRate * MAGNIFIER;
}
```

### 8. For Readability 1e18 Should Be Declared a Constant

Lines: 84
**Description:** Makes code harder to read.

#### Recommendation:

Define as a constant for clarity.

```
...
    uint256 private constant MAGNIFIER = 1e18;
...
    return
        ((
            staker.amountStaked *
            (rewardPerToken() - staker.rewardDebt)
        ) / MAGNIFIER) +
        staker.rewards;
```

### 9. Unnecessary Reentrancy Block

Line: 107

**Description:** Reentrancy is only needed if there are any external calls in the function.

### 10. Recommend Checking for `staker.amountStaked == 0` as Well

```
if (
    staker.claimedAfterUnstake == true ||
    staker.amountStaked == 0
    ) {
        return 0;
    }
```

### 11. Always Do Check, Effects, Interactions Pattern Wherever Possible

Line: 147

**Description**: Although `nonReentrant` modifier is called, keep the Checks effects interaction (CEI) consistent to have double protection.

#### Recommendation

```
function completeUnstake() {
        //CHECKS
        Staker storage staker = stakers[msg.sender];
        require(staker.amountStaked > 0, "No tokens staked");
        require(
            block.timestamp >= staker.unstakeInitTime + unstakeTimeLock,
            "Timelock not yet passed"
        );

        uint256 amount = staker.amountStaked;
        uint256 reward = staker.rewards;
        uint256 fee = (amount * unstakeFeePercent) / 10000;
        uint256 amountAfterFee = amount - fee;
        // EFFECTS
        delete stakers[msg.sender];
        //INTERACTIONS
        ...
        <Rest of LOGIC>
        ...
        emit Unstaked(msg.sender, amount);
    }
```
