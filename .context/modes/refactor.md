# Mode: Refactor (Maintenance)

## Objective
Improve code quality and maintainability without changing the external behavior of the system.

## Core Instructions
1. **Simplify:** Reduce complexity and break down large components/methods into smaller, reusable units.
2. **Decouple:** Identify tight coupling and introduce proper abstractions as defined in the `guidelines/`.
3. **Standardize:** Ensure all code follows the naming and structural conventions of the project.
4. **No Regressions:** Refactoring must not break existing features. Consult `modes/test.md` if necessary.

## Rules
- Leave the codebase cleaner than you found it.
- Prioritize readability over "clever" one-liners.