# qtVlm Modularization Plan

> **Purpose**: This document outlines a strategy to refactor qtVlm into smaller, more maintainable, and reusable modules. Modularization improves code quality, testability, and makes the codebase more welcoming to new contributors.

---

## Executive Summary

### Current State

qtVlm has several large, monolithic classes that handle multiple responsibilities:

| File | Lines | Issues |
|------|-------|--------|
| `routage.cpp` | 5,146 | Complex routing algorithm, hard to test |
| `mycentralwidget.cpp` | 4,907 | Mixing UI and business logic |
| `MainWindow.cpp` | 2,699 | God object - coordinates everything |
| `route.cpp` | 2,163 | Route + UI + file I/O mixed |
| `POI.cpp` | 2,117 | POI data + UI + management |

### Benefits of Modularization

✅ **Maintainability**: Smaller files are easier to understand and modify  
✅ **Testability**: Independent modules can be tested in isolation  
✅ **Reusability**: Modules can be reused across the application  
✅ **Collaboration**: Multiple developers can work on different modules  
✅ **Onboarding**: New contributors can focus on specific modules  

---

## Modularization Strategy

### Phase 1: Documentation & Analysis (Completed)
- ✅ Document current architecture
- ✅ Identify large, complex files
- ✅ Analyze dependencies between components
- ✅ Create modularization plan (this document)

### Phase 2: Create Module Structure (Next)
- Define module boundaries
- Create module directories
- Establish module interfaces
- Document module responsibilities

### Phase 3: Extract Business Logic
- Separate business logic from UI
- Create service classes
- Implement dependency injection
- Add unit tests for extracted logic

### Phase 4: Refactor Large Classes
- Break down God objects
- Apply Single Responsibility Principle
- Introduce design patterns where appropriate
- Maintain backward compatibility

### Phase 5: Establish Module Guidelines
- Document module conventions
- Create templates for new modules
- Set up automated checks (linting)
- Update contribution guidelines

---

## Proposed Module Structure

