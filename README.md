# EVM-Gas-Optimizations

This repo was made to document of all of the gas optimizations that I have come across / experimented with. Each optimization has a brief explanation with a code snippet. This document is for internal purposes but I might make it public someday for others that might benefit from it as well. 

<br>

## `unchecked{++i}` instead of `i++` (or use assembly when applicable)

Use `++i` instead of `i++`. This is especially useful in for loops but this optimization can be used anywhere in your code. You can also use `unchecked{++i;}` for even more gas savings but this will not check to see if `i` overflows. For extra safety if you are worried about this, you can add a require statement after the loop checking if `i` is equal to the final incremented value. For best gas savings, use inline assembly, however this limits the functionality you can achieve. For example you cant use Solidity syntax to internally call your own contract within an assembly block and external calls must be done with the `call()` or `delegatecall()` instruction. However when applicable, inline assembly will save much more gas.

### Optimization
```js

contract GasReport {
       
    //loop with i++
    function iPlusPlus() public pure {
        uint256 j = 0;
        for (uint256 i; i < 10; i++) {
            j++;
        }
    }
    
    //loop with ++i
    function plusPlusI() public pure {
        uint256 j = 0;
        for (uint256 i; i < 10; ++i) {
            j++;
        }
    }

    //loop with unchecked{++i}
    function uncheckedPlusPlusI() public pure {
        uint256 j = 0;
        for (uint256 i; i < 10; ) {
            j++;

            unchecked {
                ++i;
            }
        }
    }

    //loop with unchecked{++i} with additional overflow check
    function safeUncheckedPlusPlusI() public pure {
        uint256 j = 0;
        uint256 i = 0;
        for (i; i < 10; ) {
            j++;

            unchecked {
                ++i;
            }
        }
        
        //check for overflow
        assembly {
            if lt(i, 10) {
                mstore(0x00, "loop overflow")
                revert(0x00, 0x20)
            }
        }
    }

    //loop with inline assembly
    function inlineAssemblyLoop() public pure {
        assembly {
            let j := 0

            for {let i := 0} lt(i, 10) {i := add(i, 0x01)} {
                j := add(j, 0x01)
            }
        }
    }
}

```

### Gas Report
```js
╭────────────────────────┬─────────────────┬──────┬────────┬──────┬─────────╮
│ GasReport contract     ┆                 ┆      ┆        ┆      ┆         │
╞════════════════════════╪═════════════════╪══════╪════════╪══════╪═════════╡
│ Deployment Cost        ┆ Deployment Size ┆      ┆        ┆      ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 89535                  ┆ 479             ┆      ┆        ┆      ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ Function Name          ┆ min             ┆ avg  ┆ median ┆ max  ┆ # calls │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ iPlusPlus              ┆ 2061            ┆ 2061 ┆ 2061   ┆ 2061 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ inlineAssemblyLoop     ┆ 753             ┆ 753  ┆ 753    ┆ 753  ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ plusPlusI              ┆ 1989            ┆ 1989 ┆ 1989   ┆ 1989 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ safeUncheckedPlusPlusI ┆ 1421            ┆ 1421 ┆ 1421   ┆ 1421 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ uncheckedPlusPlusI     ┆ 1417            ┆ 1417 ┆ 1417   ┆ 1417 ┆ 1       │
╰────────────────────────┴─────────────────┴──────┴────────┴──────┴─────────╯

```


## Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly as seen below.

### Optimization
```js

contract GasReport {

    //addition in Solidity
    function addTest(uint256 a, uint256 b) public pure {
        uint256 c = a + b;
    }

    //addition in assembly
    function addAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := add(a, b)

            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }

    //subtraction in Solidity
    function subTest(uint256 a, uint256 b) public pure {
        uint256 c = a - b;
    }

    //subtraction in assembly
    function subAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := sub(a, b)

            if gt(c, a) {
                mstore(0x00, "underflow")
                revert(0x00, 0x20)
            }
        }
    }

    //multiplication in Solidity
    function mulTest(uint256 a, uint256 b) public pure {
        uint256 c = a * b;
    }

    //multiplication in assembly
    function mulAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := mul(a, b)

            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }

    //division in Solidity
    function divTest(uint256 a, uint256 b) public pure {
        uint256 c = a * b;
    }

    //division in assembly
    function divAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := div(a, b)

            if gt(c, a) {
                mstore(0x00, "underflow")
                revert(0x00, 0x20)
            }
        }
    }
}
 

```

