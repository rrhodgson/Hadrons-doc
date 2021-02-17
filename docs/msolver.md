# MSolver

-----------

## MADWFCG

### Template structure

`FImplInner` a fermion implementation for the inner solver.

`FImplOuter` a fermion implementation for the outer solver.

`nBasis` an integer for the eigenpack basis size.

### Description

Produces a solver function to perform the MADWF algorith to solve a M&ouml;bious fermion propagator using a ZM&ouml;bious inner solver. In addition the inner solver can be deflated by specifying an eigenpack to use.

Note that the guess subtraction is not implemented and will throw a Hadrons error.

### Parameters


| Parameter           | Type           | Description                                             |
|---------------------|----------------|---------------------------------------------------------|
| `innerAction`       | `std::string`  | The action for the inner solver.                        |
| `outerACtion`       | `std::string`  | The action for the outer solver.                        |
| `maxInnerIteration` | `unsigned int` | The maximum number of itterations for the inner solver. |
| `maxOuterIteration` | `unsigned int` | The maximum number of itterations for the outer solver. |
| `residual`          | `double`       | Stopping residual for both the inner and outer solvers. |
| `eigenPack`         | `std::string`  | The eigenpack for deflation of the inner solver. (empty string for no eigenpack)       |


### Dependencies

This module depends on two actions, and an eigenpack if specified.

### Products

This module produces a `Solver`.

-----------