```
src/
├── core/                    # Core application logic
│   ├── Application.h/cpp    # Application lifecycle
│   ├── Settings.h/cpp       # Settings management (exists)
│   └── Version.h            # Version info (exists)
│
├── models/                  # Data models (pure business logic)
│   ├── boat/
│   │   ├── BoatModel.h/cpp         # Base boat data
│   │   ├── BoatVLMModel.h/cpp      # VLM-specific data
│   │   ├── BoatRealModel.h/cpp     # Real boat data
│   │   └── BoatPerformance.h/cpp   # Performance calculations
│   │
│   ├── weather/
│   │   ├── GribModel.h/cpp         # GRIB data abstraction
│   │   ├── WeatherPoint.h/cpp      # Weather at a point
│   │   └── WeatherInterpolator.h/cpp
│   │
│   ├── navigation/
│   │   ├── Position.h/cpp          # Geographic position
│   │   ├── Waypoint.h/cpp          # Single waypoint
│   │   ├── Route.h/cpp             # Route data structure
│   │   └── Track.h/cpp             # Historical track
│   │
│   └── polar/
│       ├── PolarDiagram.h/cpp      # Polar data
│       └── PolarReader.h/cpp       # Polar file parsing
│
├── services/                # Business logic services
│   ├── weather/
│   │   ├── GribLoader.h/cpp        # GRIB file loading
│   │   ├── GribCache.h/cpp         # Weather data caching
│   │   └── WeatherService.h/cpp    # Weather operations
│   │
│   ├── routing/
│   │   ├── RoutingEngine.h/cpp     # Core routing algorithm
│   │   ├── IsochroneCalculator.h/cpp
│   │   ├── RouteOptimizer.h/cpp
│   │   └── RoutingService.h/cpp    # High-level routing API
│   │
│   ├── navigation/
│   │   ├── NavigationService.h/cpp # Navigation calculations
│   │   ├── OrthodromicCalc.h/cpp   # Great circle math
│   │   └── DistanceCalculator.h/cpp
│   │
│   └── network/
│       ├── VLMClient.h/cpp         # VLM API client
│       ├── GribDownloader.h/cpp    # Weather downloads
│       └── NetworkManager.h/cpp    # HTTP operations
│
├── ui/                      # User interface components
│   ├── mainwindow/
│   │   ├── MainWindow.h/cpp        # Main window (simplified)
│   │   ├── MainWindowActions.cpp   # Action handlers
│   │   └── MainWindowState.cpp     # State management
│   │
│   ├── widgets/
│   │   ├── map/
│   │   │   ├── MapView.h/cpp       # Map display
│   │   │   ├── MapRenderer.h/cpp   # Rendering logic
│   │   │   └── MapController.h/cpp # Map interaction
│   │   │
│   │   ├── boat/
│   │   │   ├── BoatWidget.h/cpp    # Boat UI component
│   │   │   └── BoatInfoPanel.h/cpp
│   │   │
│   │   └── weather/
│   │       ├── WeatherPanel.h/cpp
│   │       └── GribTimeLine.h/cpp
│   │
│   ├── dialogs/
│   │   └── (existing Dialog* files)
│   │
│   └── components/
│       ├── MenuBarComponent.h/cpp
│       ├── ToolBarComponent.h/cpp
│       └── StatusBarComponent.h/cpp
│
├── graphics/                # Graphics and rendering
│   ├── projection/
│   │   ├── Projection.h/cpp        # (exists)
│   │   ├── MercatorProjection.h/cpp
│   │   └── ProjectionFactory.h/cpp
│   │
│   ├── renderers/
│   │   ├── CoastlineRenderer.h/cpp
│   │   ├── WeatherRenderer.h/cpp
│   │   ├── RouteRenderer.h/cpp
│   │   └── BoatRenderer.h/cpp
│   │
│   └── items/
│       ├── BoatGraphicsItem.h/cpp
│       ├── RouteGraphicsItem.h/cpp
│       └── POIGraphicsItem.h/cpp
│
├── io/                      # File I/O operations
│   ├── grib/
│   │   ├── GribReader.h/cpp
│   │   ├── GribWriter.h/cpp
│   │   └── GribParser.h/cpp
│   │
│   ├── gpx/
│   │   ├── GPXReader.h/cpp
│   │   ├── GPXWriter.h/cpp
│   │   └── GPXParser.h/cpp
│   │
│   └── polar/
│       ├── PolarReader.h/cpp
│       └── PolarWriter.h/cpp
│
├── utils/                   # Utility functions
│   ├── math/
│   │   ├── GeographicMath.h/cpp    # Geographic calculations
│   │   ├── AngleUtils.h/cpp        # Angle normalization
│   │   └── InterpolationUtils.h/cpp
│   │
│   ├── string/
│   │   ├── StringUtils.h/cpp
│   │   └── FormatUtils.h/cpp
│   │
│   └── file/
│       ├── FileUtils.h/cpp
│       └── PathUtils.h/cpp
│
└── libs/                    # Third-party libraries (existing)
```

---

## Detailed Refactoring Plans

### 1. Refactor MainWindow (God Object)

**Current Issues:**
- 2,699 lines doing everything
- Mixing UI, business logic, and coordination
- Hard to test and modify

**Solution: Extract Responsibilities**

#### 1.1 Create MainWindowController
```cpp
// src/ui/mainwindow/MainWindowController.h
class MainWindowController : public QObject
{
    Q_OBJECT
public:
    MainWindowController(MainWindow *view, QObject *parent = nullptr);
    
    // Delegate specific responsibilities
    void handleBoatSelection(boat *b);
    void handleGribOpen(QString filename);
    void handleRouteCreation();
    
signals:
    void boatSelectionChanged(boat *b);
    void gribLoaded(Grib *grib);
    
private:
    MainWindow *m_view;
    WeatherService *m_weatherService;
    NavigationService *m_navService;
    // ... other services
};
```

#### 1.2 Create Action Handlers
```cpp
// src/ui/mainwindow/MainWindowActions.cpp
// Move all slot implementations here
class MainWindowActions : public QObject
{
    Q_OBJECT
public:
    MainWindowActions(MainWindow *parent);
    
public slots:
    void onOpenGrib();
    void onCloseGrib();
    void onBoatSelect();
    // ... etc
    
private:
    MainWindow *m_mainWindow;
};
```

#### 1.3 Create State Manager
```cpp
// src/ui/mainwindow/MainWindowState.h
class MainWindowState
{
public:
    // Application state
    boat *selectedBoat() const;
    void setSelectedBoat(boat *b);
    
    Grib *currentGrib() const;
    void setCurrentGrib(Grib *g);
    
    // View state
    bool isBoatLocked() const;
    void setBoatLocked(bool locked);
    
private:
    boat *m_selectedBoat = nullptr;
    Grib *m_currentGrib = nullptr;
    bool m_boatLocked = false;
    // ... other state
};
```

