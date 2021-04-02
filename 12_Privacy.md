## 12. Privacy

Goal for this exercise is:

- Unlock this contract to beat the level


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

contract Privacy {

  bool public locked = true;
  uint256 public ID = block.timestamp;
  uint8 private flattening = 10;
  uint8 private denomination = 255;
  uint16 private awkwardness = uint16(now);
  bytes32[3] private data;

  constructor(bytes32[3] memory _data) public {
    data = _data;
  }
  
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
  }

  /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
  */
}
~~~

I used `await web3.eth.getStorageAt(instance, <slot number>)` to copy
data from contract storage slots.
 
Instance storage slots were:
0:0x0000000000000000000000000000000000000000000000000000000000000001
1:0x000000000000000000000000000000000000000000000000000000006054884c
2:0x00000000000000000000000000000000000000000000000000000000884cff0a
3:0x9c0562554ab50b042caf3dd103b6231b6548abb14f05955da85baad2329cc9b5
4:0xe5b27c38e332d29e73a132f15e24da6eb6d2cc6a717e2f8e4ecd4a89ea1e3aa8
5:0x2360abf946c63a7eb1303ae04ef7f142aff57138737ae6669d4fd3ad384162d7
6:0x0000000000000000000000000000000000000000000000000000000000000000

Since slot 6 is empty all contract state variables  must be in the slots before it.


Here is the code I used to test in which part the needed key is stored:
~~~
pragma solidity ^0.6.0;

contract Privacy {

    bytes32[3] private data;
    
    // ["0x9c0562554ab50b042caf3dd103b6231b6548abb14f05955da85baad2329cc9b5",
    // "0xe5b27c38e332d29e73a132f15e24da6eb6d2cc6a717e2f8e4ecd4a89ea1e3aa8",
    // "0x2360abf946c63a7eb1303ae04ef7f142aff57138737ae6669d4fd3ad384162d7"]
    constructor(bytes32[3] memory _data) public {
        data = _data;
    }
    
    function getBytes16() public view returns (bytes16) {
        // 0x2360abf946c63a7eb1303ae04ef7f142    
        return bytes16(data[2]);
    }
  
}
~~~

Which is just first 16 bytes from slot 5.

Needed call to solve was:

- `contract.unlock("0x2360abf946c63a7eb1303ae04ef7f142")`
