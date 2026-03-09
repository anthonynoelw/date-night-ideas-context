# Feature Roadmap: Date Night AI

This roadmap is split into logical phases, focusing on **Backend/API first**, followed by the Frontend implementation for each phase.

## Phase 1: Foundation (Local Dev MVP)
*Goal: Establish a working end-to-end connection from Laravel to local Ollama and a basic API structure.*

### 1.1 Laravel Backend & API (Primary Focus)
- [ ] **Versioning Framework:** Set up API versioning (e.g., `/api/v1/...`) to ensure future scalability.
- [ ] **Ollama Integration:** Create an `LLMService` to send HTTP requests to a local Ollama instance (default: `localhost:11434`).
- [ ] **Input Checker (Validation):** Robust FormRequest validation for all incoming generation parameters.
- [ ] **Generation Endpoint:** `POST /api/v1/generate-ideas` accepting budget, vibe, energy, and no-goes.
- [ ] **Simple Prompt Builder:** Logic to transform validated inputs into a structured string prompt for the LLM.
- [ ] **Response Parsing:** Logic to handle the LLM's raw text output and format it into clean JSON for the client.

### 1.2 Flutter Frontend (Secondary)
- [ ] **Initial Setup:** Project structure and basic navigation.
- [ ] **Input Mask (Form):**
    - [ ] Budget (Slider/Input).
    - [ ] Vibe Selector (**Tags**: Action, Romantic, Chill).
    - [ ] Energy Level (Low, Medium, High).
    - [ ] **Tags** for "No-Goes".
- [ ] **Results View:** A clean list to display the JSON-parsed ideas.
- [ ] **Dual-Theme Logic:** Basic implementation of "Default" vs. "Woman-Look" (Rose/Gold) toggle.

---

## Phase 2: Data Enrichment & Context (Smart MVP)
*Goal: Enhance AI suggestions using real-time external data.*

### 2.1 Backend Enrichment (API v1.1)
- [ ] **Weather Integration:** Implement `OpenWeatherService` to fetch weather data based on date/location.
- [ ] **Geo-Data Processing:** Logic to handle coordinates (Lat/Lon) and pass location context to the LLM.
- [ ] **Advanced Prompting:** Use Blade templates for prompts to inject dynamic context (e.g., "It's raining outside").
- [ ] **JSONB History:** Store every request and LLM response in PostgreSQL for history and debugging.

### 2.2 Frontend Enhancements
- [ ] **Location Integration:** Use `geolocator` to automatically detect and send coordinates to the API.
- [ ] **Weather Preview:** Small UI widget showing the forecast for the selected date.

---

## Phase 3: User Management & Compliance
*Goal: Secure authentication and GDPR-compliant data handling.*

### 3.1 Backend Security & Quota
- [ ] **Laravel Sanctum Auth:** Implementation of secure API tokens for mobile users.
- [ ] **Weekly Quota Logic:** Database-level check to enforce the "5 free ideas per week" limit.
- [ ] **User Profile API:** Endpoints to store/retrieve persistent preferences (No-Goes, UI Mode).
- [ ] **GDPR Tools:** Automated data export and account deletion endpoints.

### 3.2 Frontend User Experience
- [ ] **Auth Flow:** Login, Registration, and Profile management screens.
- [ ] **Quota Indicator:** Display how many free ideas are remaining for the current week.

---

## Phase 4: Monetization & Production (Scale)
*Goal: Transition to professional cloud services and revenue generation.*

### 4.1 Payment & Production Backend
- [ ] **Stripe Integration:** Webhook handling for successful payments/subscriptions.
- [ ] **Production LLM Driver:** Toggle between local Ollama and OpenAI/Claude via ENV variables.
- [ ] **Response Caching:** Intelligent caching to reduce LLM costs for identical request patterns.

### 4.2 Polished Frontend
- [ ] **Stripe Checkout:** Seamless payment flow within the app.
- [ ] **Enhanced Animations:** High-quality animations (Lottie/Rive) for the specialized "Woman-Look".