---

### 2. Refactor routage.cpp (5,146 lines)

**Current Issues:**
- Monolithic routing algorithm
- Hard to understand and test
- Poor separation of concerns

**Solution: Extract Components**

#### 2.1 Create RoutingEngine
```cpp
// src/services/routing/RoutingEngine.h
class RoutingEngine
{
public:
    RoutingEngine(WeatherService *weather, PolarDiagram *polar);
    
    RouteResult calculate(const RoutingRequest &request);
    
private:
    WeatherService *m_weather;
    PolarDiagram *m_polar;
    IsochroneCalculator *m_isoCalculator;
};

// src/services/routing/RoutingRequest.h
struct RoutingRequest
{
    Position start;
    Position end;
    QDateTime startTime;
    RoutingOptions options;
};

struct RouteResult
{
    Route route;
    QDateTime estimatedArrival;
    double totalDistance;
    bool success;
    QString errorMessage;
};
```

#### 2.2 Extract Isochrone Calculator
```cpp
// src/services/routing/IsochroneCalculator.h
class IsochroneCalculator
{
public:
    IsochroneCalculator(WeatherService *weather, PolarDiagram *polar);
    
    // Calculate reachable positions after time interval
    QList<Position> calculateIsochrone(
        const Position &start,
        const QDateTime &startTime,
        int durationHours
    );
    
private:
    double calculateSpeed(double windSpeed, double windDir, 
                         double heading);
    QList<Position> expandPositions(const QList<Position> &current);
};
```

#### 2.3 Create Route Optimizer
```cpp
// src/services/routing/RouteOptimizer.h
class RouteOptimizer
{
public:
    Route optimize(const Route &initial, const RoutingOptions &options);
    
private:
    Route pruneInefficient(const Route &route);
    Route smoothPath(const Route &route);
};
```

---

### 3. Refactor mycentralwidget.cpp (4,907 lines)

**Current Issues:**
- Mixing graphics management and business logic
- Too many responsibilities

**Solution: Separate Graphics from Logic**

#### 3.1 Create MapView (Graphics)
```cpp
// src/ui/widgets/map/MapView.h
class MapView : public QGraphicsView
{
    Q_OBJECT
public:
    MapView(QWidget *parent = nullptr);
    
    void setProjection(Projection *proj);
    void addBoat(boat *b);
    void addRoute(Route *r);
    
protected:
    void drawBackground(QPainter *painter, const QRectF &rect) override;
    void drawForeground(QPainter *painter, const QRectF &rect) override;
    
private:
    Projection *m_projection;
    MapRenderer *m_renderer;
};
```

#### 3.2 Create MapController (Logic)
```cpp
// src/ui/widgets/map/MapController.h
class MapController : public QObject
{
    Q_OBJECT
public:
    MapController(MapView *view, QObject *parent = nullptr);
    
    void handleMousePress(QMouseEvent *event);
    void handleMouseMove(QMouseEvent *event);
    void handleWheel(QWheelEvent *event);
    
signals:
    void positionClicked(Position pos);
    void zoomChanged(double newZoom);
    
private:
    MapView *m_view;
    Projection *m_projection;
};
```

#### 3.3 Create MapRenderer
```cpp
// src/graphics/renderers/MapRenderer.h
class MapRenderer
{
public:
    void renderCoastline(QPainter *p, Projection *proj);
    void renderWeather(QPainter *p, Projection *proj, WeatherData *data);
    void renderRoutes(QPainter *p, Projection *proj, QList<Route*> routes);
    void renderBoats(QPainter *p, Projection *proj, QList<boat*> boats);
    
private:
    CoastlineRenderer *m_coastRenderer;
    WeatherRenderer *m_weatherRenderer;
    RouteRenderer *m_routeRenderer;
    BoatRenderer *m_boatRenderer;
};
```

---

### 4. Extract Weather Services

#### 4.1 Create WeatherService
```cpp
// src/services/weather/WeatherService.h
class WeatherService : public QObject
{
    Q_OBJECT
public:
    WeatherService(QObject *parent = nullptr);
    
    // Load weather data
    bool loadGribFile(const QString &filename);
    
    // Query weather
    WeatherPoint getWeatherAt(const Position &pos, const QDateTime &time);
    bool interpolateWind(const Position &pos, const QDateTime &time,
                        double *windSpeed, double *windDir);
    
    // Available times
    QList<QDateTime> availableTimes() const;
    
signals:
    void gribLoaded(Grib *grib);
    void gribLoadError(QString error);
    
private:
    Grib *m_grib = nullptr;
    GribCache *m_cache;
};
```

