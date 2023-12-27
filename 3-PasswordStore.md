### [H-1] Storing the password On-chain makes it visible to everyone

**Description:** 

All data stored on chain is public and visible to anyone . The `PasswordStore::s_password` is intended to be hidden and only accessible by the owner through the `PasswordStore::getPassword` function .

**Impact:** 

Anyone is able to read the private password, severely breaking the functionality of the protocol .

**Proof of Concept:**

The below test case shows how anyone could read the password directly from the blockchain .

We use foundry's cast to read directly from the storage of the contract, without being the owner .



Create a locally running chain

>  make anvil

Deploy the contract to the chain .

> make deploy

Run the storage tool .

We use 1 because that's the storage slot of `s_password` in the contract .

`cast storage <ADDRESS_HERE> 1 --rpc-url http://127.0.0.1:8545`

You will get the output that looks like this 

> 0x6d7950617373776f726400000000000000000000000000000000000000000014

And then , you can use `cast` the parse that hex to a string .

>  cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014

And get an output of:

> myPassword



**Recommended Mitigation:** 

Due to this , the overall architecture of the contract should be rethought .

One could encrypt the password off-chain, and then store the encrypted password on-chain .

This would require the user to remember another password off-chain to decrypt the stored password . 

However you're also likely to remove the view function as you wouldn't want the user to accidentally send a transaction with the decryption key .



### [H-2] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

**Description:** 

 The `PasswordStore::setPassword` function is set to be an `external` function, however the purpose of the smart contract and function's natspec indicate that `This function allows only the owner to set a new password.`

```js
function setPassword(string memory newPassword) external {
    // @Audit - There are no Access Controls.
    s_password = newPassword;
    emit SetNewPassword();
}
```



**Impact:** 

Anyone can set/change the stored password, severely breaking the contract's intended functionality



**Proof of Concept:**

**Recommended Mitigation:** 





### [H-3] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect