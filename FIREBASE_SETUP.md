# RideON Database Schema

Complete Firestore database structure for the RideON application.

## Collections Overview

The RideON app uses three main Firestore collections:
1. **users** - User profiles and authentication data
2. **vehicles** - Available cycles and e-scooters
3. **rides** - Active and completed ride records

---

## 1. Users Collection

**Collection Path:** `/users/{userId}`

### Document Structure

```javascript
{
  // User ID from Firebase Auth
  userId: "string (auto from Auth)",

  // Personal Information
  name: "string",
  email: "string",
  phone: "string",
  collegeId: "string | null", // Optional college verification

  // Account Status
  activeRide: boolean, // true if user has an active ride
  subscriptionStatus: "none" | "active" | "expired",

  // Metadata
  createdAt: timestamp,
  updatedAt: timestamp
}
```

### Example Document

```json
{
  "userId": "abc123xyz",
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+91-9876543210",
  "collegeId": "CS2023001",
  "activeRide": false,
  "subscriptionStatus": "none",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}
```

### Indexes Required

- `email` (Ascending)
- `phone` (Ascending)
- `activeRide` (Ascending)

### Security Rules

```javascript
match /users/{userId} {
  // Users can only read/write their own data
  allow read, write: if request.auth != null && request.auth.uid == userId;

  // Admins can read all users (add admin check in production)
  allow read: if request.auth != null;
}
```

---

## 2. Vehicles Collection

**Collection Path:** `/vehicles/{vehicleId}`

### Document Structure

```javascript
{
  // Vehicle Identification
  vehicleId: "string (auto-generated)",
  type: "cycle" | "escooter",

  // Status
  status: "available" | "in-use" | "maintenance",

  // Location
  location: {
    latitude: number,
    longitude: number
  },

  // Vehicle Details
  batteryLevel: number, // 0-100, only for e-scooters
  pricePerHour: number, // in rupees

  // QR Code
  qrCode: "string", // Unique QR code identifier

  // Metadata
  createdAt: timestamp,
  updatedAt: timestamp
}
```

### Example Documents

**Cycle:**
```json
{
  "vehicleId": "cycle_001",
  "type": "cycle",
  "status": "available",
  "location": {
    "latitude": 28.6139,
    "longitude": 77.2090
  },
  "pricePerHour": 20,
  "qrCode": "RIDEON_CYCLE_001",
  "createdAt": "2024-01-10T09:00:00Z",
  "updatedAt": "2024-01-15T14:22:00Z"
}
```

**E-Scooter:**
```json
{
  "vehicleId": "escooter_001",
  "type": "escooter",
  "status": "in-use",
  "location": {
    "latitude": 28.6129,
    "longitude": 77.2095
  },
  "batteryLevel": 75,
  "pricePerHour": 30,
  "qrCode": "RIDEON_SCOOTER_001",
  "createdAt": "2024-01-10T09:00:00Z",
  "updatedAt": "2024-01-15T15:45:00Z"
}
```

### Indexes Required

- `status` (Ascending)
- `type` (Ascending)
- Composite: `status` (Ascending) + `type` (Ascending)

### Security Rules

```javascript
match /vehicles/{vehicleId} {
  // All authenticated users can read
  allow read: if request.auth != null;

  // Only authenticated users can write (add admin check in production)
  allow create, update, delete: if request.auth != null;
}
```

---

## 3. Rides Collection

**Collection Path:** `/rides/{rideId}`

### Document Structure

```javascript
{
  // Ride Identification
  rideId: "string (auto-generated)",

  // References
  userId: "string (ref to users/{userId})",
  vehicleId: "string (ref to vehicles/{vehicleId})",

  // Timing
  startTime: timestamp,
  endTime: timestamp | null, // null if ride is active

  // Pricing
  totalCost: number, // in rupees

  // Status
  status: "active" | "completed" | "cancelled",

  // Location History (optional)
  startLocation: {
    latitude: number,
    longitude: number
  },
  endLocation: {
    latitude: number,
    longitude: number
  } | null,

  // Metadata
  createdAt: timestamp
}
```

### Example Documents

**Active Ride:**
```json
{
  "rideId": "ride_20240115_001",
  "userId": "abc123xyz",
  "vehicleId": "escooter_001",
  "startTime": "2024-01-15T15:30:00Z",
  "endTime": null,
  "totalCost": 0,
  "status": "active",
  "startLocation": {
    "latitude": 28.6139,
    "longitude": 77.2090
  },
  "endLocation": null,
  "createdAt": "2024-01-15T15:30:00Z"
}
```

