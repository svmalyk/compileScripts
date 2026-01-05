# C++ Project Coding Rules Template

> **Purpose:** Ensure readable, reliable, secure, and maintainable C++ code throughout the project lifecycle.

## 1) Scope & Objectives
- **Applies to:** All C++ code in this repository (production, tests, tools).
- **Goals:** Consistency, safety, performance, portability, and clear ownership.
- **Non-goals:** Enforcing personal style preferences over documented rules.

---

## 2) Language & Toolchain

### 2.1 C++ Standard
- **Target standard:** `C++20` *(update if different)*  
- **Compatibility:** Code must compile with GCC and Clang (specify exact versions).

### 2.2 Compiler Flags
- **Required warnings:**  
  - GCC/Clang: `-Wall -Wextra -Wpedantic -Wconversion -Wshadow -Wnon-virtual-dtor -Wold-style-cast -Woverloaded-virtual -Wnull-dereference -Wdouble-promotion`
- **Treat warnings as errors:** `-Werror` *(allowed to relax locally via documented justification)*

### 2.3 Static Analysis
- **Tools:** `clang-tidy`, `cppcheck` *(configure in CI)*  
- **clang-tidy config:** `.clang-tidy` in repo root, include checks for readability, performance, bugprone, modernize.

### 2.4 Build System
- **Primary:** CMake ≥ 3.22  
- **Minimum flags:**  
  ```cmake
  set(CMAKE_CXX_STANDARD 20)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
  set(CMAKE_CXX_EXTENSIONS OFF)
  add_compile_options(-Wall -Wextra -Wpedantic -Werror)
  ```
- **Generator expressions** must be used for platform-specific flags.

---

## 3) Formatting & Style

### 3.1 Automated Formatting
- **clang-format:** Use `.clang-format` checked into repo.  
- **Pre-commit enforcement:** Add git hook or pre-commit config.

#### Suggested `.clang-format` baseline
```yaml
BasedOnStyle: Google
IndentWidth: 4
TabWidth: 4
UseTab: Never
ColumnLimit: 120
PointerAlignment: Left
AllowShortFunctionsOnASingleLine: Empty
BreakConstructorInitializers: AfterColon
SpaceAfterCStyleCast: false
IncludeBlocks: Regroup
SortIncludes: true
NamespaceIndentation: All
AlignAfterOpenBracket: Align
```

### 3.2 Naming Conventions
- **Files:** `snake_case`, e.g., `http_client.cpp`
- **Namespaces:** `lowercase`, hierarchical (e.g., `net::http`)
- **Classes/Types:** `PascalCase`
- **Methods/Functions:** `camelCase`
- **Variables:** `camelCase`
- **Constants:** `kPascalCase` (for internal) or `ALL_CAPS` for macros only
- **Member variables:** suffix `_` or `m_` (pick one consistently). Example uses `_`.
- **Templates/concepts:** `PascalCase` for types, `camelCase` for concepts.

### 3.3 File Organization
- One class per file (exceptions: small utilities).
- **Header guards:** `#pragma once` (preferred) or include guard `PROJECT_MODULE_FILENAME_H`.
- Public headers in `include/<project>/...`; internal headers in `src/...`.

---

## 4) Includes & Dependencies

### 4.1 Include Order
1. Own header (if corresponding `.cpp`)
2. Local project headers
3. Third-party library headers
4. Standard library headers

### 4.2 Include Rules
- Use forward declarations where possible in headers.
- Prefer `<header>` over `"header"` only for standard library.
- Avoid cyclic dependencies; refactor via interfaces.

---

## 5) Namespaces & Modules
- Avoid global namespace pollution.
- Use anonymous namespaces in `.cpp` for internal linkage.
- **No `using namespace` at global scope**. Use qualified names or local `using` within function scope only.

---

## 6) Classes, Structs & APIs

### 6.1 Class Design
- Favor **RAII** for resource management.
- Avoid exposing raw owning pointers; use smart pointers.
- Make constructors `explicit` unless intentional implicit conversion.

```cpp
class HttpClient {
public:
    explicit HttpClient(Config config);
    std::string get(const Url& url) const;
private:
    Config config_;
};
```

### 6.2 Rule of Zero/Five
- Prefer **Rule of Zero**; define special members only if you manage resources manually.
- If needed, implement all relevant special members (copy/move/ctor/dtor).

### 6.3 Const-correctness
- Mark methods `const` when they don’t mutate observable state.

---

## 7) Functions & Parameters
- Keep functions small (<50 LOC ideally); single responsibility.
- Pass by:
  - **Value** for small types
  - **`const&`** for large types or to avoid copies
  - **`unique_ptr<T>`** to transfer ownership
  - **`shared_ptr<T>`** only when shared ownership is essential
- Avoid default arguments in virtual functions.

---

## 8) Error Handling Policy

### 8.1 Exceptions
- **Policy:** *(choose one and stick to it project-wide)*
  - **A)** Exceptions allowed for recoverable errors; no exceptions in destructors; noexcept where sensible.
  - **B)** Exceptions banned; use `expected<T, E>` or error codes.

> If **A)**:
- Throw standard exceptions (`std::runtime_error`, etc.); attach context.
- Mark functions `noexcept` if they won’t throw.
- Use RAII to maintain strong exception safety.

> If **B)**:
- Provide `expected<T, Error>` type (e.g., `tl::expected` or custom).
- Propagate errors explicitly; no hidden failures.

### 8.2 Assertions vs Runtime Checks
- Use `assert()` for internal invariants (compiled out in Release).
- Use explicit runtime checks for user inputs or IO.

---

## 9) Memory Management & Ownership

