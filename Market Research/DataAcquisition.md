# OSM DATA ACQUISITION

## **11 February 2025**

OSM – OpenStreetMaps is an open-source mapping project that provides free and editable map data to anyone. It is updated regularly (OSM data changes frequently).

## **Structure of OSM Data**

OSM data is primarily stored as a collection of three core elements:

1. **Nodes** – Represent points with latitude and longitude (e.g., intersections, landmarks).
2. **Ways** – Ordered lists of nodes representing roads, paths, or boundaries.
3. **Relations** – Groupings of nodes, ways, or other relations that define complex structures (e.g., bus routes, administrative boundaries).

Each element in OSM data can have tags, which provide metadata. Tags consist of key-value pairs, such as:

- `highway=primary` (indicating a main road)
- `building=yes` (indicating a building)
- `name=Central Park` (providing a name for a place)

## **File Formats for OSM Data**

- `.osm` (XML-based raw OSM data)
- `.pbf` (Protocolbuffer Binary Format, a compressed version of OSM XML)
- `.geojson` (GeoJSON format for spatial data applications)
- `.shp` (Shapefile format for GIS software)

## **Download OSM Data**

Since we are operational only in and around Hyderabad, a small region’s OSM data would suffice.

### **Direct Downloads – Whole Data vs. API Queries – Specific Data**

#### **Sources:**

1. **Geofabrik** ([https://download.geofabrik.de/](https://download.geofabrik.de/)) - Offers regular extracts of OSM data by continent/country/region.
2. **Overpass API** - For querying specific data within an area (**CLIENT-SUGGESTED**).
3. **BBBike Extract** ([https://extract.bbbike.org/](https://extract.bbbike.org/)) - Allows custom area selection for smaller regions.
4. **OSM API** - For accessing raw OSM data.

## **What is the Overpass API?**

The Overpass API allows you to query OpenStreetMap (OSM) data using a structured query language. You can extract specific data based on geographic regions, object types, and tag values.

For example:

- Find All Hospitals Within 10km of a Location
- Find All Buildings in a City
- Find a Specific Place by Name, e.g., Eiffel Tower

### **Special Features of Overpass API:**

- **Filters by Tag Values**
- **Sorting & Limiting Results**
- **Recursive Queries** (Find all parts of a relation)

## **How to Run Overpass Queries?**

### **1. Overpass Turbo (Web-based UI)**  
- Visit [https://overpass-turbo.eu](https://overpass-turbo.eu)
- Paste the query and click **Run**.

### **2. Direct API Call (Using curl or browser)**  
```
https://overpass-api.de/api/interpreter?data=[YOUR_QUERY]
```
- Replace `[YOUR_QUERY]` with the actual query in **URL-encoded format**.

---

# **TASK AT HAND**

We aim to obtain relevant OpenStreetMap (OSM) data:

### **1. Download OSM Data for Hyderabad**

- Option 1: **Download from Geofabrik**
  - `india-latest.osm.pbf`, suitable for Osmium, Osmosis, imposm, osm2pgsql, mkgmap, and others. (Available on Geofabrik’s official website.)
  - **Closest Extract:** Southern Zone (~500MB)
  - Extract Hyderabad’s Bounding Box (Approximate)
  - Extract Hyderabad Data from the `.pbf` file

- Option 2: **Download using Overpass API Queries**
  - **Use Overpass API for Direct Hyderabad Data**
  - GUI: Use [Overpass Turbo](https://overpass-turbo.eu)
  - Terminal-based
  - Conclusion : Time-consuming
  
#### **Example Overpass Query:**
```xml
[out:xml];
area[name="Hyderabad"]->.searchArea;
(
  way["highway"](area.searchArea);
);
out body;
>;
out skel qt;
```

### **Direct API Call - Most Efficient & Quick Download**

```
https://overpass-api.de/api/interpreter?data=[YOUR_QUERY]
```

Replace `[YOUR_QUERY]` with the actual Overpass API query.

---

### **Additional Resources:**
- [Geofabrik Download Page](https://download.geofabrik.de/asia/india-latest.osm.pbf)
- [Overpass API Documentation](https://overpass-api.de/api/interpreter?data=[YOUR_QUERY])
- [Overpass Turbo](https://overpass-turbo.eu)

---

## **Conclusion**

After evaluating multiple methods, direct Overpass API calls proved to be the most efficient for extracting Hyderabad’s road network, eliminating the need for additional filtering and processing. This method significantly reduced data size and download time while maintaining accuracy.

