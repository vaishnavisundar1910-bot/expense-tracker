# RideON - Smart Micro-Rental Mobile App

A modern QR-based rental system for cycles and e-scooters built with React Native and Firebase.

## 🚀 Features

### User Features
- **Authentication**: Secure login/signup with email and college ID verification
- **Home Screen**: Interactive map showing nearby available vehicles
- **QR Scanner**: Scan QR codes to unlock vehicles instantly
- **Ride Tracking**: Real-time ride duration, cost estimation, and GPS tracking
- **Payment System**: Multiple payment options (UPI, Card, Wallet)
- **Ride History**: Complete history with billing details
- **User Profile**: Manage profile, subscription, and wallet

### Admin Features
- **Vehicle Management**: Add, remove, and monitor vehicles
- **Active Ride Monitoring**: Track all active rides in real-time
- **Revenue Dashboard**: View total revenue and statistics
- **Location Tracking**: Monitor vehicle locations on map

## 🎨 Design

- **Modern & Minimal UI**: Clean and intuitive interface
- **Color Palette**: Blue (#2563eb) and Green (#10b981)
- **Student-Friendly**: Optimized for college students
- **Professional**: Enterprise-grade design

## 📱 Tech Stack

- **Frontend**: React Native with Expo
- **Navigation**: React Navigation (Stack & Bottom Tabs)
- **Backend**: Firebase (Authentication, Firestore, Storage)
- **Maps**: React Native Maps
- **QR Scanner**: Expo Barcode Scanner
- **Location**: Expo Location

## 📦 Installation

### Prerequisites
- Node.js (v14 or higher)
- npm or yarn
- Expo CLI
- Firebase account

### Step 1: Clone and Install

```bash
cd rideon-app
npm install
```

### Step 2: Firebase Setup

1. Create a new Firebase project at [Firebase Console](https://console.firebase.google.com/)

2. Enable Authentication:
   - Go to Authentication > Sign-in method
   - Enable Email/Password authentication

3. Create Firestore Database:
   - Go to Firestore Database
   - Create database in production mode
   - Add the following collections:

#### Users Collection
```javascript
{
  userId: "string",
  name: "string",
  email: "string",
  phone: "string",
  collegeId: "string" (optional),
  activeRide: boolean,
  subscriptionStatus: "none" | "active",
  createdAt: timestamp
}
```

#### Vehicles Collection
```javascript
{
  vehicleId: "auto-generated",
  type: "cycle" | "escooter",
  status: "available" | "in-use",
  location: {
    latitude: number,
    longitude: number
  },
  batteryLevel: number (for e-scooters),
  pricePerHour: number,
  createdAt: timestamp
}
```

#### Rides Collection
```javascript
{
  rideId: "auto-generated",
  userId: "string",
  vehicleId: "string",
  startTime: timestamp,
  endTime: timestamp,
  totalCost: number,
  status: "active" | "completed"
}
```

4. Update Firebase Configuration:
   - Open `src/config/firebase.js`
   - Replace the placeholder config with your Firebase config:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

### Step 3: Firestore Security Rules

Add these security rules in Firestore:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    match /vehicles/{vehicleId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null; // Add admin check in production
    }

    match /rides/{rideId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update: if request.auth != null &&
        resource.data.userId == request.auth.uid;
    }
  }
}
```

### Step 4: Run the App

```bash
# Start Expo development server
npm start

# Run on iOS
npm run ios

# Run on Android
npm run android

# Run on web
npm run web
```

## 📱 Usage

### For Users

1. **Sign Up**: Create an account with email and college ID
2. **Find Vehicle**: View nearby vehicles on the map
3. **Scan QR**: Tap on a vehicle and scan its QR code
4. **Start Ride**: Vehicle unlocks automatically
5. **Track Ride**: Monitor duration and cost in real-time
6. **End Ride**: Tap "End Ride" button when done
7. **Pay**: Choose payment method and complete payment

### For Admins

1. Access admin panel from profile menu
2. Add vehicles with location and pricing
3. Monitor active rides
4. View revenue statistics
5. Manage vehicle inventory

## 🔧 Configuration

### Pricing
Update pricing in `src/screens/RideTrackingScreen.js`:
```javascript
const PRICE_PER_MINUTE = 2; // Change this value
```

### Default Location
Update default map location in vehicle creation:
```javascript
latitude: 28.6139,  // Delhi coordinates
longitude: 77.2090
```

## 📂 Project Structure

```
rideon-app/
├── App.js                      # Main app entry with navigation
├── src/
│   ├── config/
│   │   ├── firebase.js         # Firebase configuration
│   │   └── theme.js            # App theme and styling
│   ├── screens/
│   │   ├── LoginScreen.js      # Login screen
│   │   ├── SignupScreen.js     # Signup screen
│   │   ├── HomeScreen.js       # Map and vehicle listing
│   │   ├── ScannerScreen.js    # QR code scanner
│   │   ├── RideTrackingScreen.js   # Active ride tracking
│   │   ├── PaymentSummaryScreen.js # Payment page
│   │   ├── RidesHistoryScreen.js   # Ride history
│   │   ├── ProfileScreen.js    # User profile
│   │   └── AdminScreen.js      # Admin panel
│   └── utils/
│       └── firebaseUtils.js    # Firebase helper functions
├── package.json
├── app.json
└── README.md
```

## 🎯 Key Components

### Authentication Flow
- Email/password authentication
- User profile creation in Firestore
- Persistent login state

### QR Code System
- Barcode scanner for QR codes
- Vehicle verification
- Automatic ride creation

### Real-time Tracking
- GPS location tracking
- Live ride duration counter
- Cost calculation
- Emergency SOS button

### Payment Integration
- Mock payment methods (UPI, Card, Wallet)
- Payment summary screen
- Ride completion workflow

## 🔐 Security Notes

1. Never commit Firebase config with real credentials
2. Implement proper Firestore security rules
3. Add server-side validation for payments
4. Implement rate limiting
5. Add admin role verification

## 🚀 Production Deployment

### iOS
1. Update `ios.bundleIdentifier` in `app.json`
2. Build with EAS: `eas build --platform ios`
3. Submit to App Store

### Android
1. Update `android.package` in `app.json`
2. Build with EAS: `eas build --platform android`
3. Submit to Play Store

## 📝 Future Enhancements

- [ ] Real payment gateway integration
- [ ] Push notifications for ride updates
- [ ] Ride sharing feature
- [ ] Referral system
- [ ] Monthly subscription plans
- [ ] Vehicle maintenance tracking
- [ ] User ratings and reviews
- [ ] IoT integration for actual vehicle control
- [ ] Analytics dashboard
- [ ] Multi-language support

## 🐛 Troubleshooting

### Firebase Connection Issues
- Verify Firebase config is correct
- Check internet connectivity
- Ensure Firestore is in production mode

### Maps Not Loading
- Check location permissions
- Verify Google Maps API key (for production)

### QR Scanner Not Working
- Grant camera permissions
- Test on physical device (not simulator)

## 📄 License

This project is created for educational purposes.

## 👥 Support

For issues and questions:
- Email: support@rideon.com
- GitHub Issues: [Create an issue](https://github.com/yourusername/rideon-app/issues)

## 🎉 Acknowledgments

Built with React Native, Expo, and Firebase for modern mobility solutions.

---

**RideON** - Smart Mobility Solution for Students
