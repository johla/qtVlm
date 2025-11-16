# qtVlm API Reference

> **Note**: This API reference documents the most important classes and interfaces in qtVlm. For complete details, refer to the source code. We're gradually adding Doxygen comments to improve this documentation.

---

## Table of Contents

1. [Core Classes](#core-classes)
2. [Data Management](#data-management)
3. [Navigation & Routing](#navigation--routing)
4. [Graphics & UI](#graphics--ui)
5. [Utility Classes](#utility-classes)
6. [Common Patterns](#common-patterns)

---

## Core Classes

### MainWindow

**File**: `src/MainWindow.h`, `src/MainWindow.cpp`  
**Inherits**: `QMainWindow`

Central application controller that coordinates all subsystems.

#### Key Methods

```cpp
// Construction
MainWindow(int width, int height, QWidget *parent = nullptr);

// GRIB Weather Operations
void openGribFile(QString fileName, bool zoom = true, bool current = false);

// Boat Management
boat* getSelectedBoat();
bool getBoatLockStatus();
void get_selectedBoatPos(double *lat, double *lon);

// Route & POI
QList<POI*>* getPois();

// UI Components
StatusBar* get_statusBar();
ToolBar* get_toolBar();
Progress* get_progress();
```

#### Signals

```cpp
// Emitted when boat selection changes
void boatSelectionChanged(boat *newSelection);

// Emitted when GRIB file is loaded
void gribLoaded(Grib *grib);
```

#### Example Usage

```cpp
// Get currently selected boat position
MainWindow *main = /* ... */;
double lat, lon;
main->get_selectedBoatPos(&lat, &lon);
qDebug() << "Boat at" << lat << "," << lon;

// Open a GRIB file
main->openGribFile("/path/to/weather.grb", true, false);
```

---

### Projection

**File**: `src/Projection.h`, `src/Projection.cpp`  
**Inherits**: `QObject`

Handles map projection and coordinate transformations using Mercator projection.

#### Constructor

```cpp
Projection(int screenWidth, int screenHeight, double centerLon, double centerLat);
```

#### Coordinate Transformations

```cpp
// Screen coordinates to geographic (lon/lat)
void screen2map(const int &screenX, const int &screenY, 
                double *lon, double *lat) const;

void screen2map(const QPoint &screenCoord, QPointF *position) const;

// Geographic to screen coordinates
void map2screen(const double &lon, const double &lat, 
                int *screenX, int *screenY) const;

void map2screenDouble(const double &lon, const double &lat,
                      double *screenX, double *screenY) const;
```

#### Zoom Operations

```cpp
// Zoom by factor k (>1 zooms in, <1 zooms out)
void zoom(const double &k);

// Zoom while keeping specific point stationary
void zoomKeep(const double &lon, const double &lat, const double &k);

// Zoom to show entire data extent
void zoomAll();

// Zoom to specific geographic rectangle
void zoomOnZone(const double &lon0, const double &lat0,
                const double &lon1, const double &lat1);
```

#### View Control

```cpp
// Move view by pixel offset
void move(const double &dx, const double &dy);

// Set center position
void setCenterInMap(const double &lon, const double &lat);

// Set scale and center together
void setScaleAndCenterInMap(const double &scale, 
                           const double &lon, const double &lat);
```

#### Accessors

```cpp
int getW() const;        // Screen width in pixels
int getH() const;        // Screen height in pixels
double getCX() const;    // Center longitude
double getCY() const;    // Center latitude
double getScale() const; // Current scale factor
```

#### Visibility Testing

```cpp
// Check if point is visible in current view
bool isPointVisible(const double &lon, const double &lat) const;

// Check if rectangle intersects view
bool intersect(const double &west, const double &east,
              const double &south, const double &north) const;
```

#### Example Usage

```cpp
Projection *proj = new Projection(1024, 768, 0.0, 0.0);

// Convert mouse click to geographic coordinates
void MyView::mousePressEvent(QMouseEvent *event) {
    double lon, lat;
    proj->screen2map(event->x(), event->y(), &lon, &lat);
    qDebug() << "Clicked at" << lat << "°N," << lon << "°E";
}

// Draw a marker at a location
void drawMarker(QPainter *painter, double lon, double lat) {
    int x, y;
    proj->map2screen(lon, lat, &x, &y);
    if (proj->isPointVisible(lon, lat)) {
        painter->drawEllipse(QPoint(x, y), 5, 5);
    }
}
```

---

## Data Management

### Grib (Abstract Base)

**File**: `src/Grib.h`, `src/Grib.cpp`  
**Inherits**: `QObject`  
**Subclasses**: `GribV1`, `GribV2`

Base class for GRIB weather file handling.

#### Static Factory

```cpp
// Load GRIB file (auto-detects version)
static Grib* loadGrib(QString fileName, DataManager *dataManager);
```

#### Weather Data Access

```cpp
// Get specific record
GribRecord* getGribRecord(int dataType, int levelType, 
                         int levelValue, time_t date);

// Find records around a time
void find_recordsAroundDate(int dataType, int levelType, int levelValue,
                           time_t date, 
                           GribRecord **before, GribRecord **after);

// Interpolate weather at position and time
static bool interpolateValue_2D(
    double lon, double lat,        // Position
    time_t now,                    // Time
    time_t t1, time_t t2,          // Surrounding times
    GribRecord *recU1, GribRecord *recV1,  // U/V at t1
    GribRecord *recU2, GribRecord *recV2,  // U/V at t2
    double *windSpeed,             // Output: wind speed
    double *windDirection          // Output: wind direction (degrees)
);
```

#### Data Type Constants

```cpp
// In dataDef.h
#define DATA_WIND_VX        1000  // Wind U-component (east-west)
#define DATA_WIND_VY        1001  // Wind V-component (north-south)
#define DATA_WIND_SPEED     1002  // Wind speed
#define DATA_WIND_DIR       1003  // Wind direction
#define DATA_PRESSURE       1004  // Atmospheric pressure
#define DATA_TEMPERATURE    1005  // Temperature
#define DATA_HUMID_REL      1006  // Relative humidity
// ... more types
```

#### Example Usage

```cpp
Grib *grib = Grib::loadGrib("/path/to/weather.grb2", dataManager);

if (grib && grib->isOk()) {
    // Get wind at specific position and time
    double lon = -30.5, lat = 48.2;
    time_t now = QDateTime::currentDateTimeUtc().toTime_t();
    
    double windSpeed, windDir;
    bool success = grib->interpolateValue_2D(
        lon, lat, now,
        &windSpeed, &windDir
    );
    
    if (success) {
        qDebug() << "Wind:" << windSpeed << "knots at" << windDir << "°";
    }
}
```

---

### GribRecord

**File**: `src/GribRecord.h`, `src/GribRecord.cpp`

Represents a single weather parameter at a specific time and level.

#### Accessors

```cpp
int getDataType() const;      // e.g., DATA_WIND_VX
int getLevelType() const;     // e.g., LV_GND_SURF (surface)
int getLevelValue() const;    // Level value (e.g., height in meters)
time_t getRecordCurrentDate() const;  // Valid time

int getNi() const;            // Grid points in longitude
int getNj() const;            // Grid points in latitude

double getXmin() const;       // Western edge (degrees)
double getXmax() const;       // Eastern edge (degrees)
double getYmin() const;       // Southern edge (degrees)
double getYmax() const;       // Northern edge (degrees)
```

#### Data Access

```cpp
// Get interpolated value at position
double getInterpolatedValue(double lon, double lat) const;

// Direct grid access
double getValue(int i, int j) const;  // i=lon index, j=lat index

// Check if position is in grid
bool isPointInGrid(double lon, double lat) const;
```

---

### DataManager

**File**: `src/DataManager.h`, `src/DataManager.cpp`  
**Inherits**: `QObject`

Central coordinator for all data sources.

#### Methods

```cpp
// GRIB management
void setGrib(Grib *grib);
Grib* getGrib();

// Current time for forecast display
void setCurrentDate(time_t date);
time_t getCurrentDate();

// Color schemes
DataColors* getDataColors();
```

---

## Navigation & Routing

### boat (Abstract Base)

**File**: `src/boat.h`, `src/boat.cpp`  
**Inherits**: `QGraphicsWidget`  
**Subclasses**: `boatVLM`, `boatReal`

Base class for all boat types.

#### Constructor

```cpp
boat(QString pseudo, bool activated,
     Projection *proj, MainWindow *main, myCentralWidget *parent);
```

#### Position & Motion

```cpp
double getLat() const;        // Current latitude
double getLon() const;        // Current longitude
QPointF getPosition() const;  // As QPointF(lon, lat)

double getSpeed() const;      // Speed in knots
double getHeading() const;    // Heading in degrees true
double getAvg() const;        // Average speed
```

#### Wind Data

```cpp
double getWindDir() const;    // Wind direction (degrees true)
double getWindSpeed() const;  // Wind speed (knots)
double getTWA() const;        // True Wind Angle (degrees)
```

#### Navigation

```cpp
double getWPLat() const;      // Waypoint latitude
double getWPLon() const;      // Waypoint longitude
QPointF getWP() const;        // Waypoint position
double getWPHd() const;       // Heading to waypoint

virtual void setWP(QPointF point, double w);  // Set waypoint
```

#### Performance

```cpp
Polar* getPolarData();        // Boat's polar diagram
QString getPolarName();       // Polar file name

// Get optimal upwind/downwind angles for wind speed
double getBvmgUp(double windSpeed);
double getBvmgDown(double windSpeed);
```

#### Display

```cpp
void setStatus(bool activated);     // Show/hide boat
bool getStatus() const;             // Visible?
bool getIsSelected() const;         // Is selected?
void setZoom(double zoom);          // Scale for display
```

---

### boatVLM

**File**: `src/boatVLM.h`, `src/boatVLM.cpp`  
**Inherits**: `boat`

Virtual boat for VLM online racing.

#### VLM-Specific

```cpp
virtual int getId();          // VLM boat ID
void setVlmSync(bool sync);   // Enable/disable server sync

// Race information
int getRank();                // Current ranking
QString getScore();           // Race score/points
```

---

### boatReal

**File**: `src/boatReal.h`, `src/boatReal.cpp`  
**Inherits**: `boat`

Real boat with GPS/NMEA tracking.

#### GPS

```cpp
void updateFromNMEA(QString nmeaSentence);  // Parse NMEA data
void setSerialPort(QString portName);       // Connect to GPS
```

---

### POI

**File**: `src/POI.h`, `src/POI.cpp`  
**Inherits**: `QObject`

Point of Interest (waypoint, mark, destination).

#### Constructor

```cpp
POI(QString name, double lat, double lon, int type);
```

#### Properties

```cpp
QString getName() const;
QString getDescription() const;

double getLat() const;
double getLon() const;
QPointF getPosition() const;

int getType() const;         // POI_WP, POI_MARK, etc.

bool hasInstruction() const; // Auto-pilot instruction attached?
```

#### Constants

```cpp
enum POIType {
    POI_WP,          // Waypoint
    POI_WPL,         // Waypoint with label
    POI_NM,          // Navigation mark
    POI_MOB,         // Man overboard
    POI_DEST         // Destination
};
```

---

### ROUTE

**File**: `src/route.h`, `src/route.cpp`

Simple route (ordered list of waypoints).

#### Methods

```cpp
void addPOI(POI *poi);           // Add waypoint to end
void insertPOI(int index, POI *poi);  // Insert at position
void removePOI(int index);       // Remove waypoint
POI* getPOI(int index);          // Get waypoint

int count() const;               // Number of waypoints
QList<POI*> getPoiList();        // All waypoints

double getTotalDistance();       // Route length (nm)
```

---

### routage

**File**: `src/routage.h`, `src/routage.cpp`

Weather routing optimizer (isochrone method).

#### Methods

```cpp
// Calculate optimal route
ROUTE* calculate(POI *start, POI *end, 
                time_t startTime, Grib *grib, Polar *polar);

// Calculation parameters
void setTimeStep(int hours);     // Isochrone time interval
void setMaxDuration(int hours);  // Maximum route duration
void setWindMargin(double deg);  // Tacking margin
```

---

## Graphics & UI

### MyView

**File**: `src/MyView.h`, `src/MyView.cpp`  
**Inherits**: `QGraphicsView`

Main map display.

#### Methods

```cpp
// Drawing
void drawBackground(QPainter *painter, const QRectF &rect) override;
void drawForeground(QPainter *painter, const QRectF &rect) override;

// User interaction
void mousePressEvent(QMouseEvent *event) override;
void mouseMoveEvent(QMouseEvent *event) override;
void wheelEvent(QWheelEvent *event) override;

// Projection
Projection* getProjection();
```

---

### MapDataDrawer

**File**: `src/MapDataDrawer.h`, `src/MapDataDrawer.cpp`

Renders map data (coastlines, GRIB overlays, etc.).

#### Methods

```cpp
// Draw coastlines
void drawGshhsData(QPainter *painter, Projection *proj);

// Draw weather data
void drawGribData(QPainter *painter, Projection *proj, 
                 Grib *grib, time_t currentTime);

// Draw wind arrows
void drawWindArrows(QPainter *painter, Projection *proj,
                   Grib *grib, time_t currentTime);

// Draw isobars
void drawIsobars(QPainter *painter, Projection *proj,
                std::list<IsoLine*> *isobars);
```

---

### MenuBar

**File**: `src/MenuBar.h`, `src/MenuBar.cpp`  
**Inherits**: `QMenuBar`

Application menu.

#### Structure

```cpp
// File menu
QMenu *fileMenu;
QAction *openGribAction;
QAction *closeGribAction;
QAction *quitAction;

// View menu
QMenu *viewMenu;
QAction *zoomInAction;
QAction *zoomOutAction;
QAction *zoomAllAction;

// Boat menu
QMenu *boatMenu;
// ... boat actions

// Route menu
QMenu *routeMenu;
// ... route actions
```

---

### ToolBar

**File**: `src/ToolBar.h`, `src/ToolBar.cpp`  
**Inherits**: `QToolBar`

Quick access toolbar.

---

### StatusBar

**File**: `src/StatusBar.h`, `src/StatusBar.cpp`  
**Inherits**: `QStatusBar`

Status information display.

#### Methods

```cpp
// Display boat information
void updateBoatInfo(boat *b);

// Display cursor position
void updateCursorInfo(double lat, double lon);

// Display weather info
void updateWeatherInfo(double windSpeed, double windDir);
```

---

## Utility Classes

### Util

**File**: `src/Util.h`, `src/Util.cpp`

Static utility functions.

#### Methods

```cpp
// File operations
static QString currentPath();
static bool fileExists(QString path);

// Geographic calculations
static double distance(double lat1, double lon1, 
                      double lat2, double lon2);  // Returns nm

static double heading(double lat1, double lon1,
                     double lat2, double lon2);   // Returns degrees

// Angle normalization
static double A360(double angle);  // Normalize to [0, 360)
static double A180(double angle);  // Normalize to [-180, 180)

// Coordinate conversion
static void degToDecimal(int deg, int min, double sec,
                        double *decimal);
```

---

### Polar

**File**: `src/Polar.h`, `src/Polar.cpp`

Boat performance (polar diagram).

#### Methods

```cpp
// Load from file
bool loadFromFile(QString filename);

// Get boat speed for wind conditions
double getSpeed(double windSpeed, double twa);  // twa = true wind angle

// Get optimal angles
double getBvmgUp(double windSpeed);     // Best upwind angle
double getBvmgDown(double windSpeed);   // Best downwind angle
```

#### Polar File Format

```
# Polar diagram
# TWS(knots)  TWA(degrees)  Speed(knots)
6.0   30    4.2
6.0   45    5.1
6.0   60    5.8
...
```

---

### Settings

**File**: `src/settings.h`, `src/settings.cpp`

Application settings (wraps QSettings).

#### Methods

```cpp
// Initialize
static void initSettings();

// Get/Set
static QVariant getSetting(QString key, QVariant defaultValue);
static void setSetting(QString key, QVariant value);

// Settings categories
static QString getVlmUser();
static void setVlmUser(QString user);

static QString getDefaultPolar();
static void setDefaultPolar(QString polar);
```

---

## Common Patterns

### Signal/Slot Usage

```cpp
// Define signal in class
class MyClass : public QObject {
    Q_OBJECT
signals:
    void dataChanged(int newValue);
};

// Connect in another class
connect(myObject, &MyClass::dataChanged,
        this, &ThisClass::onDataChanged);

// Or use lambda
connect(myObject, &MyClass::dataChanged,
        [](int value) {
            qDebug() << "Data changed to" << value;
        });
```

---

### Memory Management

```cpp
// Objects with parent are auto-deleted
MyWidget *widget = new MyWidget(parentWidget);
// No need to delete

// Objects without parent need management
QTimer *timer = new QTimer();
connect(timer, &QTimer::timeout, this, &MyClass::onTimeout);
// Later:
timer->deleteLater();  // Safe deletion
```

---

### Coordinate Handling

```cpp
// Always use Projection for conversions
double lon = -30.5, lat = 48.2;  // Geographic
int x, y;                         // Screen
projection->map2screen(lon, lat, &x, &y);

// Check visibility before drawing
if (projection->isPointVisible(lon, lat)) {
    painter->drawPoint(x, y);
}
```

---

### GRIB Data Access

```cpp
// Get weather at boat position
boat *b = mainWindow->getSelectedBoat();
double lat = b->getLat();
double lon = b->getLon();
time_t now = QDateTime::currentDateTimeUtc().toTime_t();

Grib *grib = dataManager->getGrib();
if (grib) {
    double windSpeed, windDir;
    bool ok = grib->interpolateValue_2D(
        lon, lat, now, &windSpeed, &windDir
    );
    if (ok) {
        qDebug() << "Wind:" << windSpeed << "kts @" << windDir << "°";
    }
}
```

---

### Drawing on Map

```cpp
void MyClass::drawCustomOverlay(QPainter *painter, Projection *proj)
{
    // Example: Draw a line between two points
    double lon1 = -10.0, lat1 = 45.0;
    double lon2 = -5.0, lat2 = 48.0;
    
    int x1, y1, x2, y2;
    proj->map2screen(lon1, lat1, &x1, &y1);
    proj->map2screen(lon2, lat2, &x2, &y2);
    
    painter->setPen(QPen(Qt::red, 2));
    painter->drawLine(x1, y1, x2, y2);
    
    // Draw label
    painter->drawText(x2 + 5, y2, "Destination");
}
```

---

## Error Handling

### Best Practices

```cpp
// Check pointers before use
boat *b = mainWindow->getSelectedBoat();
if (b) {
    double lat = b->getLat();
    // Use lat...
} else {
    qWarning() << "No boat selected";
}

// Check file operations
if (!grib->loadFile(fileName)) {
    qWarning() << "Failed to load GRIB:" << fileName;
    QMessageBox::warning(this, tr("Error"),
        tr("Could not load GRIB file."));
    return false;
}

// Use qDebug, qWarning, qCritical for logging
qDebug() << "Debug info";
qWarning() << "Warning message";
qCritical() << "Critical error";
```

---

## Threading Considerations

Most of qtVlm runs in the main GUI thread. When performing long operations:

```cpp
// Use QtConcurrent for background tasks
QFuture<void> future = QtConcurrent::run([this]() {
    // Long operation
    this->calculateRoute();
});

// Or use QTimer for periodic updates
QTimer *timer = new QTimer(this);
connect(timer, &QTimer::timeout, this, &MyClass::update);
timer->start(1000);  // Every second
```

---

## Further Reading

- **Source Code**: The definitive reference
- **Qt Documentation**: https://doc.qt.io/
- **ARCHITECTURE.md**: System design overview
- **CODE_ASSESSMENT.md**: Code quality insights

---

## Contributing to Documentation

Found an error or omission? Please help improve this documentation!

1. Fix or add documentation in source files
2. Update this API reference
3. Submit a pull request

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

---

*This API reference is a living document. As the code evolves, so should this documentation. Thank you for helping keep it up to date!*
