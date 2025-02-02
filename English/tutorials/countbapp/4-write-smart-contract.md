## 4. Writing smart contract (Count.sol)

### 1) Background
We will make a super simple contract called "Count".  

a. There would be just one storage variable called `count`.  
b. Users can increase `count` variable by 1 or decrease it by 1. So there would be two functions, `plus` function which increases `count` variable by 1, and `minus` function which decreases `count` variable by 1. That's all!

### 2) Define the variable
Before setting a variable, we should specify the solidity version. Let's use 0.4.24 stable version.
```solidity
pragma solidity 0.4.24; // Specify solidity's version
```

Then we will name our contract "Count".
```solidity
pragma solidity 0.4.24;

contract Count { // set contract names to "Count"

}
```

We need to declare the variable `count` as `uint`(unsigned integer) type, and initialize it to be 0.

```solidity
pragma solidity 0.4.24;

contract Count {
  uint public count = 0; // Declare count variable as uint type and intialize its value to 0.
}
```

### 3) Define functions
We need two functions, `plus` and `minus`. Each functions's role is:  
`plus` - increase the `count` by 1. (count = count + 1)  
`minus` - decrease the `count` by 1. (count = count - 1)  


```solidity
pragma solidity 0.4.24;

contract Count {
  uint public count = 0;

  function plus() public { // Make a public function called 'plus'
    count = count + 1; // 'plus' function increases count variable by 1.
  }

  function minus() public { // Make a public function called 'plus'
    count = count - 1; // 'minus' function decreases count variable by 1.
  }
}
```

*NOTE*  
To allow the functions to be called outside the contract, functions should be declared as `public`.

```solidity
function plus() public { … }
```

### 4) Let's do something more.
We want to add one more feature. How about remembering the last participant's wallet address?

#### 4-1) Add a variable
So we will have a variable, `lastParticipant` as `address` type:  
`address public lastParticipant;`

```solidity
pragma solidity 0.4.24;

contract Count {
  uint public count = 0;
  address public lastParticipant;

  function plus() public { // Make a public function called 'plus'
    count = count + 1; // 'plus' function increases count variable by 1.
  }

  function minus() public { // Make a public function called 'plus'
    count = count - 1; // 'minus' function decreases count variable by 1.
  }
}
```

#### 4-2) Update functions
To track the last particpant's address, we store the address to `lastParticipant` like the below:  

```solidity
pragma solidity 0.4.24;

contract Count {
  uint public count = 0;
  address public lastParticipant;

  function plus() public {
    count = count + 1;
    lastParticipant = msg.sender; // store msg.sender to lastParticipant
  }

  function minus() public {
    count = count - 1;
    lastParticipant = msg.sender; // store msg.sender to lastParticipant
  }
}
```

*NOTE*  
1) `public`
If you declare a variable or a function as `public`,  you can access them outside the blockchain,
i.e., you can access this variable or function from your frontend application.
You will see how to interact with the contract public methods and variables from the frontend application in the [Count componenent](5-3-frontend-count-component.md) chapter.

2) `msg.sender`  
`msg.sender` is the address that initiated the current transaction.  
To get the address of the transaction sender we can use `msg.sender` variable.

```solidity
lastParticipant = msg.sender;
```

This line will make the `lastParticipant` to have the `msg.sender`.
