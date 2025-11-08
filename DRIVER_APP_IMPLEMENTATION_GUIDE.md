# 🚗 Al Marya Driver App - Complete Implementation Guide

## ✅ What's Already Done

### Backend (100% Complete)
- ✅ Driver Model (`backend/models/Driver.js`)
- ✅ Driver Routes (`backend/routes/driver.js`)
- ✅ Order Model with driver assignment
- ✅ Staff "Hand to Driver" functionality
- ✅ Push notification service

### Driver App Project
- ✅ Flutter project created (`al_marya_driver_app/`)
- ✅ Dependencies added to pubspec.yaml:
  - provider, http, shared_preferences
  - geolocator, geocoding, permission_handler
  - google_maps_flutter, url_launcher
  - mobile_scanner (QR), flutter_local_notifications

---

## 📋 Implementation Steps

### Step 1: Copy Core Files from Staff App

**Files to copy from `al_marya_staff_app/lib/core/` to `al_marya_driver_app/lib/core/`:**

```bash
mkdir -p al_marya_driver_app/lib/core/{theme,constants,services,utils}

# Copy theme
cp al_marya_staff_app/lib/core/theme/app_theme.dart al_marya_driver_app/lib/core/theme/

# Copy constants
cp al_marya_staff_app/lib/core/constants/app_constants.dart al_marya_driver_app/lib/core/constants/

# Copy services (auth_service, api_service)
cp al_marya_staff_app/lib/core/services/pin_auth_service.dart al_marya_driver_app/lib/core/services/
cp al_marya_staff_app/lib/core/services/orders_service.dart al_marya_driver_app/lib/core/services/

# Update service names:
# - pin_auth_service.dart → Change endpoint from /api/staff to /api/driver
# - orders_service.dart → Change endpoint from /api/staff/orders to /api/driver/orders
```

**Key Changes in Services:**
```dart
// In pin_auth_service.dart
- final apiUrl = '${AppConstants.apiBaseUrl}/api/staff/auth/pin-login';
+ final apiUrl = '${AppConstants.apiBaseUrl}/api/driver/auth/pin-login';

// In orders_service.dart
- final apiUrl = '${AppConstants.apiBaseUrl}/api/staff/orders';
+ final apiUrl = '${AppConstants.apiBaseUrl}/api/driver/orders';
```

---

### Step 2: Create Authentication Feature

#### **File: `lib/features/auth/screens/pin_login_screen.dart`**
(Copy from staff app, update theme colors to green for driver app)

#### **File: `lib/features/auth/screens/qr_scanner_screen.dart`**
(Copy from staff app)

---

### Step 3: Create Orders Feature

#### **File: `lib/features/orders/screens/deliveries_list_screen.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class DeliveriesListScreen extends StatefulWidget {
  @override
  State<DeliveriesListScreen> createState() => _DeliveriesListScreenState();
}

class _DeliveriesListScreenState extends State<DeliveriesListScreen> {
  String _filterStatus = 'available'; // available, assigned, completed
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Deliveries'),
        actions: [
          IconButton(
            icon: const Icon(Icons.account_circle),
            onPressed: () => Navigator.pushNamed(context, '/profile'),
          ),
        ],
      ),
      body: Column(
        children: [
          _buildFilterChips(),
          Expanded(child: _buildOrdersList()),
        ],
      ),
    );
  }
  
  Widget _buildFilterChips() {
    return Padding(
      padding: const EdgeInsets.all(16.0),
      child: Row(
        children: [
          _buildFilterChip('Available', 'available'),
          const SizedBox(width: 8),
          _buildFilterChip('My Deliveries', 'assigned'),
          const SizedBox(width: 8),
          _buildFilterChip('Completed', 'completed'),
        ],
      ),
    );
  }
  
  Widget _buildFilterChip(String label, String value) {
    final isSelected = _filterStatus == value;
    return FilterChip(
      label: Text(label),
      selected: isSelected,
      onSelected: (selected) {
        setState(() => _filterStatus = value);
      },
      backgroundColor: Colors.grey[200],
      selectedColor: Colors.green,
      labelStyle: TextStyle(
        color: isSelected ? Colors.white : Colors.black87,
      ),
    );
  }
  
  Widget _buildOrdersList() {
    // Fetch orders from API based on _filterStatus
    // Build list of order cards with:
    // - Order number
    // - Customer name & phone
    // - Delivery address
    // - Total amount
    // - Status badge
    // - Accept/Start/Complete button based on status
    return ListView.builder(
      itemCount: 0,
      itemBuilder: (context, index) {
        return Card(/* Order card */);
      },
    );
  }
}
```

#### **File: `lib/features/orders/screens/delivery_detail_screen.dart`**

```dart
class DeliveryDetailScreen extends StatelessWidget {
  final String orderId;
  