#### 4.2 Create Weather Models
```cpp
// src/models/weather/WeatherPoint.h
struct WeatherPoint
{
    Position position;
    QDateTime time;
    
    double windSpeed;      // knots
    double windDirection;  // degrees true
    double pressure;       // hPa
    double temperature;    // Celsius
    
    bool valid;
};
```

---

### 5. Extract Navigation Services

```cpp
// src/services/navigation/NavigationService.h
class NavigationService
{
public:
    // Distance calculations
    double calculateDistance(const Position &p1, const Position &p2);
    
    // Bearing calculations
    double calculateBearing(const Position &from, const Position &to);
    double calculateFinalBearing(const Position &from, const Position &to);
    
    // Waypoint navigation
    double distanceToWaypoint(const Position &current, const Position &wp);
    double bearingToWaypoint(const Position &current, const Position &wp);
    
    // Cross-track error
    double crossTrackError(const Position &current, 
                          const Position &start, 
                          const Position &end);
    
private:
    OrthodromicCalculator m_orthodromic;
};
```

---

## Module Interface Examples

### Service Interface Pattern

```cpp
// src/services/IService.h
class IService
{
public:
    virtual ~IService() = default;
    virtual bool initialize() = 0;
    virtual void shutdown() = 0;
};

// Example implementation
class WeatherService : public IService
{
public:
    bool initialize() override {
        // Setup cache, etc.
        return true;
    }
    
    void shutdown() override {
        // Cleanup
    }
};
```

### Dependency Injection

```cpp
// src/core/ServiceLocator.h
class ServiceLocator
{
public:
    static ServiceLocator* instance();
    
    void registerService(const QString &name, IService *service);
    
    template<typename T>
    T* getService(const QString &name) {
        return static_cast<T*>(m_services.value(name));
    }
    
private:
    QMap<QString, IService*> m_services;
};

// Usage
ServiceLocator::instance()->registerService("weather", new WeatherService());
auto weather = ServiceLocator::instance()->getService<WeatherService>("weather");
```

---

## Migration Strategy

### Approach: Gradual Refactoring

**Don't rewrite everything at once!** Instead:

1. **Add New Modules Alongside Old Code**
   - Create new modular structure
   - Keep old code working
   - Gradually migrate functionality

2. **Maintain Backward Compatibility**
   - Old classes can delegate to new services
   - No breaking changes for users

3. **Test Each Migration**
   - Ensure functionality remains the same
   - Add tests for new modules

4. **Document Migration Path**
   - Clear examples of old vs new
   - Migration guide for contributors

### Example Migration

**Before (Old Code):**
```cpp
// In MainWindow.cpp
void MainWindow::openGribFile(QString filename)
{
    Grib *grib = Grib::loadGrib(filename, dataManager);
    if (grib && grib->isOk()) {
        // 50+ lines of setup code...
        this->currentGrib = grib;
        updateUI();
    }
}
```

**After (New Modular Code):**
```cpp
// In MainWindowController.cpp
void MainWindowController::handleGribOpen(QString filename)
{
    bool success = m_weatherService->loadGribFile(filename);
    if (success) {
        emit gribLoaded(m_weatherService->currentGrib());
    } else {
        emit gribLoadError(m_weatherService->lastError());
    }
}

// In MainWindow.cpp (simplified)
void MainWindow::openGribFile(QString filename)
{
    m_controller->handleGribOpen(filename);
}
```

**Transition Period:**
```cpp
// In MainWindow.cpp (during migration)
void MainWindow::openGribFile(QString filename)
{
    // Use new service if available, fall back to old code
    if (m_weatherService) {
        m_controller->handleGribOpen(filename);
    } else {
        // Old code still works
        Grib *grib = Grib::loadGrib(filename, dataManager);
        // ... old implementation
    }
}
```

---

## Testing Strategy

### Unit Tests for Services

