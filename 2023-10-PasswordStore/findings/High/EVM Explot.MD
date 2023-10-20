## Summary

This vulnerability enables any entity with a comprehensive understanding of the storage slots of the Ethereum Virtual Machine (EVM) to illicitly extract the password stored within the `PasswordStore` contract.

## Vulnerability Details

The vulnerability originates from the EVM's storage structure, known for its predictability and transparency. Through an exploration of the storage slots, a malicious entity can identify the specific storage slot housing the password.

### Attack Vectors: Exploiting EVM Storage Slots to Extract Password

#### Step 1: Deployment and Initialization

- Following the deployment steps as documented. Notably, the deployment script initializes the password as "myPassword."

```solidity
contract DeployPasswordStore is Script {
    function run() public returns (PasswordStore) {
        vm.startBroadcast();
        PasswordStore passwordStore = new PasswordStore();
        passwordStore.setPassword("myPassword");
        vm.stopBroadcast();
        return passwordStore;
    }
}
```

```

```

- The string `s_password` is stored in slot 1 of the contract storage.

```solidity
contract PasswordStore {
    error PasswordStore__NotOwner();
    address private s_owner;
    string private s_password;
}
```

#### Step 2: Determine the Contract Address

After deploying the contract, we will obtain the contract's address, which we'll refer to as `contract_address`. This address is required for accessing the contract's storage.

#### Step 3: Exploiting the EVM

- To exploit the EVM storage and extract the password, the attacker can use the "cast" tool from foundry suite and the appropriate parameters to loacate the byte code in a specific storage slot. Below is the command and its breakdown:

```sh
cast storage <contract_address> --rpc-url http://127.0.0.1:8545 1
```

- `cast storage`: This command tells the "cast" tool to retrieve data from the contract's storage.
- `<contract_address>`: Replace this with the actual contract address obtained during deployment.
- `--rpc-url http://127.0.0.1:8545`: Specifies the RPC URL for the Ethereum node in this case our ANVIL node

- The result of this command will be a hexadecimal string representing the stored password:

```sh
0x6d7950617373776726400000000000000000000000000000000000000000014
```

#### Step 4: Decode the Password

- The data obtained from the storage is encoded in bytes. To reveal the actual password, the attacker can use a decoding tool or script. In this case, the attacker can convert the bytes to ASCII using the `cast` :

```sh
cast --to-ascii 0x6d7950617373776726400000000000000000000000000000000000000000014
```

The result will be the actual password we have deplyod our contract with:

```sh
myPassword
```

### Attack Vector 2: Decode Input Data from Etherscan Transaction

-To decode this input data, w'll need to understand the structure of Ethereum function calls and the encoding of function parameters.

1. **MethodID:** The MethodID `0x290bb453` indicates the function being called, which is `setPassword(string newPassword)`.

2. **Function Parameters:**

   - `[0]` is a placeholder for the location of the first parameter, which is the offset to the actual data.
   - `[1]` is the length of the string parameter (in bytes).
   - `[2]` is the actual data, which is the new password encoded as a hexadecimal string.

3. **Decoding the String Parameter:**

   - We will use `[1]` to determine the length of the string parameter, which is `0x000000000000000000000000000000000000000000000000000000000000000e`. This indicates that the string is 14 bytes long.

   - We will use `[2]` to get the hexadecimal representation of the new password, which is `0x6e657750617373776f7264313233000000000000000000000000000000000000`.

4. **Decode to ASCII:**

   To reveal the actual password, we can use any tool or script of choice to decode the hexadecimal representation of the new password to ASCII characters. This will expose the new password set.

## Impact

The impact of this vulnerability is High with a medium likelyhood of occuring.

- It characterized by the compromise of the security of the password within the `PasswordStore` contract. Any malicious entity well-versed in the structure of EVM can flash out the password from its storage location, potentially leading to unauthorized access or disruptions to the protocol's operability.

## Tools Used

- Foundry's cast
  -Etherscan

## Recommendations

-To mitigate this vulnerability, I strongly recommended considering alternative storage mechanisms for sensitive data like passwords and avoiding the exposure of private information through the EVM. The security and confidentiality of sensitive data should be a top priority when designing and deploying smart contracts.
