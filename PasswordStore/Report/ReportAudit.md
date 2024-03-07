# Audit Report

## Summary

The purpose of the contract is to store the `_newPassword` given by the owner, and only the owner should be able to change and see.  
There was key errors in the code that could easily be fixed, let's go through them;
One major privacy problem was found related to storing of passwords; Another was found related to control access in the `setPassword()` function. Only the owner should be able to call the function, but it doesn't have any way to know who is calling. 

## Audit Scope
Contracts audited:

    ./src/PasswordStore.sol


## Findings
Let's go over the main problems found in PasswordStore.sol:  

- No Control Access in `setPassword()`  
There is no way for the function to "know" who is calling, the descriptions states that only the owner should be able to, but there's no Control Access for it.
 
- `s_Password` is not safe by being private.  
It's stored on chain, making it public to anyone to see and not being "private" as intended, we demonstrate how it's possible to see the valued stored.
      

- "@Param" comment in getPassword is unecessary.
There is no reason for that comment, as you can see, the function does not need `newPassword`:  

      * @param newPassword The new password to set. <- unnecessary
      function getPassword() external view returns (string memory) {
        if (msg.sender != s_owner) {
            revert PasswordStore__NotOwner();
        }
        return s_password;
      }

## Proof of Concept

The case below is a way that someone can access and read the password directly from the Blockchain.

1. Create a local chain and deploy the contract 
```
make anvil
make deploy
```
2. Using the storage tool
```
cast storage <ADRESS> 1 --rpc-url <LOCALHOST>
```
Output will be similar to:  
`0x6d795...0014`  

Parse the hex to a string:
```
cast parse-bytes32-string <OUTPUT>
```  

And we'll see the password:  
```
myPassword
```


## Recommendations

The focus should be in the 2 first problems, as it's related to security, privacy and breach treat.

- Add a Control Access to setPassword():  
    - 
      function setPassword(string memory newPassword) external {
        if(msg.sender != s_owner){
            revert passwordStore_NotOwner();
        }
      }

- Making the password private
    - 
    > To make the password private, a solution would be to store the password off-chain, and send the encrypted password to the blockchain. In the actual architecture, it's not possible to store the password secure and privately. A architecture redesign would be highly recommended. Using third-party open-source tools could be a way to facilitate that redesign!  


- Remove the comment line
    - 
```diff
- * @param newPassword The new password to set.
```
## Conclusion

There are 2 main core problems that sould be fixed ASAP.  
Those are security and privacy concerns that also break how the code works.  By fixing this, the code should run properly and users can be safe and garantee their privacy.
