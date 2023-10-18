---
# Title: PasswordStore Audit Report
author: Tinotendajoe
date: October 23, 2023
---

Prepared by: [Tinotenda Joe](https://x.com/tinotendajoe01)

# Table of Contents

- [Disclaimer](#disclaimer)
- [Protocol Summary](#protocol-summary)
- [Audit Details](#audit-details)
  - [Scope](#scope)
  - [Severity Criteria](#severity-criteria)
  - [Summary of Findings](#summary-of-findings)
  - [Tools Used](#tools-used)
- [Critical](#critical)
- [High](#high)
- [Medium](#medium)
- [Low](#low)
- [Informational](#informational)

# Disclaimer

Please note that this audit report is based on the provided information and code from [codehawks](https://github.com/Cyfrin/2023-10-PasswordStore).

# Summary

The PasswordStore aims to provide a secure password storage mechanism on the blockchain. It allows owner to store a private password that others won't be able to see. and lets owner update the password at any time

# Audit Details

## Scope

The audit focused on the security aspects of the PasswordStore contract and identified potential vulnerabilities and areas for improvement.

## Severity Criteria

The severity of the identified issues was categorized into the following levels:

- Critical: Vulnerabilities that can compromise the security of the contract.
- High: Significant issues that can lead to vulnerabilities or unexpected behavior.
- Medium: Areas where the code logic can be improved.
- Low: Considerations for limited functionality and additional validations.
- Informational: Suggestions for gas efficiency optimizations and code best practices

## Summary of Findings

### Critical

- Lack of Access Control
  - This vulnerability in the contract allows anyone to set a new password, compromising the security of the contract.

### High

- Missing Input Validation
  - Lack of input validation on the `newPassword` parameter can lead to potential vulnerabilities and attacks.

### Medium

- Error Handling
  - incomplete error handling

### Low

- Lack of Event Data
  - The "SetNetPassword" event is defined but doesn't emit any data.

### Informational

-Gas Efficiency
-Gas efficiency is not a vulnerability but rather a consideration for optimizing the contract's execution cost. It is recommended to review the contract for any gas-intensive operations and consider potential optimizations.

---

# Critical Findings

## Lack of Access Control

- Vulnerability:

  - The contract currently allows anyone to set a new password, compromising the security of the contract. This means that any address can change the stored password, which contradicts the goal of a private password storage.

- Mitigation:

  - To mitigate this vulnerability, we should implement proper access control to ensure that only the owner of the contract can update the password. We can achieve this by either:

    1.  Creating a custom `onlyOwner` modifier and checking the `msg.sender` against the owner's address.

```solidity
error ONLY_OWNER_CAN_SET_NEW_PASSWORD();

modifier onlyOwner() {
    if (msg.sender == s_owner) revert ONLY_OWNER_CAN_SET_NEW_PASSWORD();
    _;
}

function setPassword(string memory newPassword) external onlyOwner {
    // Rest of the function code
}
```

2.  Inheriting the `Ownable` contract from OpenZeppelin to provide built-in access control functionality.

```solidity
import "@openzeppelin/contracts/access/Ownable.sol";

contract PasswordStore is Ownable {
    // Rest of the contract code

    function setPassword(string memory newPassword) external onlyOwner {
        // Rest of the function code
    }
}
```

---

# High

## Missing Input Validation For `'newPassword'`

- Vulnerability: Gas Exhaustion:

  - Lack of input validation on the `newPassword` parameter can lead to potential vulnerabilities or attacks. Without proper validation, malicious or unintended input may be accepted, potentially leading to issues. such as
  - An attacker or an unaware user could set an exceptionally long password, which could consume a significant amount of gas when storing or processing the input data. This could lead to `out-of-gas errors` and disrupt the contract's functionality with transaction errors related to function argument encoding. In this case if a user send a transaction to the smart contract's `setPassword`, the arguments they provide must be correctly encoded according to the contract's ABI. If the encoding doesn't match the expected function signature which in this sase for
    setPassword is `290bb453` and argument types `setPassword(string)`, they'll receive an error `base fee exceeds gas limit`.
  - This error can be classified as a transaction data validation error, and it's not necessarily a vulnerability in the contract itself. It's more related to how transactions are formed and sent to the contract. This kind of error usually happens when the transaction data exceeds the gas limit, when data types are incompatible, or when data is improperly formatted.

- Mitigation:
  - To mitigate this vulnerability/bug in our contract, we must implement input validation in the `setPassword` function. Check for criteria such as minimum and maximum password length, special character requirements, and complex password criteria. This ensures that only valid and secure passwords are accepted.

---

# Medium

## Error Handling

- Vulnerability:

  - The contract uses custom error handling, which is a valid approach. However, it is crucial to ensure that all potential error scenarios are properly handled to prevent unexpected behavior.
  - Incomplete or inconsistent error handling can lead to vulnerabilities or unexpected contract states.

- Mitigation:

  - To mitigate this issue, we must review the error handling logic and ensure that appropriate error messages and actions are implemented for all relevant error scenarios.
  - This is noted by the absense of an error handler in the `setPassword` fx not handling for the case where the user is not the owner.
  - We must ensure that the contract behaves predictably in error situations and that users receive clear error messages.

---

# Lows

## Lack of Event Data

- Vulnerability:

  - The "SetNetPassword" event is defined but doesn't emit any data. The lack of event data in this contract affects transparency and the ability to understand state changes.

- Mitigation:

  - To mitigate this issue, we should modify the contract to emit relevant data with events. This enhances transparency and off-chain analysis, making it easier for users to understand what has occurred within the contract.
  - we can modify the event definition and the `setPassword` function as follows:

```solidity
event SetNetPassword(address indexed owner, string newPassword);

function setPassword(string memory newPassword) external onlyOwner {
    s_password = newPassword;
    emit SetNetPassword(s_owner, newPassword);
}


```

# Informational

## Gas Efficiency

### Consideration:

Gas efficiency is not a vulnerability but rather a consideration for optimizing the contract's execution cost. It is important to review the contract for any gas-intensive operations and consider potential optimizations to reduce transaction costs for users.

---

Please note that this report is based on the provided information and code snippets. It is important to conduct a thorough review and testing of the entire protocol to ensure its security and functionality.

```

This section provides a detailed breakdown of each finding, including why it is a vulnerability and how to mitigate it. You can copy and paste this detailed section into your audit report.
```