  const DeliveryDetailScreen({required this.orderId});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Delivery Details')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            _buildOrderInfo(),
            const SizedBox(height: 24),
            _buildCustomerInfo(),
            const SizedBox(height: 24),
            _buildDeliveryAddress(),
            const SizedBox(height: 24),
            _buildNavigationButton(),
            const SizedBox(height: 16),
            _buildActionButtons(),
          ],
        ),
      ),
    );
  }
  
  Widget _buildOrderInfo() {
    // Order number, items, total amount
    return Card(/* ... */);
  }
  
  Widget _buildCustomerInfo() {
    // Customer name, phone with call button
    return Card(
      child: ListTile(
        title: Text('Customer Name'),
        subtitle: Text('+971 50 123 4567'),
        trailing: IconButton(
          icon: const Icon(Icons.call, color: Colors.green),
          onPressed: () {
            // Launch phone dialer
            // launchUrl(Uri(scheme: 'tel', path: phoneNumber));
          },
        ),
      ),
    );
  }
  
  Widget _buildDeliveryAddress() {
    return Card(
      child: ListTile(
        leading: const Icon(Icons.location_on, color: Colors.red),
        title: const Text('Delivery Address'),
        subtitle: const Text('Street, Building, Area, City'),
      ),
    );
  }
  
  Widget _buildNavigationButton() {
    return ElevatedButton.icon(
      onPressed: () {
        // Open Google Maps with directions
        // launchUrl(Uri.parse('https://www.google.com/maps/dir/?api=1&destination=$lat,$lng'));
      },
      icon: const Icon(Icons.navigation),
      label: const Text('Start Navigation'),
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.blue,
        minimumSize: const Size(double.infinity, 50),
      ),
    );
  }
  
  Widget _buildActionButtons() {
    // Show different buttons based on order status:
    // - ready: "Accept Delivery" button
    // - out-for-delivery: "Mark as Delivered" button
    return ElevatedButton(
      onPressed: () {/* Accept or Complete */},
      child: const Text('Accept Delivery'),
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.green,
        minimumSize: const Size(double.infinity, 50),
      ),
    );
  }
}
```

---

### Step 4: Create GPS Tracking Service

#### **File: `lib/core/services/location_service.dart`**

```dart
import 'package:geolocator/geolocator.dart';
import 'package:permission_handler/permission_handler.dart';
import 'dart:async';

class LocationService {
  StreamSubscription<Position>? _positionStream;
  Position? _currentPosition;
  
  // Request location permissions
  Future<bool> requestPermissions() async {
    final status = await Permission.location.request();
    return status.isGranted;
  }
  
  // Get current location once
  Future<Position?> getCurrentLocation() async {
    try {
      final hasPermission = await requestPermissions();
      if (!hasPermission) return null;
      
      final position = await Geolocator.getCurrentPosition(
        locationSettings: const LocationSettings(
          accuracy: LocationAccuracy.high,
        ),
      );
      _currentPosition = position;
      return position;
    } catch (e) {
      print('Error getting location: $e');
      return null;
    }
  }
  
  // Start continuous location tracking
  void startTracking(Function(Position) onLocationUpdate) {
    _positionStream = Geolocator.getPositionStream(
      locationSettings: const LocationSettings(
        accuracy: LocationAccuracy.high,
        distanceFilter: 10, // Update every 10 meters
      ),
    ).listen((Position position) {
      _currentPosition = position;
      onLocationUpdate(position);
    });
  }
  
  // Stop tracking
  void stopTracking() {
    _positionStream?.cancel();
    _positionStream = null;
  }
  
  Position? get currentPosition => _currentPosition;
}
```

#### **Update location to backend every 30 seconds:**

```dart
// In delivery_detail_screen.dart
Timer? _locationUpdateTimer;

void _startLocationUpdates() {
  _locationUpdateTimer = Timer.periodic(Duration(seconds: 30), (timer) async {
    final position = await LocationService().getCurrentLocation();
    if (position != null) {
      // Send to backend
      await http.put(
        Uri.parse('${AppConstants.apiBaseUrl}/api/driver/location'),
        headers: {'Authorization': 'Bearer $token'},
        body: jsonEncode({
          'latitude': position.latitude,
          'longitude': position.longitude,
          'accuracy': position.accuracy,
        }),
      );
    }
  });
}

