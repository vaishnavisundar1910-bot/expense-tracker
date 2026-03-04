import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  ActivityIndicator,
  Alert,
} from 'react-native';
import MapView, { Marker } from 'react-native-maps';
import * as Location from 'expo-location';
import { getAvailableVehicles } from '../utils/firebaseUtils';
import { theme } from '../config/theme';

const HomeScreen = ({ navigation }) => {
  const [location, setLocation] = useState(null);
  const [vehicles, setVehicles] = useState([]);
  const [loading, setLoading] = useState(true);
  const [selectedVehicle, setSelectedVehicle] = useState(null);

  useEffect(() => {
    getLocationPermission();
    loadVehicles();
  }, []);

  const getLocationPermission = async () => {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== 'granted') {
      Alert.alert('Permission Denied', 'Location permission is required to find nearby vehicles');
      return;
    }

    const currentLocation = await Location.getCurrentPositionAsync({});
    setLocation({
      latitude: currentLocation.coords.latitude,
      longitude: currentLocation.coords.longitude,
      latitudeDelta: 0.01,
      longitudeDelta: 0.01,
    });
  };

  const loadVehicles = async () => {
    setLoading(true);
    const result = await getAvailableVehicles();
    if (result.success) {
      setVehicles(result.data);
    } else {
      Alert.alert('Error', 'Failed to load vehicles');
    }
    setLoading(false);
  };

  const getVehicleIcon = (type) => {
    return type === 'cycle' ? '🚴' : '🛴';
  };

  return (
    <View style={styles.container}>
      {loading ? (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color={theme.colors.primary} />
        </View>
      ) : (
        <>
          <MapView
            style={styles.map}
            region={location}
            showsUserLocation
            showsMyLocationButton
          >
            {vehicles.map((vehicle) => (
              <Marker
                key={vehicle.id}
                coordinate={{
                  latitude: vehicle.location.latitude,
                  longitude: vehicle.location.longitude,
                }}
                onPress={() => setSelectedVehicle(vehicle)}
              >
                <View style={styles.markerContainer}>
                  <Text style={styles.markerIcon}>{getVehicleIcon(vehicle.type)}</Text>
                </View>
              </Marker>
            ))}
          </MapView>

          <View style={styles.header}>
            <Text style={styles.headerTitle}>Find Your Ride</Text>
            <Text style={styles.headerSubtitle}>
              {vehicles.length} vehicles nearby
            </Text>
          </View>

          {selectedVehicle && (
            <View style={styles.vehicleCard}>
              <View style={styles.vehicleHeader}>
                <Text style={styles.vehicleIcon}>
                  {getVehicleIcon(selectedVehicle.type)}
                </Text>
                <View style={styles.vehicleInfo}>
                  <Text style={styles.vehicleType}>
                    {selectedVehicle.type === 'cycle' ? 'Cycle' : 'E-Scooter'}
                  </Text>
                  <Text style={styles.vehicleId}>ID: {selectedVehicle.id.slice(0, 8)}</Text>
                </View>
                <TouchableOpacity onPress={() => setSelectedVehicle(null)}>
                  <Text style={styles.closeButton}>✕</Text>
                </TouchableOpacity>
              </View>

              <View style={styles.vehicleDetails}>
                {selectedVehicle.type === 'escooter' && (
                  <View style={styles.detailRow}>
                    <Text style={styles.detailLabel}>Battery:</Text>
                    <Text style={styles.detailValue}>{selectedVehicle.batteryLevel}%</Text>
                  </View>
                )}
                <View style={styles.detailRow}>
                  <Text style={styles.detailLabel}>Price:</Text>
                  <Text style={styles.detailValue}>₹{selectedVehicle.pricePerHour}/hr</Text>
                </View>
              </View>

              <TouchableOpacity
                style={styles.unlockButton}
                onPress={() => navigation.navigate('Scanner', { vehicleId: selectedVehicle.id })}
              >
                <Text style={styles.unlockButtonText}>Scan QR to Unlock</Text>
              </TouchableOpacity>
            </View>
          )}

          <TouchableOpacity
            style={styles.refreshButton}
            onPress={loadVehicles}
          >
            <Text style={styles.refreshButtonText}>🔄</Text>
          </TouchableOpacity>
        </>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: theme.colors.background,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  map: {
    flex: 1,
  },
  header: {
    position: 'absolute',
    top: 50,
    left: theme.spacing.md,
    right: theme.spacing.md,
    backgroundColor: theme.colors.card,
    borderRadius: theme.borderRadius.lg,
    padding: theme.spacing.md,
    ...theme.shadows.medium,
  },
  headerTitle: {
    fontSize: theme.fontSize.lg,
    fontWeight: theme.fontWeight.bold,
    color: theme.colors.text,
  },
  headerSubtitle: {
    fontSize: theme.fontSize.sm,
    color: theme.colors.textSecondary,
    marginTop: theme.spacing.xs,
  },
  markerContainer: {
    backgroundColor: theme.colors.card,
    borderRadius: 20,
    padding: 8,
    borderWidth: 2,
    borderColor: theme.colors.primary,
  },
  markerIcon: {
    fontSize: 24,
  },
  vehicleCard: {
    position: 'absolute',
    bottom: 20,
    left: theme.spacing.md,
    right: theme.spacing.md,
    backgroundColor: theme.colors.card,
    borderRadius: theme.borderRadius.lg,
    padding: theme.spacing.lg,
    ...theme.shadows.medium,
  },
  vehicleHeader: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: theme.spacing.md,
  },
  vehicleIcon: {
    fontSize: 40,
    marginRight: theme.spacing.md,
  },
  vehicleInfo: {
    flex: 1,
  },
  vehicleType: {
    fontSize: theme.fontSize.lg,
    fontWeight: theme.fontWeight.bold,
    color: theme.colors.text,
  },
  vehicleId: {
    fontSize: theme.fontSize.sm,
    color: theme.colors.textSecondary,
    marginTop: theme.spacing.xs,
  },
  closeButton: {
    fontSize: 24,
    color: theme.colors.textSecondary,
    padding: theme.spacing.xs,
  },
  vehicleDetails: {
    marginBottom: theme.spacing.md,
  },
  detailRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: theme.spacing.sm,
  },
  detailLabel: {
    fontSize: theme.fontSize.md,
    color: theme.colors.textSecondary,
  },
  detailValue: {
    fontSize: theme.fontSize.md,
    fontWeight: theme.fontWeight.semibold,
    color: theme.colors.text,
  },
  unlockButton: {
    backgroundColor: theme.colors.primary,
    borderRadius: theme.borderRadius.sm,
    padding: theme.spacing.md,
    alignItems: 'center',
  },
  unlockButtonText: {
    color: '#ffffff',
    fontSize: theme.fontSize.md,
    fontWeight: theme.fontWeight.semibold,
  },
  refreshButton: {
    position: 'absolute',
    top: 130,
    right: theme.spacing.md,
    backgroundColor: theme.colors.card,
    borderRadius: 30,
    width: 50,
    height: 50,
    justifyContent: 'center',
    alignItems: 'center',
    ...theme.shadows.medium,
  },
  refreshButtonText: {
    fontSize: 24,
  },
});

export default HomeScreen;
