# AttendQR Flutter - Claude Code Project Guide

## Project Overview

AttendQR Enterprise is a Flutter-based QR attendance management system with geofencing, payroll integrations, white-label support, and a comprehensive admin dashboard. It connects to a Node.js/Express + MongoDB backend.

## Architecture

```
lib/
├── main.dart              # App entry point, provider setup
├── app.dart               # MaterialApp.router configuration
├── config/
│   ├── theme.dart         # Dark theme, AppColors, AppTheme
│   ├── constants.dart     # API endpoints, defaults, strings
│   └── routes.dart        # GoRouter configuration
├── models/                # Data classes with JSON serialization
│   ├── user_model.dart
│   ├── attendance_model.dart
│   ├── employee_model.dart
│   ├── geo_fence_model.dart
│   ├── payroll_integration_model.dart
│   ├── white_label_config_model.dart
│   └── database_config_model.dart
├── services/              # Business logic, API calls, device APIs
│   ├── api_service.dart           # HTTP client wrapping all endpoints
│   ├── auth_service.dart          # Login/logout, token management
│   ├── location_service.dart      # GPS, Haversine, geofence check
│   ├── qr_service.dart            # QR generation, HMAC-SHA256 signing
│   ├── device_fingerprint_service.dart  # Hardware fingerprinting
│   └── storage_service.dart       # SharedPreferences wrapper
├── providers/             # ChangeNotifier state management
│   ├── auth_provider.dart
│   ├── dashboard_provider.dart
│   ├── attendance_provider.dart
│   ├── geo_tag_provider.dart
│   ├── settings_provider.dart
│   ├── payroll_provider.dart
│   ├── white_label_provider.dart
│   └── database_config_provider.dart
├── screens/               # UI screens (each in its own folder)
│   ├── splash/            # Animated splash screen
│   ├── login/             # Admin login with form validation
│   ├── dashboard/         # Stats, chart, live QR, attendance log
│   │   └── widgets/       # StatsCard, WeeklyChart, LiveQRCard, LogTable
│   ├── mobile_app/        # QR scanner, attendance history, app preview
│   ├── geo_tagging/       # Geofence map, config, validation rules
│   ├── payroll/           # Integration cards, API/webhook config
│   ├── white_labeling/    # Brand config, color picker, app preview
│   ├── database/          # DB type toggle, connection settings, schemas
│   ├── settings/          # General, rules, security, notifications
│   └── main_shell.dart    # Sidebar + top bar shell layout
└── widgets/               # Shared reusable widgets
    ├── glass_card.dart    # Styled container card
    ├── status_badge.dart  # Present/Absent/Late/WFH badge
    └── app_sidebar.dart   # Navigation sidebar
```

## Tech Stack

- **Flutter** >= 3.2.0, Dart >= 3.2.0
- **State Management**: Provider (ChangeNotifier pattern)
- **Navigation**: GoRouter with ShellRoute for sidebar layout
- **HTTP**: `http` package for REST API calls
- **QR Code**: `qr_flutter` (display), `mobile_scanner` (camera scan)
- **Location**: `geolocator` for GPS, `google_maps_flutter` for maps
- **Charts**: `fl_chart` for bar charts
- **Storage**: `shared_preferences` + `flutter_secure_storage`
- **Crypto**: `crypto` package for HMAC-SHA256
- **Device Info**: `device_info_plus` for fingerprinting

## Coding Conventions

- **File naming**: snake_case for all files
- **Class naming**: PascalCase
- **Private widgets**: Prefix with underscore, use as nested classes in same file
- **State**: All mutable state lives in providers, screens are stateless where possible
- **Theme**: All colors from `AppColors`, no hardcoded colors in screens
- **Responsive**: Use `LayoutBuilder` / `MediaQuery` for adaptive layouts (800px breakpoint for sidebar, 1000px for two-column layouts)

## Key Commands

```bash
flutter pub get          # Install dependencies
flutter run              # Run on connected device/simulator
flutter build apk        # Build Android APK
flutter build ios        # Build iOS (requires Mac + Xcode)
flutter build appbundle  # Build Android App Bundle for Play Store
flutter test             # Run tests
flutter analyze          # Static analysis
```

## Backend API

The app connects to a Node.js/Express backend at `http://localhost:5000`:
- `GET  /api/generate-qr` — Generate time-limited QR session
- `POST /api/validate-session` — Validate QR session ID
- `POST /mark-attendance` — Submit attendance with location + fingerprint
- `GET  /api/attendance?rollNo=X` — Fetch attendance records
- `GET  /api/attendance/dates` — All class dates
- `GET  /api/attendance/by-date?date=YYYY-MM-DD` — Attendance by date
- `GET  /api/students/profile?rollNo=X` — Student profile

## Demo Credentials

- Username: `admin`
- Password: `admin123`

## Notes for Claude

- The app currently includes sample/demo data in providers for offline development
- `ApiService.setBaseUrl()` must be called to point to the production backend
- QR codes are generated client-side using HMAC-SHA256 for demo; production should use server-generated QR
- Device fingerprinting uses `device_info_plus` traits hashed with SHA-256
- The geofence visualization uses `CustomPainter`, not Google Maps (for simplicity)
- All screens are responsive with mobile + tablet + desktop breakpoints