@override
void dispose() {
  _locationUpdateTimer?.cancel();
  LocationService().stopTracking();
  super.dispose();
}
```

---

### Step 5: Create Driver Dashboard

#### **File: `lib/features/dashboard/screens/driver_dashboard_screen.dart`**

```dart
class DriverDashboardScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Dashboard'),
        actions: [
          // Status toggle (Available/Offline)
          Switch(
            value: true, // isAvailable
            onChanged: (value) {
              // Update driver status
            },
          ),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16.0),
        children: [
          _buildStatsCard('Today\\'s Deliveries', '12'),
          _buildStatsCard('Total Earnings', 'AED 240'),
          _buildStatsCard('Average Time', '18 min'),
          _buildStatsCard('Rating', '4.8 ⭐'),
        ],
      ),
    );
  }
  
  Widget _buildStatsCard(String title, String value) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            Text(title, style: TextStyle(color: Colors.grey)),
            const SizedBox(height: 8),
            Text(value, style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
          ],
        ),
      ),
    );
  }
}
```

---

### Step 6: Implement Push Notifications

#### **File: `lib/core/services/fcm_service.dart`**
(Copy from customer app, update channel ID to 'delivery_updates')

---

### Step 7: Create Main App

#### **File: `lib/main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'features/auth/screens/pin_login_screen.dart';
import 'features/orders/screens/deliveries_list_screen.dart';
import 'core/theme/app_theme.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Al Marya Driver',
      theme: AppTheme.driverTheme, // Green theme for driver app
      home: PinLoginScreen(),
      routes: {
        '/deliveries': (context) => DeliveriesListScreen(),
        '/dashboard': (context) => DriverDashboardScreen(),
      },
    );
  }
}
```

---

## 🔔 Backend Endpoints for Driver App

### Authentication
```
POST /api/driver/auth/pin-login
Body: { "pin": "1234" }
Response: { "success": true, "token": "...", "driver": {...} }
```

### Orders
```
GET /api/driver/orders?status=available
Response: { "success": true, "orders": [...] }

POST /api/driver/orders/:id/accept
Response: { "success": true, "order": {...} }

PUT /api/driver/orders/:id/status
Body: { "status": "delivered" }
Response: { "success": true }
```

### Location
```
PUT /api/driver/location
Body: { "latitude": 25.1234, "longitude": 55.5678, "accuracy": 10 }
Response: { "success": true }
```

---

## 🎨 UI Design Guidelines

### Colors (Driver App)
```dart
Primary: Colors.green (delivery/success theme)
Accent: Colors.blue (navigation)
Background: Colors.white
Surface: Colors.grey[100]
```

### Order Status Colors
```dart
ready: Colors.orange (pending pickup)
out-for-delivery: Colors.blue (in progress)
delivered: Colors.green (completed)
```

---

## 📱 Required Permissions

### iOS (`ios/Runner/Info.plist`)
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>We need your location to track deliveries</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>We need your location to track deliveries even when app is in background</string>
<key>NSCameraUsageDescription</key>
<string>We need camera access to scan QR codes</string>
```

### Android (`android/app/src/main/AndroidManifest.xml`)
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.CAMERA" />
```

---

## 🧪 Testing Flow

1. **Login as Driver** → PIN/QR authentication
2. **View Available Deliveries** → See orders marked "ready" by staff
3. **Accept Delivery** → Order status changes to "out-for-delivery"
4. **Start Navigation** → Opens Google Maps with customer address
5. **Location Tracking** → Backend receives driver's live location
6. **Mark as Delivered** → Order status changes to "delivered"
7. **Customer Notification** → Customer receives "Order Delivered" push notification

---

## ✅ Completion Checklist

- [ ] Install dependencies (`flutter pub get`)
- [ ] Copy core files from staff app
- [ ] Create authentication screens (PIN/QR)
- [ ] Build deliveries list screen with filters
- [ ] Implement delivery detail screen with navigation
- [ ] Add GPS tracking service
- [ ] Create location update background service
- [ ] Build driver dashboard with stats
- [ ] Add push notifications (FCM)
- [ ] Configure iOS/Android permissions
- [ ] Test complete delivery flow
- [ ] Update staff app notification when driver accepts

---

**🎉 Your driver app ecosystem is complete when you can:**
Customer orders → Staff prepares → Driver receives notification → Driver accepts → Tracks with GPS → Delivers → Customer notified! 🚗☕
