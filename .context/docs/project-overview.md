# Project Overview: Date Night AI

## 1. Vision & Purpose
A mobile application designed for couples and daters to generate personalized date night ideas. By combining user preferences (budget, vibe, energy level) with real-time data (weather, location), the app uses LLMs to create unique, actionable suggestions.

## 2. Target Audience
* Couples looking to break their routine.
* People on dates seeking tailored activities.
* Users who value aesthetic diversity (Dual-Theme support).

## 3. Architecture Overview
The system follows a classic Client-Server architecture with a focus on security and scalability:
* Client (Flutter): Handles the UI, location services (Geo-data), and theme switching. It communicates exclusively with the Laravel API.
* Use ddev enviornment
* Server (Laravel ): Acts as the central orchestrator. It validates requests, fetches external weather data, manages user quotas (Stripe-integrated), and constructs prompts for the LLM.
* Database (PostgreSQL): Stores user profiles, preferences, and session history.
* LLM Layer: Development: Local Ollama instance. Production: Cloud-based LLM (e.g., OpenAI/Anthropic) accessed via Laravel.

## 4. Key Features (MVP)
* Smart Input Mask: Captures age, date, location (Geo-data), radius, budget, "No-Goes" (dietary/activities), vibe (action/romantic), and energy level.
* Weather-Aware Generation: Integrates real-time weather data for the selected date to ensure suggestions are practical (Indoor vs. Outdoor).
* Dual-Theme System: Offers a "Woman-Look" (Rose/Gold, specialized animations) and a "Default-Look" (minimalist), toggleable at any time.
* Monetization (Stripe): Free Tier: 5 generated ideas per week. Premium: Paid access via Stripe for additional generations to cover API costs.
* GDPR Compliance: Built with privacy-first principles (data minimization, right to be forgotten).

## 5. External Integrations
* Stripe: Subscription and quota management.
* OpenWeather API: Real-time and forecasted weather data.
* LLM Provider: Ollama (Local) / OpenAI or similar (Cloud).