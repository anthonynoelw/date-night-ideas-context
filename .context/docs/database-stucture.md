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
* **`stripe_id`**: String (Indexed, Nullable)
* **`pm_type`**: String (Nullable)
* **`pm_last_four`**: String (4, Nullable)
* **`trial_ends_at`**: Timestamp (Nullable)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `user_settings`
Extensible storage for all user preferences (e.g., `theme_preference` = `rose_gold`).
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users)
* **`key`**: String (Indexed, e.g., 'ui_theme', 'measurement_system')
* **`value`**: Text (The actual value)
* **`type`**: Enum ('string', 'boolean', 'integer', 'json') - Helps frontend casting
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `personal_access_tokens` (Sanctum)
* **`id`**: ULID (Primary Key)
* **`tokenable_type`**: String
* **`tokenable_id`**: ULID
* **`name`**: String
* **`token`**: String (Unique, Hashed)
* **`abilities`**: Text (JSON, Nullable)
* **`last_used_at`**: Timestamp (Nullable)
* **`expires_at`**: Timestamp (Nullable)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `subscriptions` (Cashier)
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users)
* **`type`**: String (Default: 'default')
* **`stripe_id`**: String (Unique)
* **`stripe_status`**: String (active, trialing, past_due, canceled)
* **`stripe_price`**: String (Nullable)
* **`quantity`**: Integer (Nullable)
* **`trial_ends_at`**: Timestamp (Nullable)
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
* **`budget_tier`**: String (e.g., '$-$$')
* **`energy_level`**: String (low, medium, high)
* **`llm_provider`**: String (e.g., 'ollama', 'openai')
* **`llm_model`**: String (e.g., 'llama3')
* **`prompt_tokens`**: Integer
* **`completion_tokens`**: Integer
* **`processing_time_ms`**: Integer
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

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

### `tags`
General-purpose tags for restrictions, dietary needs, etc.
* **`id`**: ULID (Primary Key)
* **`name`**: String
* **`slug`**: String (Unique)
* **`type`**: String (Enum: 'restriction', 'dietary', 'activity_type')
* **`icon`**: String (Nullable)
* **`created_at`**: Timestamp
* **`updated_at`**: Timestamp

### `taggables`
Polymorphic pivot to attach tags to Users (Preferences) or Logs (Specific Request filters).
* **`tag_id`**: ULID (Foreign Key -> tags)
* **`taggable_id`**: ULID
* **`taggable_type`**: String
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
* **`user_id`**: ULID (Foreign Key -> users)
* **`document_type`**: String (e.g., 'tos', 'privacy')
* **`version`**: String (e.g., '1.0')
* **`ip_address`**: String (Nullable, Anonymized)
* **`agreed_at`**: Timestamp

### `data_export_requests`
* **`id`**: ULID (Primary Key)
* **`user_id`**: ULID (Foreign Key -> users)
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