```cpp
// tests/services/WeatherServiceTest.cpp
class WeatherServiceTest : public QObject
{
    Q_OBJECT
    
private slots:
    void testLoadGrib() {
        WeatherService service;
        bool success = service.loadGribFile("test_data/sample.grb");
        QVERIFY(success);
    }
    
    void testInterpolation() {
        WeatherService service;
        service.loadGribFile("test_data/sample.grb");
        
        Position pos(-30.0, 48.0);
        QDateTime time = QDateTime::currentDateTimeUtc();
        
        double windSpeed, windDir;
        bool success = service.interpolateWind(pos, time, 
                                              &windSpeed, &windDir);
        QVERIFY(success);
        QVERIFY(windSpeed > 0);
    }
};
```

---

## Benefits Realized

### Immediate Benefits

✅ **Clearer Responsibilities**: Each module has one job  
✅ **Easier Testing**: Services can be tested independently  
✅ **Better Documentation**: Smaller files are easier to document  
✅ **Reduced Complexity**: Break down 5000-line files into 200-line modules  

### Long-Term Benefits

✅ **Easier Maintenance**: Changes isolated to specific modules  
✅ **Better Collaboration**: Multiple people can work on different modules  
✅ **Code Reuse**: Services can be used across the application  
✅ **Future-Proof**: Easier to add new features without breaking existing code  

---

## Implementation Timeline

### Phase 1: Foundation (1-2 weeks)
- Create directory structure
- Define service interfaces
- Create first service (WeatherService)
- Add basic tests

### Phase 2: Extract Services (3-4 weeks)
- RoutingEngine and components
- NavigationService
- Network services
- Test each service

### Phase 3: Refactor UI (3-4 weeks)
- Extract MainWindowController
- Separate MapView from logic
- Create widget components
- Update dialogs

### Phase 4: Polish (2-3 weeks)
- Remove old code (if fully migrated)
- Add comprehensive tests
- Update documentation
- Create migration guide

**Total Estimated Time: 9-13 weeks**

---

## Guidelines for New Code

### Single Responsibility Principle

Each class should have ONE reason to change:

```cpp
// ❌ Bad: Multiple responsibilities
class BoatManager {
    void loadBoat(QString file);     // I/O
    void drawBoat(QPainter *p);      // Graphics
    double calculateSpeed();         // Business logic
    void handleClick(QMouseEvent *e); // UI interaction
};

// ✅ Good: Single responsibility each
class BoatLoader {
    Boat* load(QString file);
};

class BoatRenderer {
    void render(QPainter *p, Boat *boat);
};

class BoatPerformance {
    double calculateSpeed(Boat *boat, Weather weather);
};

class BoatController {
    void handleClick(QMouseEvent *e);
};
```

### Dependency Injection

Don't create dependencies inside classes:

```cpp
// ❌ Bad: Hard-coded dependency
class RouteCalculator {
    RouteCalculator() {
        m_weather = new WeatherService();  // Hard to test!
    }
    WeatherService *m_weather;
};

// ✅ Good: Injected dependency
class RouteCalculator {
    RouteCalculator(WeatherService *weather)
        : m_weather(weather)  // Easy to mock for testing!
    {}
    WeatherService *m_weather;
};
```

### Interface Segregation

Create small, focused interfaces:

```cpp
// ❌ Bad: Monolithic interface
class IDataManager {
    virtual Grib* loadGrib() = 0;
    virtual void saveGrib() = 0;
    virtual Route* loadRoute() = 0;
    virtual void saveRoute() = 0;
    virtual Boat* loadBoat() = 0;
    virtual void saveBoat() = 0;
    // ... 20 more methods
};

// ✅ Good: Focused interfaces
class IGribLoader {
    virtual Grib* load(QString file) = 0;
};

class IRouteLoader {
    virtual Route* load(QString file) = 0;
};

class IBoatLoader {
    virtual Boat* load(QString file) = 0;
};
```

---

## Conclusion

Modularization is a journey, not a destination. This plan provides:

1. **Clear Structure**: Organized modules with defined responsibilities
2. **Gradual Migration**: No need to rewrite everything at once
3. **Testing Strategy**: Make the code testable and reliable
4. **Developer Guidelines**: Standards for future development

**Next Steps:**

1. Review and discuss this plan
2. Get team buy-in
3. Start with Phase 1 (Foundation)
4. Iterate and improve based on feedback

**Remember:** The goal is not perfection, but **continuous improvement**. Every small refactoring makes the codebase better!

---

## Questions & Feedback

Have suggestions for this plan? Want to help with modularization?

- Open an issue for discussion
- See [CONTRIBUTING.md](CONTRIBUTING.md) for how to contribute
- Check [ARCHITECTURE.md](ARCHITECTURE.md) for current structure

---

*"Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler*
