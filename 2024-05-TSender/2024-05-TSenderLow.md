## Summary


**Compilation** of low risk findings/bugs/vulnerabilities:

1. `EquivalenceTest.sol` - an unused declared variable
2. `TSender.sol` - missing test file
3. `ITSender.sol` - interface function signature almost match
4. `ERC-20` - fuzzy compliancy state
5. Tokens integration - mutiple issues
6. Missing check in `transfer` function
7. Short address type attack
8. `Base_Test.t.sol` - constant variable not generalizable to all tokens



### Context

Some of the aforementioned findings/vulnerabilities/bugs are motivated by the following:

**Assumption**: The contract/project `TSender` seems to be positioned as a bulk sending token contract(s). With that in mind, the gas saving considerations are even more stringent than in "regular" projects. This is indexed to the potential of large number of addresses/recipients.
Moreover, the 3 declinations of the reference contract seems to abide this assumption.

In other words and for the project's specificity, squeezing out every possible bit of optimization also constitutes a **relevant** finding.



## Vulnerability Details

(Delve deep and elaborate on the identified issue, including where it exists in the codebase.)

### 1. `EquivalenceTest.sol` - an unused declared variable

In the files dedicated to test the project contracts, the `EquivalenceTest.sol` file contains an **unused** `constant` variable: `ONE`.

Line of interest location:

`https://github.com/Cyfrin/2024-05-TSender/blob/c6da9ef0c28741c007a02dfa07b7e899c1c22e47/test/EquivalenceTest.sol#L28`

Unused constant variable:

```
uint256 constant ONE = 1e18;
```

### 2. `TSender.sol` - missing test file

Contract `TSender.sol`, located under: 

`https://github.com/Cyfrin/2024-05-TSender/tree/c6da9ef0c28741c007a02dfa07b7e899c1c22e47/src/protocol` 

doesn't have a dedicated test file as for the reference implementation i.e. `TSenderReference.sol`, located under:

`https://github.com/Cyfrin/2024-05-TSender/tree/c6da9ef0c28741c007a02dfa07b7e899c1c22e47/src/reference`



### 3. `ITSender.sol` - interface function signature almost match

The interface `ITSender.sol` located under:

`https://github.com/Cyfrin/2024-05-TSender/tree/c6da9ef0c28741c007a02dfa07b7e899c1c22e47/src/interfaces`

doesn't have a 100% match with their implementations in `TSender.sol` and `TSenderReference.sol`(out of scope). The implementations have a visibility that are more gas optimized i.e. `pure`. For coherence and implementation model, the interface `ITSender.sol` should be prototyped as/with `pure`.

### 4. `ERC-20` - fuzzy compliancy state

The position of `TSender` project with regards to its compliance with the `ERC-20` standard is not crystal clear. This could lead to a number of attacks on `ERC-20` tokens that are(or loosely) compliant with the `ERC-20` standard.

The coroborating elements are the following:

1. The project description which expects to use the following tokens:
   - USDC
   - USDT
   - LINK
   - WETH

2. Its function name: `airdropERC20`

3. The reference implementation: `TSenderReference.sol` which uses OpenZeppelin's `IERC20` interface.

Concerning the `TSender.sol` contract, [WHAT ABOUT THE OTHER IMPLEMENTATIONS?]

The issue here is that an `ERC-20` compliant token/contract has to implement at least the following specifications:

```
totalSupply() - ?
balanceOf(address) - ?
transfer(address,uint256) - OK
transferFrom(address,address,uint256) - OK
approve(address,uint256) - ?
allowance(address,address) - ?
```

Reference: `https://eips.ethereum.org/EIPS/eip-20`

Considering that the project only uses a subset of the functionalities provided by the `ERC-20` standard, this doesn't automatically close the doors to "unforseen/unexpected" behaviors.

### 5. Tokens integration - mutiple issues

The following issues are linked/a byproduct of issue described in point `4.`

