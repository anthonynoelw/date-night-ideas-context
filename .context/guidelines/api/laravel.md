# Laravel API Guidelines & Best Practices

This document defines the architectural standards and coding practices for the Date Night AI Backend, optimized for development with **Laravel Boost (MCP)**.

## 1. Development Tools & AI Integration
- **Laravel Boost (MCP):** Use the Laravel Boost MCP server to provide the LLM with deep context about the codebase, including routes, models, and service structures.
- **Prompt Management:** LLM prompts must be stored as **Blade templates** in `resources/views/prompts/`. This allows for dynamic prompt construction and easy versioning without touching PHP logic.

## 2. Core Principles & Architecture
- **Strict Typing:** All methods must use return types and property types (PHP 8.2+).
- **DTOs (Data Transfer Objects):** Use DTOs for all data passing between Controllers and Services.
    - *Preferred Package:* `spatie/laravel-data` (integrates validation + DTO).
- **Service Layer Pattern:** All business logic (LLM communication, Weather fetching) must reside in dedicated Service classes.
- **Single Action Classes:** For complex, isolated tasks (e.g., `app/Actions/GenerateDateIdeaAction.php`), use the Action pattern to keep services clean.

## 3. API Standards & Versioning
- **Versioning:** All endpoints must be versioned (e.g., `/api/v1/...`).
- **RESTful naming:** Use kebab-case for URLs (`/api/v1/date-ideas`).
- **Data Integrity:** Use `FormRequests` for validation and `API Resources` for consistent JSON responses.

## 4. LLM & Ollama Integration (Driver Pattern)
To allow switching between local Ollama and Production LLMs (OpenAI/Claude), use an interface-based driver approach:

- **Interface:** `app/Interfaces/LLMDriverInterface.php`
- **Implementation (Local):** `app/Services/LLM/OllamaDriver.php`
- **Logic:** Laravel uses the `Http` client to communicate with the local Ollama API (`stream: false`).

## 5. Security & Compliance (GDPR)
- **Sanctum:** Use Laravel Sanctum for secure, token-based mobile authentication.
- **Rate Limiting:** Implement strict throttling (RateLimiting) on all generation endpoints to prevent (D)DoS and brute force attacks.
- **Data Minimization:** Adhere to GDPR by only storing necessary user data.
- **Encryption:** Personal identifiable information (PII) must be encrypted at rest.

## 6. Database & Storage
- **PostgreSQL JSONB:** Use JSONB columns for storing LLM outputs and user-specific flexible settings.
- **Response Caching:** Cache external API responses (e.g., OpenWeather) for at least 30 minutes.

## 7. Error Handling & Testing
- **JSON Errors:** Always return a consistent error structure with appropriate HTTP status codes (422 for validation, 401 for auth, etc.).
- **Pest PHP:** Use Pest for all tests.
- **Mocking:** Always mock external API calls (LLM, Weather) in tests to ensure speed and zero costs.