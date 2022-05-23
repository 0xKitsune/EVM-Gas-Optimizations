# EVM-Gas-Optimizations

This repo was made to document of all of the gas optimizations that I have come across / experimented with. Each optimization has a brief explanation with a code snippet. This document is for internal purposes but I might make it public someday for others that might benefit from it as well. 

<br>

## `unchecked{++i}` instead of `i++`

Use `unchecked{++i}` instead of `i++`. This is especially useful in for loops but this optimization can be used anywhere in your code. 

```js

//unoptimized
//i++ -> Load, Store, Add, Store
for (uint i = 0; i < 10; i++) {
// code here
}

//optimized
//++i -> Load, Add, Store
for (uint i = 0; i < 10; ++i) {
// code here
}

//more optimized
for (uint i = 0; i < 10) {
// code here
 unchecked{++i;}
}

//even more optimized
assembly {
  for {let i := 0} lt(i, 10) {i := add(i, 0x01)} {
    //code goes here
  }
}

```


## Short circuiting

```js

// f(x) is low cost
// g(y) is expensive

//unoptimized
if (g(y) || f(x)){
    //code here
}


//optimized
if (f(x) || g(y)){
    //code here
}
```

## Use `calldata` instead of `memory` where possible

```js
contract Unoptimized {
    function unoptimizedGasTest(bytes memory data) public {}
}

contract Optimized {
    function optimizedGasTest(bytes calldata data) public {}
}

```

## Pack calldata where possible

```js

contract Unoptimized {
    function unoptimizedGasTest(address exampleAddress, uint256 exampleUint256) public{
        //do something
    }
}

contract Optimized {
    function optimizedGasTest(bytes calldata data) public {
        (address exampleAddress, uint256 exampleUint256) = abi.decode(data,(address, uint256));

        //do something
    }
}

```

## Pack structs

```js
struct UnoptimizedStruct{
    uint128 a;
    uint256 b;
    uint128 c;   
}

struct OptimizedStruct{
    uint128 a;
    uint128 b;
    uint256 c;   
}

```

## Mark storage variables as `immutable` if they never change after initialization


```js

contract Unoptimized {
    uint256 fee;
    
    constructor(){
        fee = 10;
    }
}


contract Optimized {
    uint256 immutable fee;
    
       constructor(){
        fee = 10;
    }
}
```


## Use constants for values that never change

```js
contract Unoptimized {
    uint256 fee;
    
    constructor(){
        fee = 10;
    }
}


contract Optimized {
    uint256 constant FEE = 10;
}


```

## `int` can be more expensive than `uint`

Negative ints are encoded with leading 0xf bytes. Nonzero bytes in calldata use 4x more gas than zero bytes.

```js

 »  abi.encode(int(-8))
0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff8


 »  abi.encode(uint(8))
0x0000000000000000000000000000000000000000000000000000000000000008

```


## Use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH.

```js
//unoptimized
uint256 thisContractBalance = address(this).balance;

//optimized
uint256 thisContractBalance;
assembly{
    thisContractBalance := selfbalance()
}
```


## Use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

```js
//unoptimized
uint256 contractBalance = someAddress.balance();

//optimized
uint256 contractBalance;
assembly{
    contractBalance := balance(someAddress)
}
```


## Use assembly to hash efficiently

```js
//unoptimized
keccak256(abi.encodePacked(a, b))

//optimized
assembly{
  mstore(0x00, a)
  mstore(0x20, b)
  keccak256(0x00, 0x40)
}
```


## Use `if iszero(x)` instead of `if (x)`

```js
//unoptimized
if (x){
  //code here
}

//optimized
assembly{
  if iszero(x){
    //code here
   }
}
```


## Right shift instead of dividing by two

```js

contract Unoptimized {
    function unoptimizedGasTest() public {
        uint256 val = 10;
        uint256 valMulTwo = val * 2;
    }
}

contract Optimized {
    function optimizedGasTest() public {
        uint256 val = 10;
        uint256 valMulTwo = val >> 2;
    }
}
```

## Left shift instead of multiplying by two

```js

contract Unoptimized {
    function unoptimizedGasTest() public {
        uint256 val = 10;
        uint256 valMulTwo = val/2;
    }
}

contract Optimized {
    function optimizedGasTest() public {
        uint256 val = 10;
        uint256 valMulTwo = val << 2;
    }
}
```

## Use assembly to check for address(0)
```js
    //unoptimized
    function unoptimizedGasTest(address owner) public view {
        require(owner != address(0), "zero address)");
    }

    //optimized
    function optimizedGasTest(address owner) public view {
        assembly {
            if iszero(owner) {
                mstore(0x00, "zero address")
                revert(0x00, 0x20)
            }
        }
    }

```

## Use assembly to check if msg.sender == owner (or any stored address)


```js

contract Unoptimized {
    address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

    function unoptimizedGasTest() public {
        require(msg.sender == owner, "!auth");
    }
}

contract Optimized {
    address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

    function optimizedGasTest() public {
        assembly {
            if iszero(eq(sload(owner.slot), caller())) {
                mstore(0x00, "!auth")
                revert(0x00, 0x20)
            }

        }
    }
}

```


## Use assembly to update owner (or any stored address/value)


```js


contract Unoptimized {
    address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

    function unoptimizedGasTest(address newOwner) public {
        owner = newOwner;
    }
}

contract Optimized {
    address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

    function optimizedGasTest(address newOwner) public {
        assembly {
            sstore(owner.slot, newOwner)
        }
    }
}

```

## Use multiple `require()` statments insted of `require(expression && expression && ...)`

```js

//unoptimized
function singleRequire(uint256 num) public pure {
    require(num > 1 && num < 10 && num == 3);
}


//optimized
function multipleRequire(uint256 num) public pure {
        require(num > 1);
        require(num < 10);
        require(num == 3);
}