**Completed Ride:**
```json
{
  "rideId": "ride_20240115_002",
  "userId": "abc123xyz",
  "vehicleId": "cycle_001",
  "startTime": "2024-01-15T14:00:00Z",
  "endTime": "2024-01-15T14:45:00Z",
  "totalCost": 15,
  "status": "completed",
  "startLocation": {
    "latitude": 28.6139,
    "longitude": 77.2090
  },
  "endLocation": {
    "latitude": 28.6155,
    "longitude": 77.2100
  },
  "createdAt": "2024-01-15T14:00:00Z"
}
```

### Indexes Required

- `userId` (Ascending)
- `vehicleId` (Ascending)
- `status` (Ascending)
- Composite: `userId` (Ascending) + `status` (Ascending)
- Composite: `userId` (Ascending) + `startTime` (Descending)

### Security Rules

```javascript
match /rides/{rideId} {
  // Users can read all rides (for admin purposes)
  allow read: if request.auth != null;

  // Users can create new rides
  allow create: if request.auth != null;

  // Users can only update their own rides
  allow update: if request.auth != null &&
    (resource.data.userId == request.auth.uid ||
     !exists(/databases/$(database)/documents/rides/$(rideId)));

  // No deletion allowed
  allow delete: if false;
}
```

---

## Relationships

### User → Rides (One-to-Many)
```
users/{userId} → rides (where userId == users.userId)
```

### Vehicle → Rides (One-to-Many)
```
vehicles/{vehicleId} → rides (where vehicleId == vehicles.vehicleId)
```

### Active Ride Check
```
users/{userId}.activeRide == true
  → exists active ride in rides collection where userId matches
```

---

## Query Examples

### Get All Available Vehicles
```javascript
const vehiclesRef = collection(db, 'vehicles');
const q = query(vehiclesRef, where('status', '==', 'available'));
const snapshot = await getDocs(q);
```

### Get User's Ride History
```javascript
const ridesRef = collection(db, 'rides');
const q = query(
  ridesRef,
  where('userId', '==', currentUserId),
  orderBy('startTime', 'desc')
);
const snapshot = await getDocs(q);
```

### Get Active Ride for User
```javascript
const ridesRef = collection(db, 'rides');
const q = query(
  ridesRef,
  where('userId', '==', currentUserId),
  where('status', '==', 'active')
);
const snapshot = await getDocs(q);
```

### Get All Active Rides (Admin)
```javascript
const ridesRef = collection(db, 'rides');
const q = query(ridesRef, where('status', '==', 'active'));
const snapshot = await getDocs(q);
```

---

## Data Validation Rules

### Users
- `name`: Required, string, 2-50 characters
- `email`: Required, valid email format
- `phone`: Required, valid phone format
- `activeRide`: Required, boolean
- `subscriptionStatus`: Required, enum

### Vehicles
- `type`: Required, must be "cycle" or "escooter"
- `status`: Required, must be "available", "in-use", or "maintenance"
- `location.latitude`: Required, number, -90 to 90
- `location.longitude`: Required, number, -180 to 180
- `batteryLevel`: Required for escooters, 0-100
- `pricePerHour`: Required, positive number

### Rides
- `userId`: Required, valid user reference
- `vehicleId`: Required, valid vehicle reference
- `startTime`: Required, timestamp
- `status`: Required, must be "active", "completed", or "cancelled"
- `totalCost`: Required, non-negative number

---

## Backup and Migration

### Export Data
```bash
gcloud firestore export gs://[BUCKET_NAME]
```

### Import Data
```bash
gcloud firestore import gs://[BUCKET_NAME]/[EXPORT_FOLDER]
```

---

## Performance Optimization

1. **Compound Indexes**: Create for frequently used query combinations
2. **Pagination**: Use `limit()` and `startAfter()` for large collections
3. **Caching**: Cache static data like vehicle locations
4. **Real-time Listeners**: Use only for active rides, not history

---

## Future Enhancements

- **Payments Collection**: Store payment transactions
- **Subscriptions Collection**: Manage monthly plans
- **Reviews Collection**: User ratings and feedback
- **Maintenance Collection**: Vehicle maintenance logs
- **Analytics Collection**: Usage statistics

---

This schema provides a solid foundation for the RideON app and can be extended based on future requirements.
