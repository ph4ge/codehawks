

## Summary


1. No check for `address(0)` in `transferAdmin(address)` - `Adminable.sol`

## Vulnerability Details

### 1. No check for `address(0)` in `transferAdmin(address)` - `Adminable.sol`

The [`transferAdmin(address)`](https://github.com/Cyfrin/2024-05-Sablier/blob/43d7e752a68bba2a1d73d3d6466c3059079ed0c6/v2-core/src/abstracts/Adminable.sol#L34) (`0x75829def`) doesn't check that the next admin owner address is different than the null address.

## Impact

### 1. No check for `address(0)` in `transferAdmin(address)` - `Adminable.sol`

Not checking the admin can result in a lock of the `admin` role because of the non-operable aspect of `address(0)`.


## Tools Used 

- slither
- manual code analysis
- foundry toolbox

## Recommended Mitigation

### 1. No check for `address(0)` in `transferAdmin(address)` - `Adminable.sol`

The check can be performed in at several ways(`require`, `assert` statements) and in two different locations:

- Inside the `onlyAdmin` [modifier](https://github.com/Cyfrin/2024-05-Sablier/blob/43d7e752a68bba2a1d73d3d6466c3059079ed0c6/v2-core/src/abstracts/Adminable.sol#L22)
- Inside the [`transferAdmin(address)`](https://github.com/Cyfrin/2024-05-Sablier/blob/43d7e752a68bba2a1d73d3d6466c3059079ed0c6/v2-core/src/abstracts/Adminable.sol#L34)  function

```
require(msg.sender != address(0), "next admin cannot be null");
```
