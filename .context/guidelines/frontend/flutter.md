# Flutter Frontend Guidelines & Best Practices

This document defines the architectural standards and coding practices for the Date Night AI Flutter application.

## 1. Core Architecture (Clean Architecture)
To ensure maintainability and testability, we follow a layer-based approach:
- **Data Layer:** Repositories and Data Sources (API clients using `dio`).
- **Domain Layer:** Models (Entities) and Business Logic.
- **Presentation Layer:** Widgets and State Management (Providers).

## 2. State Management
- **Riverpod:** Use `flutter_riverpod` for all state management.
- **AsyncValue:** Always use `AsyncValue` to handle loading, error, and data states for API calls.
- **Code Generation:** Use `riverpod_generator` for type-safe and simplified provider definitions.

## 3. Dual-Theme System (Theming Strategy)
The app must support two distinct visual modes: **Default** and **Woman-Look**.
- **ThemeData:** Define two separate `ThemeData` objects in `lib/theme/app_themes.dart`.
- **Dynamic Switching:** The active theme must be controlled via a `StateProvider` or `Notifier` that persists the choice in local storage (e.g., `shared_preferences`) and syncs with the Laravel backend.
- **Woman-Look specifics:** Use custom `ThemeExtension` for specialized colors (Rose/Gold) and animations that aren't covered by standard Material 3 properties.

## 4. API Communication
- **Dio:** Use the `dio` package for all HTTP requests.
- **Interceptors:** Implement an interceptor for handling:
    - Bearer Tokens (Laravel Sanctum).
    - Consistent Error Handling (Mapping Laravel's 422/401 errors to Flutter exceptions).
- **JSON Serialization:** Use `freezed` and `json_serializable` for immutable data models and type-safe JSON parsing.

## 5. UI & UX Standards
- **Material 3:** Use Material 3 as the base design system.
- **Responsive Layouts:** Use `LayoutBuilder` or `ScreenUtil` to ensure the app looks great on different screen sizes.
- **Animations:** 
    - *Simple:* Use implicit animations (`AnimatedContainer`, `AnimatedOpacity`).
    - *Complex:* Use `Lottie` or `Rive` for the specialized animations in "Woman-Look".
- **Validation:** Client-side validation must mirror Laravel's `FormRequest` logic for immediate user feedback.

## 6. Project Structure
```text
lib/
├── src/
│   ├── core/           # Common utilities, constants, theme definitions
│   ├── features/       # Feature-based folders (e.g., auth, generator, profile)
│   │   ├── data/       # Repositories & DTOs
│   │   ├── domain/     # Entities & Use cases
│   │   └── presentation/ # Widgets & Riverpod providers
│   └── app.dart        # Root widget (MaterialApp with Riverpod)
└── main.dart           # Entry point
```

## 7. Testing & Quality
- **Linter:** Use `flutter_lints` with strict rules enabled.
- **Widget Tests:** Test core UI components (e.g., Input Mask, Theme Toggle).
- **Golden Tests:** Ensure the "Woman-Look" UI remains visually consistent across updates.
- **Mocking:** Use `mocktail` to mock repositories and API clients during testing.

## 8. Localization & GDPR
- **i18n:** Use `flutter_localizations` (supporting German and English).
- **Privacy:** Implement a clear consent dialog for Geo-location and Data processing according to GDPR.