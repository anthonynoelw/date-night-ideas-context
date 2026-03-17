# Database Structure Plan

This document outlines the final database schema for the Date Night AI project. It uses **ULIDs** for all primary keys (except for standard Laravel tables where unnecessary) and separates concerns into logical clusters.

Database is a Postresql Db

## 1. User & Access Cluster
Handling authentication, extensive user preferences, and subscription management (Cashier).

### `users`
* **`id`**: ULID (Primary Key)
* **`name`**: String
* **`email`**: String (Unique)
* **`email_verified_at`**: Timestamp (Nullable)
* **`password`**: String (Hashed)
* **`remember_token`**: String (100, Nullable)
* **`deleted_at`**: Timestamp (Nullable) - *Soft deletes enabled*
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

**Relationships:**
* **1:1** - `user_settings`, `personal_access_tokens`, `subscriptions`, `legal_consents`
* **1:n** - `generation_logs` (generations), `data_export_requests` (exports)

### `user_settings`
Extensible storage for all user preferences (e.g., `theme_preference` = `rose_gold`).
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users, Nullable)
* **`key`**: String (Indexed, e.g., 'ui_theme', 'measurement_system')
* **`value`**: Text (The actual value)
* **`type`**: Enum ('string', 'boolean', 'integer', 'json') - Helps frontend casting
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `personal_access_tokens` (Sanctum)
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users, Nullable)
* **`name`**: String
* **`token`**: String (Unique, Hashed)
* **`abilities`**: Text (JSON, Nullable)
* **`last_used_at`**: Timestamp (Nullable)
* **`expires_at`**: Timestamp (Nullable)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `subscriptions` (Cashier)
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users, Nullable)
* **`type`**: String (Default: 'default')
* **`stripe_id`**: String (Unique)
* **`stripe_status`**: String (active, trialing, past_due, canceled)
* **`stripe_price`**: String (Nullable)
* **`quantity`**: Integer (Nullable)
* **`ends_at`**: Timestamp (Nullable)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `subscription_items` (Cashier)
* **`id`**: ULID (Primary Key)
* **`subscription_id`**: ULID (Foreign Key -> subscriptions)
* **`stripe_id`**: String (Unique)
* **`stripe_product`**: String
* **`stripe_price`**: String
* **`quantity`**: Integer (Nullable)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

---

## 2. Generation Engine Cluster
The core logic for creating and storing date ideas. Segregates the **inputs** (Log) from the **outputs** (Idea).

### `generation_logs`
Records *what* was asked for (Analytics/History).
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users, Nullable)
* **`latitude`**: Decimal (10,8)
* **`longitude`**: Decimal (11,8)
* **`city`**: String (Resolved city name for history)
* **`weather_snapshot`**: JSON (Full payload from OpenWeather at that second)
* **`maximum_budget`**: Decimal (e.g., 100.00)
* **`energy_level`**: String (low, medium, high)
* **`llm_provider`**: String (e.g., 'ollama', 'openai')
* **`llm_model`**: String (e.g., 'llama3')
* **`prompt_tokens`**: Integer
* **`completion_tokens`**: Integer
* **`processing_time_ms`**: Integer
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

**Relationships:**
* **m:n** - `vibes` (via `generation_log_vibes`)
* **m:n** - `activity_types` (via `generation_log_activity_types`)
* **m:n** - `no_goes` (via `generation_log_no_goes`)

### `generated_ideas`
Records *what* was returned.
* **`id`**: ULID (Primary Key)
* **`generation_log_id`**: ULID (Foreign Key -> generation_logs)
* **`title`**: String
* **`description`**: Text
* **`location_suggestion`**: String (Nullable, specific place if AI suggested one)
* **`estimated_cost`**: String
* **`is_saved`**: Boolean (Default: false)
* **`feedback_score`**: Integer (Nullable, 1-5 for RLHF)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

---

## 3. Taxonomy Cluster
Managing categorization to avoid hardcoded strings.

### `vibes`
Specialized taxonomy for the core "Vibe" selection.
* **`id`**: ULID (Primary Key)
* **`name`**: String (Display name)
* **`slug`**: String (Unique)
* **`description`**: String (Nullable, for UI)
* **`prompt_instruction`**: Text (Hidden instructions injected into LLM)
* **`icon`**: String (Nullable)
* **`is_active`**: Boolean (Default: true)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `generation_log_vibes`
Pivot linking logs to multiple vibes.
* **`generation_log_id`**: ULID (Foreign Key -> generation_logs)
* **`vibe_id`**: ULID (Foreign Key -> vibes)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `activity_types`
Replaces generic tags for classifying the specific type of activity.
* **`id`**: ULID (Primary Key)
* **`name`**: String
* **`slug`**: String (Unique)
* **`icon`**: String (Nullable)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `generation_log_activity_types`
Pivot linking logs to multiple activity types.
* **`generation_log_id`**: ULID (Foreign Key -> generation_logs)
* **`activity_type_id`**: ULID (Foreign Key -> activity_types)
* **`created_at`**: Timestamp

### `no_goes`
Replaces generic tags for managing restrictions, dietary needs, and activities to explicitly avoid.
* **`id`**: ULID (Primary Key)
* **`name`**: String
* **`slug`**: String (Unique)
* **`icon`**: String (Nullable)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `generation_log_no_goes`
Pivot linking logs to multiple no goes.
* **`generation_log_id`**: ULID (Foreign Key -> generation_logs)
* **`no_go_id`**: ULID (Foreign Key -> no_goes)
* **`created_at`**: Timestamp

---

## 4. Caching & Utility
optimizing expensive API calls.

### `weather_caches`
* **`id`**: ULID (Primary Key)
* **`latitude`**: Decimal (8,4) (Rounded for fuzzy match)
* **`longitude`**: Decimal (9,4) (Rounded for fuzzy match)
* **`date`**: Date
* **`data`**: JSON (OpenWeather response)
* **`expires_at`**: Timestamp
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `geo_caches`
* **`id`**: ULID (Primary Key)
* **`latitude`**: Decimal (8,4)
* **`longitude`**: Decimal (9,4)
* **`city`**: String
* **`country`**: String
* **`formatted_address`**: String
* **`expires_at`**: Timestamp
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

---

## 5. Compliance & Legal
GDPR Support.

### `legal_consents`
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users, Nullable)
* **`document_type`**: String (e.g., 'tos', 'privacy')
* **`version`**: String (e.g., '1.0')
* **`ip_address`**: String (Nullable, Anonymized)
* **`agreed_at`**: Timestamp

### `data_export_requests`
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users, Nullable)
* **`status`**: String (pending, processing, completed, failed)
* **`download_path`**: String (Nullable)
* **`expires_at`**: Timestamp
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

---

## 6. Standard Laravel Tables (Reference)
These use standard auto-incrementing IDs or strings as per framework defaults.

* `failed_jobs`
* `jobs`
* `cache`
* `cache_locks`
* `sessions`
* `password_reset_tokens`

