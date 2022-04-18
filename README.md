# EVM-Gas-Optimizations

This repo was made to document of all of the gas optimizations that I have come across / experimented with. Each optimization has a brief explanation with a code snippet. This document is for internal purposes but I might make it public someday for others that might benefit from it as well. 

<br>

## `++i` instead of `i++`

Use `++i` instead of `i++`. This is especially useful in for loops but this optimization can be used anywhere in your code. 

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

```


## Short circuiting

```js
//
```

## Use `calldata` instead of `memory` where possible

```js
//
```

## Pack calldata where possible

```js
//
```

## Pack structs


```js
//
```

## Mark storage variables as `immutable` if they never change after initialization


```js
//
```


## Use constants for values that never change

```js
//
```

## `int` can be more expensive than `uint`

Negative ints are encoded with leading 0xf bytes. Nonzero bytes in calldata use 4x more gas than zero bytes.

```js

 »  abi.encode(int(-8))
0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff8


 »  abi.encode(uint(8))
0x0000000000000000000000000000000000000000000000000000000000000008

```


## Use `selfbalance()` instead of `balance(this)`

```js
//
```


## Use `balance(address)` instead of `address.balance()`

```js
//
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
//
```

## Left shift instead of multiplying by two

```js
//
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


