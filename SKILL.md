# AttendQR Flutter - Skills & Capabilities Reference

## Feature Skills Matrix

### 1. QR Code Management
- **Generate QR Codes**: Time-limited QR sessions with HMAC-SHA256 signing
- **Scan QR Codes**: Camera-based scanning using `mobile_scanner`
- **Validate Sessions**: Server-side session validation with expiry check
- **Auto-refresh**: Countdown timer with automatic QR rotation
- **Offline QR**: Client-side generation for demo/testing

### 2. Attendance Tracking
- **Mark Attendance**: GPS + QR + device fingerprint triple validation
- **View History**: Per-student attendance records with date filtering
- **Daily Stats**: Present, absent, late, WFH counts with trend indicators
- **Weekly Trends**: Bar chart visualization of attendance patterns
- **Percentage Calculation**: Auto-computed attendance percentage
- **Export**: Attendance data export capability

### 3. Geofencing & Location
- **GPS Location**: High-accuracy position capture with permission handling
- **Haversine Distance**: Server-validated distance calculation
- **Geofence Radius**: Configurable allowed radius (default: 100m)
- **Live Map View**: Visual geofence with connected device overlay
- **Rotation Interval**: Configurable QR/geo rotation timing
- **Validation Rules**: Toggle QR check, geofence, RFID, late marking

### 4. Device Fingerprinting
- **Hardware Traits**: Model, brand, manufacturer, device info
- **Software Traits**: OS version, system name, identifiers
- **Daily Rotation**: Fingerprint includes date for daily uniqueness
- **SHA-256 Hashing**: Secure fingerprint generation
- **Caching**: Per-day fingerprint cache in SharedPreferences

### 5. Admin Dashboard
- **Real-time Stats**: Live attendance summary cards
- **Attendance Log**: Today's check-in table with status badges
- **Live QR Display**: Admin-facing QR code with countdown
- **Search**: Employee search functionality
- **Notifications**: Bell icon with unread indicator

### 6. Payroll Integrations
- **Provider Cards**: DevMode, Slack HR, Zoho, Paychex
- **API Configuration**: Endpoint, key, headers, data mapping
- **Webhook Events**: Configurable event triggers
- **Connection Testing**: API health check
- **Status Tracking**: Connected/Disconnected/Pending/Error states

### 7. White Labeling
- **Brand Configuration**: Company name, colors, domain, email
- **Color Palette**: Visual color picker with 6 preset options
- **Logo Upload**: File upload support for branding
- **Live Preview**: Real-time mobile app preview with brand changes
- **Store Deployment**: App Store / Play Store buttons

### 8. Database Configuration
- **Multi-DB Support**: MongoDB and PostgreSQL toggle
- **Connection Settings**: Host, port, database, credentials
- **Connection Testing**: Test and connect/disconnect buttons
- **Schema Viewer**: Expandable collection/table browser
- **Document Counts**: Per-collection record counts

### 9. System Settings
- **General**: Language, timezone, date/time format, dark mode
- **Attendance Rules**: Late threshold, work hours, working days
- **Security**: 2FA toggle, session timeout, API key regeneration
- **Notifications**: Push notifications, email reports toggles

## Technical Skills

### State Management Pattern
```dart
// Provider pattern: ChangeNotifier + Consumer
class MyProvider extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
  void increment() { _count++; notifyListeners(); }
}

// In widget:
Consumer<MyProvider>(
  builder: (context, provider, _) => Text('${provider.count}'),
)
```

### API Integration Pattern
```dart
// All API calls go through ApiService
final response = await ApiService.get('/api/endpoint', queryParams: {...});
if (response.success) {
  // Handle response.data
} else {
  // Handle response.error
}
```

### Responsive Layout Pattern
```dart
// Width-based layout switching
final isWide = MediaQuery.of(context).size.width > 1000;
if (isWide)
  Row(children: [Expanded(child: Left()), Expanded(child: Right())])
else
  Column(children: [Left(), Right()])
```

### Navigation Pattern
```dart
// GoRouter with ShellRoute for persistent sidebar
context.go('/dashboard');   // Navigate
context.push('/qr-scanner'); // Push on stack
```

## Platform-Specific Skills

### iOS
- Camera permission for QR scanning (NSCameraUsageDescription)
- Location permission (NSLocationWhenInUseUsageDescription)
- Secure storage via Keychain

### Android
- Camera permission (android.permission.CAMERA)
- Fine location permission (android.permission.ACCESS_FINE_LOCATION)
- Internet permission (android.permission.INTERNET)
- Secure storage via EncryptedSharedPreferences

## Deployment Skills
- Android: APK and App Bundle generation
- iOS: Archive and IPA export
- CI/CD: GitHub Actions compatible
- Environment configs: Dev, staging, production API URLs
