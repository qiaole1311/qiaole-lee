# Cambridge Parking Finder - Setup Guide

## Overview
A web application that helps residents and visitors of Cambridge, MA find available parking within a specified radius of any searched address.

## Features
- Interactive map interface (using Leaflet.js)
- Address search with geocoding
- Radius-based filtering (50m, 100m, 200m, 500m, 1km)
- Different parking types: Free, Metered, Paid lots/garages, Permit zones
- Detailed parking information: cost, time limits, restrictions
- Color-coded markers for easy identification
- Responsive design

## How to Use
1. Open `cambridge-parking-finder.html` in a web browser
2. Enter an address in the search box (e.g., "2 Peabody Terrace, Cambridge, MA")
3. Select your desired search radius
4. Click "Search" or press Enter
5. View parking spots within the radius on the map
6. Click on markers to see detailed parking information

## Current Implementation
The app currently uses **sample data** with 8 example parking locations around Cambridge. This demonstrates the functionality and user interface.

## Integrating Real Cambridge Parking Data

### Available Data Sources

#### 1. Cambridge Open Data Portal
**URL**: https://data.cambridgema.gov

**Available Datasets**:
- Parking meters and metered parking zones
- Parking lots and garages
- Visitor permit boundaries
- Street occupancy permits
- Special event parking restrictions

**How to Access**:
```javascript
// Example: Fetch parking data from Cambridge Open Data (Socrata API)
async function fetchCambridgeParkingData() {
    const endpoint = 'https://data.cambridgema.gov/resource/[DATASET_ID].json';
    const response = await fetch(endpoint);
    const data = await response.json();
    return data;
}
```

#### 2. Cambridge GIS Maps
**URL**: https://www.cambridgema.gov/GIS

**Data Available**:
- Parking meters locations
- Public parking lots and garages
- Parking zones
- Downloadable GIS datasets

**Contact for API Access**:
- Phone: 617-349-4140
- Email: OpenData@CambridgeMA.GOV

#### 3. ParkMobile API (For Real-Time Availability)
Cambridge uses ParkMobile zones (1555-6981) for parking payments.

**Potential Integration**:
- Real-time parking availability
- Payment integration
- Zone-based pricing

### Step-by-Step Integration Guide

#### Step 1: Obtain API Access
1. Visit https://data.cambridgema.gov
2. Browse available parking datasets
3. Note the dataset IDs (usually in format: `xxxx-xxxx`)
4. Some APIs may require an app token (free registration)

#### Step 2: Update the Data Structure
Replace the `sampleParkingData` array in the HTML file with real data:

```javascript
// Example data structure to match with API response
const parkingData = apiResponse.map(spot => ({
    id: spot.id || spot.objectid,
    name: spot.location_name || spot.street_name,
    lat: parseFloat(spot.latitude || spot.lat),
    lng: parseFloat(spot.longitude || spot.lng),
    type: determineType(spot), // 'free', 'metered', 'paid', 'permit'
    cost: spot.rate || spot.hourly_rate || 'Free',
    timeLimit: spot.time_limit || 'No limit',
    restrictions: spot.restrictions || spot.hours || 'None',
    available: spot.available !== false,
    parkingType: spot.parking_type || 'Street parking'
}));
```

#### Step 3: Create API Fetch Function
Add this function to fetch real parking data:

```javascript
async function fetchParkingData() {
    try {
        // Example endpoint - replace with actual Cambridge data URL
        const response = await fetch('https://data.cambridgema.gov/resource/[DATASET_ID].json?$limit=1000');
        const data = await response.json();

        // Transform data to match your structure
        return data.map(spot => ({
            id: spot.objectid,
            name: spot.location || 'Parking Location',
            lat: parseFloat(spot.latitude),
            lng: parseFloat(spot.longitude),
            type: spot.type || 'metered',
            cost: spot.rate || '$1.50/hour',
            timeLimit: spot.time_limit || '2 hours',
            restrictions: spot.restrictions || 'Mon-Sat 8am-8pm',
            available: true,
            parkingType: spot.parking_type || 'Street parking'
        }));
    } catch (error) {
        console.error('Error fetching parking data:', error);
        return sampleParkingData; // Fallback to sample data
    }
}

// Use it on page load
let parkingData = [];

async function initializeApp() {
    parkingData = await fetchParkingData();
    displayAllParking();
}

// Call on page load
initializeApp();
```

#### Step 4: Common Cambridge Dataset Fields
When working with Cambridge Open Data, look for these common fields:

- **Location**: `latitude`, `longitude`, `lat`, `lng`, `location`, `geolocation`
- **Address**: `street_name`, `address`, `location_name`
- **Type**: `parking_type`, `meter_type`, `zone_type`
- **Rate**: `rate`, `hourly_rate`, `daily_rate`
- **Hours**: `hours`, `restrictions`, `enforcement_hours`
- **Time Limits**: `time_limit`, `max_duration`
- **Zone**: `zone_id`, `permit_zone`, `parkmobile_zone`

### Specific Dataset Examples

#### Parking Meters Dataset
If Cambridge provides a parking meters dataset:

