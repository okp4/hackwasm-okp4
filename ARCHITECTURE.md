# Architecture

## Overview

This library consists of several packages:

- **`prolog`:** `database/sql`-like high-level interface for interpreter
- **`prolog/engine`:** virtual machine and other implementation details
- **`prolog/cmd/1pl`:** simple toplevel
- **`prolog/examples`:** example programs

## Virtual Machine

We based our VM on one described in [A PORTABLE PROLOG COMPILER (Bowen et al. 83)](http://www.softwarepreservation.org/projects/prolog/lisbon/lpw83/p74-Bowen.pdf) which is presumably referred as ZIP by others.

ZIP is a high-level VM written in Prolog.
Thus, we had to bring missing control features in Go like nondeterminism and cut.
Hence, our deviation from the original ZIP arose.

### Instructions

We use the same VM opcodes you can find in the original paper:

- `opGetConst` / `opPutConst`
- `opGetVar` / `opPutVar`
- `opGetFunctor` / `opPutFunctor`
- `opPop`
- `opEnter`
- `opCall`
- `opExit`

Note that `const`, `var`, and `functor` in the original paper work differently in the head/body with the clever use of a differential list.
We split them into `opGet*` for the head and `opPut*` for the body so that we can base our VM on Go slices instead of (differential) lists.

Also, we've added some extra opcodes:

- `opCut` to perform cut operation
- `opGetList` / `opPutList` to handle lists
- `opGetPartial` / `opPutPartial` to handle partial lists

### Registers

We use the same registers you can find in the original paper except `xr`:

- `pc`
- `vars`
- `cont`
- `args`
- `astack`

We don't need `xr` register because each instruction can point to the constant via a Go pointer.
In other words, our XR table is the Go heap.

Also, we've added some extra registers:

- `env` to keep track of variable bindings (environment)
- `cutParent` to keep track of cut parent
