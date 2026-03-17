# Feature Roadmap: Date Night AI

This roadmap follows an **API-First** approach. Security, authentication, and secure data handling are integrated throughout the entire development lifecycle rather than being treated as an afterthought. Development starts with the backend and API infrastructure before moving to the frontend.

## Phase 1: Core API & Security Foundation (Backend)
*Goal: Establish a secure, robust Laravel API with basic user management and LLM integration.*

* **Security & Authentication Setup:** Implement Laravel Sanctum for secure API tokens, enforce HTTPS, and set up the foundation for GDPR-compliant data storage (data minimization principles).
* **Database & User Profiles:** Create user schemas, persistent preferences (No-Goes, UI Mode, vibes), and user legal consent records.
* **Versioning Framework:** Set up a structured API versioning system (e.g., `/api/v1/`) to ensure long-term stability.
* **Ollama Integration:** Create an `LLMService` to securely communicate with a local Ollama instance for development.
* **Validation & Input Handling:** Build robust FormRequest validation classes for all incoming generation parameters to prevent injection or malformed data.
* **Generation Endpoint:** Develop the `POST /api/v1/generate-ideas` endpoint to process budget, vibe, energy, and no-goes securely.
* **Prompt Builder & Parser:** Implement logic to construct context-rich prompts and safely parse raw LLM text into standardized JSON responses.

## Phase 2: Context Enrichment & Production Scaling (Backend)
*Goal: Enhance AI accuracy with real-time data and prepare the API for production and monetization.*

* **Weather & Location Integration:** Build services to fetch real-time OpenWeather data based on coordinates, implementing caching to prevent rate-limit abuse and reduce costs.
* **Advanced Prompting:** Update prompt templates to dynamically inject environmental context (e.g., "It's raining outside") securely.
* **Data History & Quotas:** Implement database logic to track request history securely and enforce "5 free ideas per week" rate limiting per user.
* **GDPR Tooling:** Develop automated endpoints for user data export and account deletion to guarantee compliance.
* **Granular API Key Management:** Allow consumers to generate multiple API keys with fine-grained permissions (e.g., Read-only, Write-only). This supports the principle of least privilege, limits exposure if a key is compromised, and provides clear audit trails for different integrations.
* **Monetization Hooks:** Integrate Stripe webhooks for secure subscription and payment handling.
* **Production LLM Driver:** Add secure environment variables to easily toggle between local Ollama and cloud-based LLMs (e.g., OpenAI/Claude) for production.

## Phase 3: Core App Setup & Experience (Frontend)
*Goal: Build the initial Flutter application connecting to the secure foundational API.*

* **Project Initialization:** Set up the Flutter project structure, safe storage for API tokens, and core routing.
* **Auth Flow:** Implement Login, Registration, and Profile management screens connecting to the Sanctum endpoints.
* **Input Mask (Form):** Build user interfaces for parameters (Budget slider, Vibe tags, Energy Level, No-Goes) ensuring client-side validation matches the API rules.
* **Results View:** Create a clean, accessible list interface to parse and display the structured JSON date ideas.
* **Quota & Status Indicators:** Show users their remaining free generations and provide transparent loading states for data processing.

## Phase 4: Polish & Advanced Features (Frontend)
*Goal: Implement context-aware features, premium upgrades, and UI refinements.*

* **Location Services:** Integrate `geolocator` to securely prompt for permissions and detect user coordinates for the API.
* **Contextual UI:** Add small widgets (like Weather Previews) to show users the active context for their generated date.
* **Dual-Theme Logic:** Implement the "Default" and "Woman-Look" (Rose/Gold) theme toggles, respecting user preferences stored in the API.
* **Premium Upgrades (Stripe Checkout):** Create seamless, secure in-app purchase flows to upgrade limits.
* **Enhanced Animations:** Add high-quality animations (Lottie/Rive) for loading states and specialized visual experiences.