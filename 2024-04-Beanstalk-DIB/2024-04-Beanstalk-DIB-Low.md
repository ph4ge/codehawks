# Low Risk Findings

## <a id='L-01'></a>L-01. Mocking provider reentrancy            

### Relevant GitHub Links
	
https://github.com/BeanstalkFarms/Basin/blob/74759bc3d94e954d9aeae078c2baf8982aca7d74/mocks/wells/MockReserveWell.sol

## Summary

Reentrancy in `MockReserveWell.sol` function `update(address,uint256[],bytes)`

## Vulnerability Details

A pump is an interface to support/operationalize Multi Flow - `MultiFlowPump.sol`.

In order to simulate/test Well reserves and the functioning of Multi Flow, a mocking mechanism is necessary. These are provided with a number of mocks.

The tests make use of these mocks. The object of concern is: `MockReserveWell.sol`

Tests for Multi Flow pumps are located in `test/pumps/`:

```bash
test/pumps/
├── Pump.CapReserve.t.sol
├── Pump.Fuzz.t.sol
├── PumpHelpers.sol
├── Pump.Helpers.t.sol
├── Pump.Longevity.t.sol
├── Pump.NotInitialized.t.sol
├── Pump.TimeWeightedAverage.t.sol
├── Pump.Update.t.sol
└── simulate.py
```

The mocking mechanism of interest is provided by: `mocks/wells/MockReserveWell.sol`

This mock is used by `Pump.Update.t.sol` which instantiates a `MockReserveWell` to proceed with testing.



## Impact

Severity: Low

Contract: MockReserveWell

Both `MultiFlowPump` and `MockReserveWell` implement `IPump` interface.

Function name: update(address,uint256[],bytes)

SigHash: 9e67eb4a



## Tools Used

- High level UML
- Manual code analysis
- Mythril

## Recommendations

Consider that no state modifications are executed after this call by applying a form of reentrancy guard.