### Gas Report
```js

╭────────────────────┬─────────────────┬─────┬────────┬─────┬─────────╮
│ GasReport contract ┆                 ┆     ┆        ┆     ┆         │
╞════════════════════╪═════════════════╪═════╪════════╪═════╪═════════╡
│ Deployment Cost    ┆ Deployment Size ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 128377             ┆ 673             ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ Function Name      ┆ min             ┆ avg ┆ median ┆ max ┆ # calls │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ addAssemblyTest    ┆ 308             ┆ 308 ┆ 308    ┆ 308 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ addTest            ┆ 391             ┆ 391 ┆ 391    ┆ 391 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ divAssemblyTest    ┆ 332             ┆ 332 ┆ 332    ┆ 332 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ divTest            ┆ 369             ┆ 369 ┆ 369    ┆ 369 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ mulAssemblyTest    ┆ 354             ┆ 354 ┆ 354    ┆ 354 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ mulTest            ┆ 348             ┆ 348 ┆ 348    ┆ 348 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ subAssemblyTest    ┆ 329             ┆ 329 ┆ 329    ┆ 329 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ subTest            ┆ 322             ┆ 322 ┆ 322    ┆ 322 ┆ 1       │
╰────────────────────┴─────────────────┴─────┴────────┴─────┴─────────╯

```


## Short circuiting
When an if statement has more than one possible case, list the more freqently occuring case first. This will allow less computation to evaluate the case and enter the body of the if statement. The gas report shown below is a result of passing in `0` as `a`. 

```js
contract GasReport {
    function noShortCircuit(uint256 a) public pure {
        if (a > 1000 || a != 0) {}
    }

    function shortCircuit(uint256 a) public pure {
        if (a != 0 || a > 1000) {}
    }
}
```

### Gas Report
```js
╭────────────────────┬─────────────────┬─────┬────────┬─────┬─────────╮
│ GasReport contract ┆                 ┆     ┆        ┆     ┆         │
╞════════════════════╪═════════════════╪═════╪════════╪═════╪═════════╡
│ Deployment Cost    ┆ Deployment Size ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 39293              ┆ 227             ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ Function Name      ┆ min             ┆ avg ┆ median ┆ max ┆ # calls │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ noShortCircuit     ┆ 251             ┆ 251 ┆ 251    ┆ 251 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ shortCircuit       ┆ 229             ┆ 229 ┆ 229    ┆ 229 ┆ 1       │
╰────────────────────┴─────────────────┴─────┴────────┴─────┴─────────╯

```




## Use `calldata` instead of `memory` where possible
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`.

```js

contract GasReport {
    function useMemory(uint256[] memory data) public pure {
        //do something
    }

    function useCalldata(uint256[] calldata data) public pure {
        //do something
    }
}

```

### Gas Report
```js
╭────────────────────┬─────────────────┬─────┬────────┬─────┬─────────╮
│ GasReport contract ┆                 ┆     ┆        ┆     ┆         │
╞════════════════════╪═════════════════╪═════╪════════╪═════╪═════════╡
│ Deployment Cost    ┆ Deployment Size ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 95941              ┆ 511             ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ Function Name      ┆ min             ┆ avg ┆ median ┆ max ┆ # calls │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ useCalldata        ┆ 375             ┆ 375 ┆ 375    ┆ 375 ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ useMemory          ┆ 680             ┆ 680 ┆ 680    ┆ 680 ┆ 1       │
╰────────────────────┴─────────────────┴─────┴────────┴─────┴─────────╯

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


