---
title: Protocol Audit Report
author: Tinotendajoe
date: October 23, 2023
header-includes:
  - \usepackage{titling}
  - \usepackage{graphicx}
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
- [High](#high)
- [Medium](#medium)
- [Low](#low)
- [Informational](#informational)
- [Gas](#gas)

# Disclaimer

Please note that this audit report is based on the provided information and code snippets. It is important to conduct a thorough review and testing of the entire protocol to ensure its security and functionality.

# Protocol Summary

The protocol aims to provide a secure password storage mechanism on the blockchain. It includes features such as access control, password hashing, error handling, and input validation.

# Audit Details

## Scope

The audit focused on the security aspects of the PasswordStore contract and identified potential vulnerabilities and areas for improvement.

## Severity Criteria

The severity of the identified issues was categorized into the following levels:

- Critical: Vulnerabilities that can compromise the security of the contract.
- High: Issues that can potentially expose sensitive data or lead to security risks.
- Medium: Areas where error handling and input validation can be improved.
- Low: Considerations for limited functionality

- Low: Considerations for limited functionality and additional input validation.
- Informational: Suggestions for gas efficiency optimizations.

## Summary of Findings

1. Critical:

   - Lack of Access Control: The contract allows anyone to set a new password, compromising its security. It is recommended to add a modifier to restrict the `setPassword` function to only the owner. This can be achieved by implementing the `onlyOwner` modifier and checking the `msg.sender` against the owner's address.
   - Alternative Solution: Inheriting the `Ownable` contract in the `PasswordStore` contract can provide built-in access control functionality.

2. High:

   - Visibility of Password: Storing the password as a private variable on the blockchain without additional encryption or secure storage mechanisms can potentially expose sensitive data. It is recommended to modify the `PasswordStore` contract to store the hashed version of the password instead of the plain text password. This can be achieved by using the `keccak256` hash function to compute the password hash and storing it in a `bytes32` variable.

3. Medium:

   - Error Handling: While the custom error handling approach is valid, it is important to ensure that all potential error scenarios are properly handled to prevent unexpected behavior. It is recommended to review the error handling logic and ensure that appropriate error messages and actions are implemented.
   - Missing Input Validation: Lack of input validation on the `newPassword` parameter can lead to potential vulnerabilities and attacks. It is recommended to implement input validation to ensure that the `newPassword` parameter meets certain criteria, such as length or format requirements.

4. Low:

   - Limited Functionality: The contract's limited functionality may not directly impact security but could be a consideration depending on the specific requirements of the use case.
   - Missing Input Validation: Implementing input validation to ensure the `newPassword` parameter meets certain criteria can further enhance the security of the contract.

5. Informational:
   - Gas Efficiency: Gas efficiency is not a vulnerability but rather a consideration for optimizing the contract's execution cost. It is important to review the contract for any gas-intensive operations and consider potential optimizations.

## Tools Used

The audit was conducted using manual code review techniques and analysis of the provided code snippets.

# Critical

- ## Lack of Access Control:
  - This vulnerability allows anyone to set a new password, compromising the security of the contract.
  - we must consider adding a modifier to restrict the setPassword fx to only the owner

```
error ONLY_OWNER_CAN_SET_NEW_PASSWORD();

 modifier onlyOwner() {
     if(msg.sender == s_owner) revert ONLY_OWNER_CAN_SET_NEW_PASSWORD();
     _;
 }

 function setPassword(string memory newPassword) external onlyOwner {
     // Rest of the function code
 }
```

- or the ultimatum we can Inherit the Ownable contract in the PasswordStore contract:

```
import "@openzeppelin/contracts/access/Ownable.sol";
contract PasswordStore is Ownable {
    // Rest of the contract code
    function setPassword(string memory newPassword) external onlyOwner {
    // Rest of the function code
}
}
```

# High

- Error Handling: While the custom error handling approach is valid, it is important to ensure that all potential error scenarios are properly handled to prevent unexpected behavior. It is recommended to review the error handling logic and ensure that appropriate error messages and actions are implemented.

# Medium

- ## Visibility of Password:
  - Storing the password as a private variable on the blockchain without additional encryption or secure storage mechanisms can potentially expose sensitive data.
  - It is recommended to modify the `PasswordStore` contract to store the hashed version of the password instead of the plain text password.
  - This can be achieved by using the `keccak256` hash function to compute the password hash and storing it in a `bytes32` variable.

```
contract PasswordStore is Ownable {
bytes32 private s_passwordHash;

// Rest of the contract code

function setPassword(string memory newPassword) external onlyOwner {
s_passwordHash = keccak256(abi.encodePacked(newPassword));
emit SetNetPassword();
}

// Rest of the contract code
}
```

# Low

- Limited Functionality: The contract's limited functionality may not directly impact security but could be a consideration depending on the specific requirements of the use case.
- Missing Input Validation: Lack of input validation on the `newPassword` parameter can lead to potential vulnerabilities and attacks. It is recommended to implement input validation to ensure that the `newPassword` parameter meets certain criteria, such as length or format requirements.

# Informational

- Gas Efficiency: Gas efficiency is not a vulnerability but rather a consideration for optimizing the contract's execution cost. It is recommended to review the contract for any gas-intensive operations and consider potential optimizations.

---

Please note that this report is based on the provided information and code snippets. It is important to conduct a thorough review and testing of the entire protocol to ensure its security and functionality.
