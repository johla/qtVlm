# qtVlm Architecture Documentation

> **Note**: This document is written with empathy for developers who are new to the codebase. We understand that jumping into a large Qt/C++ application can be overwhelming. Take your time, and don't hesitate to ask questions.

---

## Table of Contents
1. [Overview](#overview)
2. [Design Philosophy](#design-philosophy)
3. [System Architecture](#system-architecture)
4. [Core Components](#core-components)
5. [Data Flow](#data-flow)
6. [Threading Model](#threading-model)
7. [Coordinate Systems](#coordinate-systems)
8. [Extension Points](#extension-points)

---

## Overview

### What is qtVlm?

qtVlm (Virtual Loup de mer GUI) is a desktop application for virtual sailing enthusiasts. It provides:

- **Weather Visualization**: Display GRIB weather data on an interactive map
- **Route Planning**: Calculate optimal sailing routes based on weather conditions
- **Virtual Sailing**: Connect to Virtual Regatta (VLM) for online racing
- **Real Boat Tracking**: Support for actual GPS-connected boats
- **Navigation Tools**: Points of interest, barriers, orthodromic calculations

### Technology Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| UI Framework | Qt 5.x/4.x | Cross-platform GUI |
| Language | C++11 | Core logic |
| Graphics | Qt Graphics View | Map rendering |
| Networking | QNetworkAccessManager | HTTP/HTTPS communication |
| Data Formats | GRIB, XML, JSON, NMEA | Weather, boat data |
| Build System | qmake | Project compilation |

---

## Design Philosophy

### Guiding Principles

1. **User-Centric Design**: The interface should be intuitive for sailors, not programmers
2. **Data Visualization First**: Weather and navigation data must be clear and actionable
3. **Platform Independence**: Work seamlessly on Windows, Linux, and macOS
4. **Extensibility**: New features should be addable without major refactoring
5. **Performance**: Real-time updates with minimal lag

### Qt Patterns Used

- **Model-View-Controller (MVC)**: Separation of data, presentation, and logic
- **Signal-Slot**: Loose coupling between components
- **Parent-Child Memory Management**: Automatic cleanup through Qt's ownership
- **Graphics View Framework**: Scalable 2D graphics

---

## System Architecture

### High-Level Component Diagram

```
┌────────────────────────────────────────────────────────────┐
│                     Application Layer                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              MainWindow (QMainWindow)                 │  │
│  │  • Central application controller                     │  │
│  │  • Coordinates all subsystems                         │  │
│  │  • Manages application lifecycle                      │  │
│  └────────────┬────────────────────────────┬─────────────┘  │
└───────────────┼────────────────────────────┼────────────────┘
                │                            │
    ┌───────────▼─────────────┐  ┌──────────▼──────────────┐
    │  Presentation Layer     │  │   Data Layer            │
    └─────────────────────────┘  └─────────────────────────┘
    
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                        │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   MenuBar    │  │   ToolBar    │  │   StatusBar      │  │
│  │              │  │              │  │                  │  │
│  │ • File ops   │  │ • Quick      │  │ • Real-time      │  │
│  │ • Settings   │  │   actions    │  │   status info    │  │
│  │ • Help       │  │ • Zoom       │  │ • Boat data      │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              myCentralWidget (Container)               │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │         MyView (QGraphicsView)                   │ │ │
│  │  │  • Main map display                              │ │ │
│  │  │  • Handles user input (pan, zoom, click)         │ │ │
│  │  │  • Renders all graphics items                    │ │ │
│  │  │                                                  │ │ │
│  │  │  ┌────────────────────────────────────────────┐ │ │ │
│  │  │  │  QGraphicsScene (Scene Graph)              │ │ │ │
│  │  │  │  • Boats (QGraphicsWidget)                 │ │ │ │
│  │  │  │  • Routes (QGraphicsItem)                  │ │ │ │
│  │  │  │  • POIs (QGraphicsItem)                    │ │ │ │
│  │  │  │  • Weather overlays (QGraphicsItem)        │ │ │ │
│  │  │  │  • Map data (coastlines, etc.)             │ │ │ │
│  │  │  └────────────────────────────────────────────┘ │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                  Dialogs/ (50+ dialogs)                │ │
│  │  • Configuration dialogs                               │ │
│  │  • Data entry forms                                    │ │
│  │  • Information displays                                │ │
│  └────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                       Data Layer                             │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────┐   │
│  │           DataManager (Central Data Hub)             │   │
│  │  • Coordinates all data sources                      │   │
│  │  • Manages data lifecycle                            │   │
│  │  • Notifies views of changes                         │   │
│  └───┬──────────────────────────────────────────┬───────┘   │
│      │                                          │           │
│  ┌───▼──────────┐  ┌───────────┐  ┌───────────▼────────┐  │
│  │ Grib         │  │ Boat Data │  │ Geographic Data    │  │
│  │              │  │           │  │                    │  │
│  │ • GribV1     │  │ • boat    │  │ • GshhsReader     │  │
│  │ • GribV2     │  │ • boatVLM │  │   (coastlines)    │  │
│  │ • Records    │  │ • boatReal│  │ • GisReader       │  │
│  │ • Isobars    │  │ • Player  │  │   (GIS data)      │  │
│  │              │  │           │  │ • Terrain         │  │
│  └──────────────┘  └───────────┘  └────────────────────┘  │
│                                                             │
│  ┌──────────────┐  ┌────────────┐  ┌────────────────────┐ │
│  │ Navigation   │  │ Settings   │  │ Network            │ │
│  │              │  │            │  │                    │ │
│  │ • route      │  │ • settings │  │ • inetConnexion   │ │
│  │ • routage    │  │ • Polars   │  │ • inetClient      │ │
│  │ • POI        │  │            │  │ • xmlBoatData     │ │
│  │ • Barrier    │  │            │  │                    │ │
│  └──────────────┘  └────────────┘  └────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      Support Layer                           │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Projection   │  │ Util         │  │ Interpolation    │  │
│  │              │  │              │  │                  │  │
│  │ • Map        │  │ • Helpers    │  │ • Weather data   │  │
│  │   transforms │  │ • File ops   │  │ • Time-based     │  │
│  │ • Coord      │  │ • String     │  │ • Spatial        │  │
│  │   conversion │  │   utilities  │  │                  │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. MainWindow - Application Controller

**File**: `MainWindow.cpp/h`  
**Responsibility**: Central application controller and coordinator

```cpp
class MainWindow : public QMainWindow
{
    Q_OBJECT
    
    // Key responsibilities:
    // 1. Create and manage all major subsystems
    // 2. Handle application-level events
    // 3. Coordinate data flow between components
    // 4. Manage application state
};
```

**Key Features**:
- Creates MenuBar, ToolBar, StatusBar
- Initializes DataManager and myCentralWidget
- Handles file operations (open GRIB, save routes)
- Manages boat selection and tracking
- Coordinates dialog displays

**Important Methods**:
- `openGribFile()`: Load weather data
- `continueSetup()`: Complete application initialization
- `getSelectedBoat()`: Access currently selected vessel
- `slot_POI_input()`: Handle POI creation/editing

---

### 2. MyView - Graphics View

**File**: `MyView.cpp/h`  
**Responsibility**: Main map display and user interaction

```cpp
class MyView : public QGraphicsView
{
    Q_OBJECT
    
    // Key responsibilities:
    // 1. Render the map and all overlays
    // 2. Handle mouse/keyboard input
    // 3. Implement zoom and pan
    // 4. Draw weather data visualization
};
```

**Rendering Pipeline**:
1. **Background**: Ocean/land from GSHHS data
2. **Grid**: Latitude/longitude lines
3. **Weather**: GRIB data as colored overlays/arrows
4. **Navigation**: Routes, POIs, barriers
5. **Boats**: Player and opponent vessels
6. **UI Overlays**: Compass rose, magnifier, etc.

**User Interactions**:
- **Left Click**: Select boat, POI, or route
- **Right Click**: Context menu
- **Wheel**: Zoom in/out
- **Drag**: Pan the map
- **Shift+Drag**: Measure distance

---

### 3. Projection - Coordinate System

**File**: `Projection.cpp/h`  
**Responsibility**: Map projection and coordinate transformations

```cpp
class Projection : public QObject
{
    // Implements Mercator projection
    // Converts between:
    // - Screen coordinates (pixels)
    // - Map coordinates (longitude/latitude in degrees)
};
```

**Critical Transformations**:

```cpp
// Screen pixels → Geographic coordinates
void screen2map(const int &i, const int &j, double *lon, double *lat);

// Geographic coordinates → Screen pixels
void map2screen(const double &lon, const double &lat, int *i, int *j);

// Zoom operations
void zoom(const double &k);  // Scale by factor k
void zoomKeep(const double &lon, const double &lat, const double &k);
```

**Understanding Mercator Projection**:
- **Longitude (X)**: Linear mapping
- **Latitude (Y)**: Logarithmic (ln(tan(lat))) to preserve angles
- **Distortion**: Increases toward poles
- **Suitable for**: Navigation (preserves bearing angles)

---

### 4. Grib - Weather Data

**Files**: `Grib.cpp/h`, `GribV1.*`, `GribV2.*`, `GribRecord.*`  
**Responsibility**: Weather data loading, parsing, and access

```cpp
class Grib : public QObject
{
    // Base class for GRIB file handling
    // Version-specific implementations:
    // - GribV1: GRIB Edition 1
    // - GribV2: GRIB Edition 2 (more common)
};

class GribRecord
{
    // Represents a single weather parameter at a specific time
    // Contains gridded data (e.g., wind speed/direction)
};
```

**GRIB Data Structure**:
```
GRIB File
├── Record 1: Wind U-component at 2024-01-01 00:00
├── Record 2: Wind V-component at 2024-01-01 00:00
├── Record 3: Pressure at 2024-01-01 00:00
├── Record 4: Wind U-component at 2024-01-01 06:00
└── ... (multiple times and parameters)
```

**Key Operations**:
```cpp
// Get weather data at a specific location and time
bool interpolateValue_2D(
    double lon, double lat,      // Position
    time_t now,                  // Time
    double *windSpeed,           // Output: wind speed
    double *windDirection        // Output: wind direction
);
```

---

### 5. Boat Hierarchy - Vessel Management

**Files**: `boat.h`, `boatVLM.h`, `boatReal.h`

```
        boat (Abstract base)
        /              \
   boatVLM           boatReal
(Virtual sailing)  (GPS tracking)
```

**boat (Base Class)**:
- Common boat properties (position, speed, heading)
- Polar diagram management (performance curves)
- Waypoint navigation
- Visual representation

**boatVLM (Virtual Regatta)**:
- Connects to VLM server
- Receives virtual race data
- Sends commands (heading, sail changes)
- Displays race rankings

**boatReal (Physical GPS)**:
- Reads NMEA GPS data
- Tracks actual boat position
- No command capability (read-only)

**Key Boat Data**:
```cpp
double lat, lon;          // Position (degrees)
double speed;             // Speed (knots)
double heading;           // Direction (degrees true)
double windSpeed;         // Wind speed at boat (knots)
double windDir;           // Wind direction (degrees true)
double TWA;               // True Wind Angle (degrees)
QPointF WP;               // Current waypoint (lon, lat)
Polar *polarData;         // Performance data
```

---

### 6. Route & Routing

**Files**: `route.cpp/h`, `routage.cpp/h`

**route**: Simple route (list of waypoints)
```cpp
class ROUTE
{
    QList<POI*> poiList;  // Ordered list of waypoints
    // Basic route management
};
```

**routage**: Weather routing (optimization)
```cpp
class routage
{
    // Calculates optimal route based on:
    // - Weather forecast (GRIB data)
    // - Boat polar diagram
    // - Start/end points
    // - Time constraints
    
    // Uses isochrone method:
    // - Expand possible positions at time intervals
    // - Find fastest path through weather patterns
};
```

**Isochrone Routing Algorithm**:
```
Start Point
     │
Time T+1h: ─── Possible positions after 1 hour
     │
Time T+2h: ───── Possible positions after 2 hours
     │
Time T+3h: ──────── Possible positions after 3 hours
     │
     └──────────────────> Destination
     
At each step:
1. Calculate boat speed in all directions (using polar)
2. Apply current weather conditions
3. Expand reachable positions
4. Prune inefficient paths
5. Continue until destination reached
```

---

### 7. DataManager - Data Coordinator

**File**: `DataManager.cpp/h`  
**Responsibility**: Central coordination of all data sources

```cpp
class DataManager : public QObject
{
    // Manages:
    // - GRIB weather data
    // - Color schemes
    // - Data interpolation
    // - Time management for forecasts
};
```

**Why DataManager?**
- **Single Source of Truth**: One place to query data
- **Decoupling**: Views don't need to know about GRIB internals
- **Caching**: Optimize repeated queries
- **Consistency**: Ensure all views use same data

---

### 8. POI - Points of Interest

**File**: `POI.cpp/h`

```cpp
class POI : public QObject
{
    // Represents a navigation waypoint
    double lat, lon;           // Position
    QString name;              // User-defined label
    QString description;       // Notes
    int type;                  // POI type (waypoint, mark, etc.)
    bool hasInstruction;       // Auto-pilot instruction
};
```

**POI Types**:
- **WP**: Simple waypoint
- **WPL**: Waypoint with lat/lon specified
- **NM**: Navigation mark
- **MOB**: Man overboard
- **DEST**: Destination

---

## Data Flow

### Application Startup Sequence

```
1. main()
   ├─> Create QApplication
   ├─> Initialize Settings
   ├─> Load translations (i18n)
   └─> Create MainWindow
       │
2. MainWindow::continueSetup()
   ├─> Create myCentralWidget
   ├─> Create MyView (graphics view)
   ├─> Create Projection
   ├─> Create DataManager
   ├─> Create MenuBar, ToolBar, StatusBar
   ├─> Load saved settings
   ├─> Restore window geometry
   ├─> Load last opened GRIB (if any)
   └─> Show window
       │
3. Event Loop Starts
   └─> Application ready for user interaction
```

### Opening a GRIB File

```
User: File → Open GRIB
   │
   ▼
MainWindow::slot_openGribFile()
   │
   ▼
DialogLoadGrib (file selection)
   │
   ▼
MainWindow::openGribFile(fileName)
   │
   ├─> Grib::loadGrib(fileName)
   │   └─> Determine GRIB version
   │       ├─> GribV1::loadFile() or
   │       └─> GribV2::loadFile()
   │           ├─> Parse binary GRIB format
   │           ├─> Create GribRecord objects
   │           └─> Build time/parameter indices
   │
   ├─> DataManager::setGrib(grib)
   │
   ├─> Update UI (time slider, available parameters)
   │
   └─> MyView::update()
       └─> Render weather data on map
```

### Real-Time Weather Interpolation

```
User moves boat on map
   │
   ▼
boat::setPosition(lat, lon)
   │
   ▼
boat::updateWindData()
   │
   ▼
Grib::interpolateValue_2D(lat, lon, currentTime, &windSpeed, &windDir)
   │
   ├─> Find 4 surrounding grid points
   │   (NW, NE, SW, SE)
   │
   ├─> Find 2 surrounding time steps
   │   (before, after)
   │
   ├─> Bilinear spatial interpolation
   │   (within each time step)
   │
   ├─> Linear temporal interpolation
   │   (between time steps)
   │
   └─> Return interpolated values
       │
       ▼
boat::calculatePerformance()
   ├─> Lookup polar diagram
   ├─> Calculate speed for current TWA
   └─> Update display
```

---

## Threading Model

### Current Threading Strategy: **Primarily Single-Threaded**

**Main (GUI) Thread**:
- All UI operations
- Event handling
- Graphics rendering
- Most data processing

**Background Operations** (where implemented):
- Network downloads (QNetworkAccessManager has built-in threading)
- Some file I/O operations

### Implications

**Pros**:
- Simpler code (no synchronization needed)
- Fewer race conditions
- Easier to debug

**Cons**:
- Long operations block UI
- No multi-core utilization for heavy calculations
- Route optimization can freeze interface

### Future Threading Opportunities

1. **Route Calculation**:
   ```cpp
   // Could use QtConcurrent::run()
   QFuture<ROUTE*> future = QtConcurrent::run(
       this, &routage::calculate, start, end
   );
   ```

2. **GRIB Loading**:
   ```cpp
   // Background parsing of large files
   QThread *loaderThread = new QThread;
   GribLoader *loader = new GribLoader(fileName);
   loader->moveToThread(loaderThread);
   connect(loaderThread, &QThread::started, 
           loader, &GribLoader::load);
   loaderThread->start();
   ```

3. **Weather Interpolation**:
   ```cpp
   // Pre-calculate interpolations for visible area
   QThreadPool::globalInstance()->start(
       new InterpolationTask(grib, viewport)
   );
   ```

---

## Coordinate Systems

### Understanding qtVlm Coordinates

**1. Geographic Coordinates (WGS84)**
- **Longitude**: -180° (West) to +180° (East)
- **Latitude**: -90° (South Pole) to +90° (North Pole)
- **Usage**: Storing positions, GRIB data, GPX files

**2. Screen Coordinates**
- **Origin**: Top-left corner (0, 0)
- **X-axis**: Left to right (pixels)
- **Y-axis**: Top to bottom (pixels)
- **Usage**: Mouse events, widget positioning, rendering

**3. Map Coordinates (Projected)**
- **Mercator Projection**: Lon/Lat transformed for linear display
- **X**: Directly proportional to longitude
- **Y**: ln(tan(lat)) - logarithmic latitude
- **Usage**: Internal calculations, scaling

### Coordinate Conversion Examples

```cpp
// 1. User clicks on map
void MyView::mousePressEvent(QMouseEvent *event)
{
    // Screen coordinates from mouse
    int screenX = event->pos().x();
    int screenY = event->pos().y();
    
    // Convert to geographic
    double lon, lat;
    projection->screen2map(screenX, screenY, &lon, &lat);
    
    // Now we have the clicked position
    qDebug() << "Clicked at" << lat << "°N," << lon << "°E";
}

// 2. Drawing a boat at a position
void boat::paint(QPainter *painter)
{
    // Boat has geographic position
    double boatLon = this->lon;
    double boatLat = this->lat;
    
    // Convert to screen for drawing
    int screenX, screenY;
    projection->map2screen(boatLon, boatLat, &screenX, &screenY);
    
    // Draw boat icon at screen position
    painter->drawPixmap(screenX, screenY, boatIcon);
}
```

### Handling the Date Line (±180°)

```cpp
// When crossing the date line:
// Longitude might wrap: 179° → -179°

double normalizeLongitude(double lon)
{
    while (lon > 180.0) lon -= 360.0;
    while (lon < -180.0) lon += 360.0;
    return lon;
}

// Calculate distance considering date line:
double longitudeDifference(double lon1, double lon2)
{
    double diff = lon2 - lon1;
    if (diff > 180.0) diff -= 360.0;
    if (diff < -180.0) diff += 360.0;
    return diff;
}
```

---

## Extension Points

### How to Add New Features

#### 1. Adding a New Weather Parameter

**Scenario**: Display wave height from GRIB

1. **Add data type constant**:
   ```cpp
   // In dataDef.h
   #define DATA_WAVES_SIGNIFICANT_HEIGHT  1000
   ```

2. **Parse from GRIB**:
   ```cpp
   // In GribV2.cpp
   if (parameterNumber == 100) {  // GRIB2 code for wave height
       dataType = DATA_WAVES_SIGNIFICANT_HEIGHT;
   }
   ```

3. **Add visualization**:
   ```cpp
   // In MyView.cpp or MapDataDrawer.cpp
   if (showWaveHeight) {
       drawWaveHeightOverlay(grib, currentTime);
   }
   ```

4. **Add UI controls**:
   - Menu item to toggle display
   - Color scale legend
   - Settings for display style

#### 2. Adding a New Boat Type

**Scenario**: Support for AIS (Automatic Identification System) data

1. **Create new boat subclass**:
   ```cpp
   // boatAIS.h
   class boatAIS : public boat
   {
       Q_OBJECT
   public:
       boatAIS(QString mmsi, Projection *proj, 
               MainWindow *main, myCentralWidget *parent);
       
       void updateFromAIS(AISMessage msg);
       
   private:
       QString mmsi;  // Unique AIS identifier
       QString vesselName;
       int vesselType;
   };
   ```

2. **Add AIS data source**:
   ```cpp
   // AISReceiver.h
   class AISReceiver : public QObject
   {
       Q_OBJECT
   signals:
       void aisMessageReceived(AISMessage msg);
   };
   ```

3. **Integrate into MainWindow**:
   ```cpp
   // MainWindow.cpp
   AISReceiver *aisReceiver = new AISReceiver();
   connect(aisReceiver, &AISReceiver::aisMessageReceived,
           this, &MainWindow::handleAISMessage);
   ```

#### 3. Adding a New File Format

**Scenario**: Import routes from GPX files

1. **Create parser**:
   ```cpp
   // GPXReader.h
   class GPXReader
   {
   public:
       QList<ROUTE*> parseGPX(QString filename);
   private:
       ROUTE* parseTrack(QDomElement trackElement);
       POI* parseWaypoint(QDomElement wptElement);
   };
   ```

2. **Add menu action**:
   ```cpp
   // In MenuBar.cpp
   QAction *importGPX = new QAction(tr("Import GPX..."), this);
   connect(importGPX, &QAction::triggered, 
           mainWindow, &MainWindow::slot_importGPX);
   ```

3. **Implement import handler**:
   ```cpp
   void MainWindow::slot_importGPX()
   {
       QString fileName = QFileDialog::getOpenFileName(
           this, tr("Import GPX"), "", "GPX Files (*.gpx)"
       );
       
       GPXReader reader;
       QList<ROUTE*> routes = reader.parseGPX(fileName);
       
       for (ROUTE *route : routes) {
           addRoute(route);  // Add to application
       }
   }
   ```

---

## Best Practices for Contributing

### When Working with the Codebase

1. **Understand Qt Ownership**:
   - Objects with a parent are deleted automatically
   - Don't delete objects with parents manually
   - Use `deleteLater()` for safe deletion

2. **Use Qt Types**:
   - `QString` instead of `std::string`
   - `QList` instead of `std::vector`
   - Qt types integrate better with signals/slots

3. **Signal/Slot Connections**:
   - Always connect signals to slots of same signature
   - Use new connection syntax when possible:
     ```cpp
     // New (type-safe)
     connect(sender, &Sender::signal, 
             receiver, &Receiver::slot);
     
     // Old (string-based, avoid)
     connect(sender, SIGNAL(signal()), 
             receiver, SLOT(slot()));
     ```

4. **Coordinate Conversions**:
   - Always go through `Projection` class
   - Never do manual lat/lon to pixel calculations
   - Remember: screen Y increases downward, lat increases upward

5. **Performance**:
   - Cache frequently accessed data
   - Use `const` references for large objects
   - Profile before optimizing

6. **Error Handling**:
   - Check return values (especially for file operations)
   - Use `qWarning()` or `qDebug()` for logging
   - Provide user-friendly error messages

---

## Common Pitfalls

### ❌ Don't Do This:

```cpp
// 1. Manual deletion of child objects
MyView *view = new MyView(parent);
delete view;  // BAD: parent will try to delete again

// 2. Forgetting to normalize longitude
double distance = lon2 - lon1;  // BAD: breaks at date line

// 3. Blocking the UI thread
void calculateRoute() {
    // Long calculation...
}  // BAD: UI freezes

// 4. Direct screen drawing without projection
painter.drawLine(lon1, lat1, lon2, lat2);  // BAD: wrong coordinates
```

### ✅ Do This Instead:

```cpp
// 1. Let parent handle deletion
MyView *view = new MyView(parent);
// No manual delete needed

// 2. Use helper functions
double distance = Util::longitudeDifference(lon1, lon2);

// 3. Use background processing
QtConcurrent::run(this, &MyClass::calculateRoute);

// 4. Always convert coordinates
int x1, y1, x2, y2;
projection->map2screen(lon1, lat1, &x1, &y1);
projection->map2screen(lon2, lat2, &x2, &y2);
painter.drawLine(x1, y1, x2, y2);
```

---

## Questions & Support

**Still confused?** That's completely normal! This is a complex application with many interconnected parts. Here are some tips:

1. **Start Small**: Focus on one component at a time
2. **Use Debugger**: Step through code to understand flow
3. **Read Qt Docs**: Qt documentation is excellent
4. **Ask Questions**: Don't suffer in silence!
5. **Look at Similar Code**: See how existing features work

**Common Questions**:

- *"Where do I add menu items?"* → `MenuBar.cpp`
- *"How do I access weather data?"* → Through `DataManager` and `Grib`
- *"Where is the map drawn?"* → `MyView::paintEvent()` and related
- *"How do boats update?"* → Timers trigger `boat::update()` methods
- *"Where are settings stored?"* → `Settings` class wraps QSettings

---

*Remember: Every expert was once a beginner. Take your time, experiment, and don't be afraid to make mistakes. That's how we all learn!*

**Happy Coding! ⛵**