- Prefer `std::unique_ptr` for exclusive ownership.
- Use `std::shared_ptr` only when shared ownership is required; avoid cycles.
- No manual `new/delete` in application code—wrap in factories returning smart pointers.
- Avoid raw pointers for ownership; raw pointers only for non-owning references.

```cpp
std::unique_ptr<Widget> makeWidget(Config cfg);
void useWidget(const Widget* widget); // non-owning
```

---

## 10) Concurrency

- Use `<thread>`, `<mutex>`, `<future>`, `<atomic>`; prefer **RAII locks**.
- Avoid data races; guard shared mutable state with `std::mutex`.
- Prefer task-based abstractions over manual thread management.
- Document thread-safety guarantees in public APIs (`Thread-safe`, `Not thread-safe`, `Conditionally thread-safe`).

---

## 11) Performance

- Measure before optimizing; include micro-benchmarks where useful.
- Avoid premature optimization; write clear code first.
- Use move semantics; avoid unnecessary copies.
- Consider `reserve()` for vectors and `emplace_back()` appropriately.
- Prefer `std::string_view` for non-owning string parameters.

---

## 12) Security

- Validate all external inputs; avoid undefined behavior.
- Avoid C-style string APIs; use `std::string`/`std::string_view`.
- Prevent integer overflow/underflow where relevant (`std::numeric_limits` checks).
- Avoid UB: no out-of-bounds, no dangling references, no double-free.
- Zeroize sensitive buffers when applicable.

---

## 13) Logging & Diagnostics

- Use a centralized logging facility (e.g., spdlog/loguru or custom).
- Log at appropriate levels: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`.
- No PII or secrets in logs.
- Include correlation IDs for distributed systems.

---

## 14) Documentation

- Doxygen-compatible comments for public headers.
- Each module has a `README.md` describing purpose, architecture, and examples.
- Document invariants, preconditions, postconditions in headers.

```cpp
/// Computes the normalized score.
/// @param rawScore [0,100]
/// @returns [0,1]
double normalize(double rawScore);
```

---

## 15) Testing

- **Framework:** *(e.g., GoogleTest)*  
- **Coverage target:** ≥ 80% (lines), ≥ 70% (branches) — enforced in CI.
- Unit tests for logic; integration tests for IO/threads; property tests for parsers.
- No global state in tests; deterministic and isolated.

---

## 16) Code Review Checklist

**Must pass before merge:**
- [ ] Builds on all supported compilers/targets  
- [ ] No new compiler warnings; static analysis clean or justified  
- [ ] Follows formatting and naming rules  
- [ ] Ownership and lifetimes are clear; no raw owning pointers  
- [ ] Exception/error policy respected  
- [ ] Thread-safety documented and correct  
- [ ] Tests added/updated; coverage unaffected or improved  
- [ ] Public APIs documented; breaking changes announced  
- [ ] Security considerations addressed (inputs validated, no UB)

---

## 17) Directory & Layout

```
/cmake/                 # cmake helpers, toolchains
/include/<project>/     # public headers
/src/                   # implementation
/tests/                 # unit/integration tests
/tools/                 # dev tools/scripts
/docs/                  # architecture & usage docs
/.clang-format
/.clang-tidy
/CMakeLists.txt
/CODING_RULES.md
```

---

## 18) Example: Header & Implementation

**`include/project/http_client.h`**
```cpp
#pragma once
#include <string>
#include <string_view>

namespace project::net {

class HttpClient {
public:
    explicit HttpClient(std::string baseUrl);

    /// Performs a GET request.
    /// @throws std::runtime_error on network errors.
    std::string get(std::string_view path) const;

private:
    std::string baseUrl_;
};

} // namespace project::net
```

**`src/http_client.cpp`**
```cpp
#include "project/http_client.h"
#include <stdexcept>

namespace project::net {

HttpClient::HttpClient(std::string baseUrl) : baseUrl_(std::move(baseUrl)) {
    if (baseUrl_.empty()) {
        throw std::invalid_argument("baseUrl must not be empty");
    }
}

std::string HttpClient::get(std::string_view path) const {
    // TODO: Implement networking (placeholder)
    if (path.empty()) {
        throw std::invalid_argument("path must not be empty");
    }
    return "OK";
}

} // namespace project::net
```

---

## 19) Configuration Files (Templates)

**`.clang-tidy`**
```yaml
Checks: >
  bugprone-*,performance-*,readability-*,modernize-*,
  cppcoreguidelines-*,hicpp-*
WarningsAsErrors: '*'
HeaderFilterRegex: '.*'
FormatStyle: none
```

**Pre-commit (optional) `.pre-commit-config.yaml`**
```yaml
repos:
  - repo: https://github.com/pre-commit/mirrors-clang-format
    rev: v18.1.3
    hooks:
      - id: clang-format
        files: \.(h|hpp|cpp|cc|cxx)$
  - repo: https://github.com/cpplint/cpplint
    rev: v1.6.1
    hooks:
      - id: cpplint
        args: [--filter=-legal/copyright]
```

---

## 20) Governance & Exceptions

- Changes to this document require approval by the **Architecture/Tech Lead**.
- Exceptions must be documented inline with justification and tracked in PR description.
- Deprecations follow a 2-release grace period unless security-critical.

---

## 21) Portability Targets

- Supported OS/architectures: *(fill in: e.g., Linux x86_64, Windows, macOS)*  
- Use feature detection (`CMAKE_CXX_COMPILER_ID`, `__has_include`) and avoid platform-specific APIs in core libraries.

---

## 22) Onboarding Checklist

- [ ] Install toolchain (GCC/Clang versions)  
- [ ] Configure IDE to use `.clang-format`/`.clang-tidy`  
- [ ] Build & run tests locally  
- [ ] Read `docs/architecture.md`  
- [ ] Read `CODING_RULES.md` and sign off in first PR
