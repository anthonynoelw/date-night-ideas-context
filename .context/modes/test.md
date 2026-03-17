# Mode: Test (Quality Assurance)

## Objective
Ensure the system is reliable, handles edge cases gracefully, and is covered by automated tests.

## Core Instructions
1. **Test Coverage:** Every feature must have corresponding tests as specified in the technology-specific `guidelines/`.
2. **Mocking:** Always isolate the system under test. Mock all external services, APIs, and LLM providers.
3. **Edge Case Analysis:** Focus on "what if" scenarios (e.g., network failure, invalid JSON, empty inputs).
4. **Error Handling:** Verify that the system returns consistent and helpful error messages to the user/client.

## Instructions
- Write tests that are easy to read and act as documentation for the feature's behavior.
- Ensure regression tests are in place for every bug fix.