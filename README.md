# sovereign-adr

> C++ admissibility validator for the SnapKitty ecosystem.
> Non-recursive staged traversal. Prime index validation. Binary op validation.

[![License: Sovereign Source](https://img.shields.io/badge/License-Sovereign%20Source-blue.svg)](SOVEREIGN.md)
[![C++](https://img.shields.io/badge/C++-20-blue.svg)](https://isocpp.org/)
[![Tests](https://img.shields.io/badge/Tests-6%20passing-brightgreen.svg)](#testing)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    ADMISSIBILITY VALIDATOR                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │                    Input AST                               │  │
│   │              (source code tree)                           │  │
│   └──────────────────────────────────────────────────────────┘  │
│                           │                                      │
│                           ▼                                      │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │              Staged Traversal (non-recursive)             │  │
│   │                                                          │  │
│   │   Stage 1: Prime Index Validation                        │  │
│   │   Stage 2: Binary Operation Validation                   │  │
│   │   Stage 3: Stratum Boundary Validation                   │  │
│   │   Stage 4: Type Consistency Validation                   │  │
│   │                                                          │  │
│   └──────────────────────────────────────────────────────────┘  │
│                           │                                      │
│          ┌────────────────┼────────────────┐                    │
│          ▼                ▼                ▼                    │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐       │
│  │   Accepted   │ │   Rejected   │ │  Rejection       │       │
│  │   Receipt    │ │   Receipt    │ │  Reason          │       │
│  └──────────────┘ └──────────────┘ └──────────────────┘       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Validation Rules

| Rule | Description | Error Code |
|------|-------------|------------|
| **PRIME_INDEX** | All indices must be prime | `E001` |
| **BINARY_OP** | Binary ops must have two operands | `E002` |
| **STRATUM** | Stratum boundaries must be monotonic | `E003` |
| **TYPE_CONSIST** | Operands must have matching types | `E004` |
| **NO_RECURSION** | No recursive definitions allowed | `E005` |
| **CONSTANT_FOLD** | Constants must be foldable | `E006` |

## Quick Start

### Build

```bash
mkdir build && cd build
cmake ..
make -j$(nproc)
```

### Run Tests

```bash
./admissibility_test
```

### API Usage

```cpp
#include "AdmissibilityValidator.h"

using namespace pirtm;

int main() {
    // Create validator
    AdmissibilityValidator validator;
    
    // Validate AST
    AST ast = parse_source(source_code);
    ValidationResult result = validator.validate(ast);
    
    if (result.accepted) {
        std::cout << "AST is admissible" << std::endl;
        std::cout << "Receipt: " << result.receipt.to_json() << std::endl;
    } else {
        std::cout << "AST rejected: " << result.reason << std::endl;
        std::cout << "Rejection: " << result.rejection.to_json() << std::endl;
    }
}
```

## Interactive Demo

```bash
# Demo 1: Valid AST
$ ./admissibility valid_source.pirtm
✓ AST accepted
  Receipt: {
    "status": "accepted",
    "hash": "sha256:a1b2c3d4...",
    "timestamp": "2025-01-01T00:00:00Z"
  }

# Demo 2: Invalid AST (non-prime index)
$ ./admissibility invalid_source.pirtm
✗ AST rejected
  Reason: Non-prime index at line 5
  Rejection: {
    "status": "rejected",
    "error": "E001",
    "line": 5,
    "hash": "sha256:e5f6a7b8..."
  }

# Demo 3: Batch validation
$ ./admissibility --batch sources/
Processed 10 files:
  ✓ 8 accepted
  ✗ 2 rejected
```

## Rejection Receipt Shape

```json
{
  "standard": "PIRTM-ADMISSIBILITY-1",
  "algorithm": "sha256-v1",
  "status": "rejected",
  "error": "E001",
  "message": "Non-prime index",
  "location": {
    "line": 5,
    "column": 12
  },
  "source_hash": "sha256:a1b2c3d4...",
  "rejection_hash": "sha256:e5f6a7b8...",
  "timestamp": "2025-01-01T00:00:00Z"
}
```

## Invariants

| Invariant | Description |
|-----------|-------------|
| **No Recursion** | Traversal is iterative stack-based |
| **Deterministic** | Same AST → same result |
| **Fail-Closed** | Any error terminates validation |
| **Observable** | All rejections are logged with receipts |

## Testing

```bash
# Run all tests
./admissibility_test

# Run with verbose output
./admissibility_test --verbose
```

## License

Sovereign Source License — see [SOVEREIGN.md](SOVEREIGN.md)

---

```
SOVEREIGN-ADR-001
Validate. Traverse. Reject. Receipt. Seal.
Same AST. Same result.
No recursion. No borrowed thesis.
```


---

## Citation

If you use this work, please cite:

```bibtex
@misc{snapkittywest2026sovereigncompute,
  title = {SNAPKITTYWEST: Sovereign Compute Architecture with Linear Types, WORM Seals, and Goldilocks Field Arithmetic},
  author = {SnapKitty Collective},
  year = {2026},
  doi = {10.5281/zenodo.21132094},
  url = {https://doi.org/10.5281/zenodo.21132094}
}
```

**Paper:** https://doi.org/10.5281/zenodo.21132094
**ORCID:** https://orcid.org/0009-0006-1916-5245
