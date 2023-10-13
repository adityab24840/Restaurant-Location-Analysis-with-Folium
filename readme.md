# Restaurant Location Analysis with Folium

Spatially analysis of Zomato Dataset for restaurants in Bangalore, India, and visualize them on a map using Folium.
This also calculates distances and clusters from a user-specified location to each restaurant.

## Table of Contents

1. [Libraries Installation](#libraries-installation)
2. [Geocoding with Geopy](#geocoding-with-geopy)
3. [Haversine Formula](#haversine-formula)
4. [Plotting Spatial Data on Maps](#plotting-spatial-data-on-maps)
5. [Plotting All Restaurants in Bangalore](#plotting-all-restaurants-in-bangalore)
6. [Creating a Heatmap](#creating-a-heatmap)
7. [Using Marker Clusters](#using-marker-clusters)
8. [Calculating Distances to Restaurants](#calculating-distances-to-restaurants)

## 1. Libraries Installation <a name="libraries-installation"></a>

The code starts by installing necessary Python libraries. These libraries are used for various tasks in the project.

```python
!pip install geopy
!pip install folium
!pip install wordcloud
```

## 2. Geocoding with Geopy <a name="geocoding-with-geopy"></a>

This part of the code uses Geopy to perform geocoding, which means converting an address into latitude and longitude coordinates.

```python
# Initialize the geolocator
geolocator = Nominatim(user_agent="your_user_agent")

# Add your location here
address = 'Christ University'  

# Geocode the address to get latitude and longitude
location = geolocator.geocode(address)

# Check if the location was found
if location is not None:
    # Extract latitude and longitude
    latitude = location.latitude
    longitude = location.longitude

    # Print the latitude and longitude
    print(f"Latitude: {latitude}")
    print(f"Longitude: {longitude}")

    # Reverse geocode to get the address
    location_reverse = geolocator.reverse((latitude, longitude))
    address_final = location_reverse.address
    print(f"Reverse Geocoded Address: {address_final}")
else:
    print("Location not found")
```

## 3. Haversine Formula <a name="haversine-formula"></a>

The Haversine formula is used to calculate the distance between two sets of geographical coordinates. This formula calculates the distance in kilometers.

```python
# Building a Formula
def distance(lat1, lon1, lat2, lon2):
  p = 0.017453292519943295 
  # p is for degree to radian, π/180, used to ensure that the latitude and longitude values are correctly expressed in radians
  c = math.cos
  a = 0.5 - c((lat2 - lat1) * p)/2 + c(lat1 * p) * c(lat2 * p) * (1 - c((lon2 - lon1) * p))/2
  return 12742 * math.asin(math.sqrt(a))

# Example usage
d = distance(12.9703944526, 77.6447132975, 12.9732913, 77.6404672)
print("The Distance is= ", d, 'kilometer')
```

## 4. Plotting Spatial Data on Maps <a name="plotting-spatial-data-on-maps"></a>

This section involves plotting your geographical coordinates on a map. The user's coordinates are plotted on the map.

```python
# Creating the Map with latitudes and longitudes set to the user's location
user_map = folium.Map(location=[latitude, longitude], zoom_start=16)

# Plotting the user coordinates
a = folium.map.FeatureGroup()
location_name = address_final.split(',')[0]
a.add_child(folium.Marker([latitude, longitude], radius=10, color='black', fill_color='black', popup=location_name))
user_map.add_child(a)
```

## 5. Plotting All Restaurants in Bangalore <a name="plotting-all-restaurants-in-bangalore"></a>

This part of the code reads a dataset of restaurant locations and plots them on the map. Each restaurant is represented by a red circle marker.

```python
data = pd.read_csv("Data/Bangalore_Restaurants.csv")

# Iteratively plot the data on the map
a = folium.map.FeatureGroup()

# Loop over the restaurant dataset
for lat, lng, label in zip(data.Latitude, data.Longitude, data.Restaurant_Name):
  a.add_child(folium.CircleMarker([lat, lng], radius=2, color='red', fill=True, fill_color='black', popup=label))

# Add the markers to the user map
user_map.add_child(a)
```

## 6. Creating a Heatmap <a name="creating-a-heatmap"></a>

In this section, you'll create a heatmap to visualize restaurant density in Bangalore.

```python
heatmap_data = data[['Latitude', 'Longitude']].values
user_map.add_child(HeatMap(heatmap_data, radius=15))
```

## 7. Using Marker Clusters <a name="using-marker-clusters"></a>

This part involves using marker clusters to handle a large number of markers efficiently.

```python
marker_cluster = MarkerCluster().add_to(user_map)

# Add markers for each restaurant
for lat, lng, label in zip(data.Latitude, data.Longitude, data.Restaurant_Name):
    folium.Marker(location=[lat, lng], popup=label).add_to(marker_cluster)   
```

## 8. Calculating Distances to Restaurants <a name="calculating-distances-to-restaurants"></a>

Here, the code calculates the distances from the user's location to each restaurant and displays them as markers on the map.

```python
# Zoom out to see the cluster, it may take some time
user_map = folium.Map(location=[latitude, longitude], zoom_start=16)

# User's location
user_location = (latitude, longitude)

# Calculate distance from user location to each restaurant
def calculate_distance(lat, lon, location_name):
    location = (lat, lon)
    distance = geopy.distance.distance(user_location, location).km
    return f'Distance to {location_name}: {distance:.2f} km'

# Add markers with distance information
for lat, lon, label in zip(data.Latitude, data.Longitude, data.Restaurant_Name):
    popup = folium.Popup(calculate_distance(lat, lon, label), max_width=200)
    folium.Marker(location=[lat, lon], popup=popup, tooltip=label).add_to(user_map)
```