```javascript
// Fetch parking meters
const metersResponse = await fetch('https://data.cambridgema.gov/resource/parking-meters.json');
const meters = await metersResponse.json();

// Transform to app format
const meterParking = meters.map(meter => ({
    id: meter.meter_id,
    name: `Meter ${meter.meter_id}`,
    lat: meter.latitude,
    lng: meter.longitude,
    type: 'metered',
    cost: `$${meter.hourly_rate}/hour`,
    timeLimit: `${meter.max_hours} hours`,
    restrictions: meter.enforcement_schedule,
    available: true,
    parkingType: 'Street parking'
}));
```

#### Parking Lots/Garages Dataset
```javascript
// Fetch parking facilities
const facilitiesResponse = await fetch('https://data.cambridgema.gov/resource/parking-facilities.json');
const facilities = await facilitiesResponse.json();

const facilityParking = facilities.map(facility => ({
    id: facility.facility_id,
    name: facility.facility_name,
    lat: facility.latitude,
    lng: facility.longitude,
    type: facility.is_paid ? 'paid' : 'free',
    cost: facility.rates || 'Varies',
    timeLimit: 'No limit',
    restrictions: facility.hours || '24/7',
    available: true,
    parkingType: facility.type // 'Parking garage' or 'Parking lot'
}));
```

### Advanced Features to Add

#### Real-Time Availability
If Cambridge provides real-time data:

```javascript
async function updateAvailability() {
    const response = await fetch('https://api.cambridge.gov/parking/availability');
    const availability = await response.json();

    // Update markers based on current availability
    availability.forEach(spot => {
        const marker = parkingMarkers.find(m => m.id === spot.id);
        if (marker) {
            marker.setIcon(spot.available ? icons.available : icons.full);
        }
    });
}

// Refresh every 5 minutes
setInterval(updateAvailability, 300000);
```

#### Filter by Parking Type
Add filter buttons:

```html
<div class="filters">
    <button onclick="filterParking('all')">All</button>
    <button onclick="filterParking('free')">Free Only</button>
    <button onclick="filterParking('metered')">Metered</button>
    <button onclick="filterParking('paid')">Lots/Garages</button>
</div>
```

```javascript
function filterParking(type) {
    parkingMarkers.forEach(marker => {
        if (type === 'all' || marker.parkingType === type) {
            marker.addTo(map);
        } else {
            map.removeLayer(marker);
        }
    });
}
```

#### Walking Directions
Integrate with routing API:

```javascript
function getDirections(parkingLat, parkingLng, userLat, userLng) {
    const routing = L.Routing.control({
        waypoints: [
            L.latLng(userLat, userLng),
            L.latLng(parkingLat, parkingLng)
        ],
        routeWhileDragging: false
    }).addTo(map);
}
```

## Alternative Mapping Services

### Using Google Maps Instead of Leaflet
Replace Leaflet with Google Maps API:

```html
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY"></script>
```

### Using Mapbox
For better styling and geocoding:

```html
<script src='https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.js'></script>
<link href='https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.css' rel='stylesheet' />
```

## Resources

### Cambridge Open Data
- **Portal**: https://data.cambridgema.gov
- **GIS Maps**: https://www.cambridgema.gov/GIS
- **Contact**: OpenData@CambridgeMA.GOV
- **Phone**: 617-349-4140

### Parking Information
- **Parking Guide**: https://www.cambridgema.gov/iwantto/parkacarincambridge
- **Meters Map**: https://www.cambridgema.gov/iwantto/parkacarincambridge/map

### API Documentation
- **Socrata API Docs**: https://dev.socrata.com/
- **Nominatim (Geocoding)**: https://nominatim.org/release-docs/latest/api/Overview/
- **Leaflet.js Docs**: https://leafletjs.com/reference.html

## Deployment

### Option 1: Static Hosting (Simplest)
Host the HTML file on:
- **GitHub Pages** (free)
- **Netlify** (free)
- **Vercel** (free)
- **AWS S3** + CloudFront

### Option 2: Web Server
Deploy on:
- **Heroku**
- **DigitalOcean**
- **AWS EC2**

### Option 3: Local Development
Simply open the HTML file in a browser:
```bash
open cambridge-parking-finder.html
# or
python -m http.server 8000
# then visit http://localhost:8000
```

## Troubleshooting

### CORS Issues
If you encounter CORS errors when fetching data:
1. Use a proxy server
2. Enable CORS on the API endpoint
3. Run a local server instead of opening the file directly

### Geocoding Rate Limits
Nominatim has rate limits. For production:
- Use Mapbox Geocoding API
- Use Google Maps Geocoding API
- Implement caching for common addresses

### No Parking Data Showing
1. Check browser console for errors
2. Verify API endpoints are correct
3. Check data format matches expected structure
4. Ensure latitude/longitude are valid numbers

## Future Enhancements
- Mobile app version (React Native)
- User accounts and saved locations
- Push notifications for parking expiration
- Integration with payment systems (ParkMobile, PayByPhone)
- Historical availability data and predictions
- User-reported parking spots
- Accessibility features
- Multiple language support

## License
This is a demonstration project. When integrating real data, ensure compliance with Cambridge's data usage policies.

## Contact
For questions about Cambridge parking data:
- Email: OpenData@CambridgeMA.GOV
- Phone: 617-349-4140
