## 8. Vault

Goal for this exercise is:

- Unlock the vault to pass the level


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) public {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
~~~

<br>

Had to learn how`await web3.eth.getStorageAt(<address>, <storage slot>)`can be used to fetch data.

In this contract `password`is located in 2nd storage slot.
Calling `await web3.eth.getStorageAt(instance, 1)` returns:
`0x412076657279207374726f6e67207365637265742070617373776f7264203a29`
which is our password.

Call to unlock was:

- `contract.unlock("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")`