As indicated in the `TSender` project description, I flag/report the following elements pertaining to this project.

For:

1. USDT
   - The [transfer](https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code#L85) function doesn't specify a return value.

2. USDC
   - The smallest unit of representation of the token is non-standard `decimals`: [6](https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#readContract#F6)

### 6. Missing check in `transfer` function

The potential to transfer of tokens to the contract instance(`this`) seems to be logically little "inconsistent" with the contract primary objective.

### 7. Short address type attack

The requirements for this type of attack are:

- User input does not need to be sanitized/padding checked,
- The attacker has to control an address which ends with a null byte: `00`
- The contract/exchange has to have enough tokens in reserve so that the attacker can take advantage of how the evm handles the truncated address.

### 8. `Base_Test.t.sol` - constant variable not generalizable to all tokens

The `Base_Test.t.sol` testing contract uses a constant variable to compute and verify operations on tokens amounts. The [variable of concern](https://github.com/Cyfrin/2024-05-TSender/blob/c6da9ef0c28741c007a02dfa07b7e899c1c22e47/test/Base_Test.t.sol#L17): `ONE` which represents the atomic unit value.
The issue is that the integration with other `ERC-20` type tokens would behave "strangely" i.e. not as expected. This is because the smallest representing unit of a token differs from the standard value i.e. `1e18`. The token of concern is USDC which has a decimal of 6.

## Impact


### 1. `EquivalenceTest.sol` - an unused declared variable

This has a low impact.

### 2. `TSender.sol` - missing test file

This impacts the project's completeness of the code coverage tests and testing.

### 3. `ITSender.sol` - interface function signature almost match

In `ITSender.sol`, the function `areListsValid(address[] calldata,uint256[] calldata)` is declared with the `view` visibility.

If the contract is only exposed through the interface, this can lead to sub-optimal implementation and bytecode.

To illustrate this last point, hereafter follows a comparison of the bytecode of `TSender.sol` contract compiled with the `view` and `pure` visibilities.

After disassembly as individual instructions of **discrepancy between the two bytecodes**, we have:

If declared using the `view` visibility:

```
00000000: INVALID
00000001: DUP15
00000002: INVALID
00000003: BLOCKHASH
00000004: INVALID
00000005: INVALID
00000006: SWAP14
00000007: SWAP4
00000008: INVALID
00000009: INVALID
0000000a: INVALID
0000000b: INVALID
0000000c: INVALID
0000000d: SHL
0000000e: INVALID
0000000f: INVALID
00000010: INVALID
00000011: INVALID
00000012: INVALID
00000013: PUSH6 0x1e6643ba12e9
0000001a: DUP7
0000001b: INVALID
0000001c: INVALID
0000001d: LT
0000001e: INVALID
0000001f: INVALID
```

If declared using the `pure` visibility:

```
00000000: INVALID
00000001: INVALID
00000002: MUL
00000003: DUP6
00000004: DUP5
00000005: PUSH15 0xdb81f8f055ec8e0bf90f40da0dfa9e
00000015: INVALID
00000016: INVALID
00000017: PUSH1 0x68
00000019: INVALID
0000001a: INVALID
0000001b: INVALID
0000001c: BALANCE
0000001d: ISZERO
0000001e: SWAP1
0000001f: INVALID
```

There is a difference of 10 opcodes in favor of the `pure` version - based on a non-deployed bytecode.

### 4. `ERC-20` - fuzzy compliancy state

The issue that stems from using a selected/subset number of specifications of `ERC-20` is that:

- unexpected behaviors may still arise because of the disregard for the other functionalities
  - => it prevents thorough testing of the protocol


### 5. Tokens integration - mutiple issues

Respectively for issues 1 and 2,

1. The `transfer` function could fail silently thus preventing it to allocate tokens to the recicipent(s).
2. This needs to be taken into account when conducting arithmetic operations. Otherwise, the results of these would not represent the actual amounts that has to be dealt with.

### 6. Missing check in `transfer` function

If allowing sending tokens to the contract instance is an intented use case then there is no impact. However, transfering tokens to the contract can lead to withdrawal from a third-party.


### 7. Short address type attack


In the worst case scenario, an amount of tokens can be lost if sent to an non-existing address.

References:

https://ericrafaloff.com/analyzing-the-erc20-short-address-attack/

https://gist.github.com/Dexaran/ddb3e89fe64bf2e06ed15fbd5679bd20


### 8. `Base_Test.t.sol` - constant variable not generalizable to all tokens

The impact is that the arithmetic operations on amounts would not reflect the expected values to be computed. This issue is linked to the "fuzzy" `ERC-20` compliancy state of some tokens.


## Tools Used 

- slither
- manual code analysis
- foundry toolbox
- solc


## Recommended Mitigation


### 1. `EquivalenceTest.sol` - an unused declared variable

Remove unused variable at line:

`https://github.com/Cyfrin/2024-05-TSender/blob/c6da9ef0c28741c007a02dfa07b7e899c1c22e47/test/EquivalenceTest.sol#L28`


### 2. `TSender.sol` - missing test file


A possible test stub could similarly be like:

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;

import {Base_Test, ITSender} from "test/Base_Test.t.sol";
import {TSender} from "src/protocol/TSender.sol";
import {MockERC20} from "test/mocks/MockERC20.sol";

contract TSenderTest is Base_Test {
    function setUp() public {
        TSender tSenderRef = new TSender();
        tSender = ITSender(address(tSenderRef));
        mockERC20 = new MockERC20();
        _hasSafetyChecks = true;
    }
}
```

Thus, the `TSender.sol` contract needs the:

```
import {ITSender} from "src/interfaces/ITSender.sol";
```

and specify the inheritance:

```
contract TSender is ITSender
```

`https://github.com/Cyfrin/2024-05-TSender/blob/c6da9ef0c28741c007a02dfa07b7e899c1c22e47/src/protocol/TSender.sol#L17`


### 3. `ITSender.sol` - interface function signature almost match

The interface `ITSender.sol` located under:

`https://github.com/Cyfrin/2024-05-TSender/tree/c6da9ef0c28741c007a02dfa07b7e899c1c22e47/src/interfaces`

can be adjusted as follows:

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;

interface ITSender {
    function airdropERC20(address,address[] calldata,uint256[] calldata,uint256) external;
    function areListsValid(address[] calldata,uint256[] calldata) external pure returns (bool);
}
```


### 4. `ERC-20` - fuzzy compliancy state

There is at least two possible options:

1. Consider refactoring `ITSender` as an [abstract](https://docs.soliditylang.org/en/v0.8.24/contracts.html#abstract-contracts) contract,
2. Propose a new ERC/EIP


### 5. Tokens integration - mutiple issues

1. Similarly to what has been implemented in the `TSender.sol` contract, the type of token should be checked in order to properly handle the transfer function for USDT tokens.

2. Do not rely on the `decimals` functionality of the `ERC-20` standard to conduct arithmetic operations for USDC tokens.


### 6. Missing check in `transfer` function

In the `TSenderReference.sol` contract and adapt to their optimized versions, register `error` and a check in the following vein:

```
error TSender__ThisAddress();
```

```
if (recipients[i] == address(0)) {
	revert TSender__ZeroAddress();
        }
else if (recipients[i] != address(this)) {
	revert TSender__ThisAddress();
	}
```

### 7. Short address type attack

Input validation needs to be refined to take into account the validity of the recipient address. Checking for the address length is not sufficient.

Other possible path of mitigation: `ERC-223`


### 8. `Base_Test.t.sol` - constant variable not generalizable to all tokens

In order to avoid arithmetic operations that would not precisely represent the token amounts. The call to the token's `decimals` function should be used to conduct calculations. In other words, the use of a single variable which assumes that all the underlying `ERC-20` tokens have the same atomic unit is error-prone.



