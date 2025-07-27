# Cycling Helper - Development Guidelines

This document provides essential information for developers working on the Cycling Helper app, a Kotlin Multiplatform Mobile (KMM) application for cyclists that follows Google Material 3 Expressive UI guidelines.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Build and Configuration Instructions](#build-and-configuration-instructions)
3. [Testing Information](#testing-information)
4. [Development Guidelines](#development-guidelines)
5. [Architecture](#architecture)

## Project Overview

Cycling Helper is a cross-platform mobile application for cyclists with the following key features:
- Road navigation
- Route planning
- Trip recordings management

The app is built using Kotlin Multiplatform Mobile (KMM) to share code between Android and iOS platforms, with platform-specific UI implementations using Jetpack Compose for Android and SwiftUI for iOS.

## Build and Configuration Instructions

### Prerequisites

- Android Studio Arctic Fox (2021.3.1) or newer
- Xcode 13.0 or newer (for iOS development)
- JDK 17
- Kotlin 1.9.20 or newer
- Gradle 8.0 or newer
- Android SDK 34
- Cocoapods (for iOS development)

### Environment Setup

1. **Android Setup**:
   - Install Android Studio
   - Install Android SDK (API level 34)
   - Configure Android SDK location in `local.properties`:
     ```properties
     sdk.dir=/path/to/your/android/sdk
     ```

2. **iOS Setup** (for macOS users):
   - Install Xcode
   - Install Cocoapods: `sudo gem install cocoapods`

### Building the Project

#### From Android Studio

1. Open the project in Android Studio
2. Sync the project with Gradle files
3. Select the desired build variant and target device
4. Click "Run" to build and run the application

#### From Command Line

- **Build the entire project**:
  ```bash
  ./gradlew build
  ```

- **Build Android app**:
  ```bash
  ./gradlew :androidApp:assembleDebug
  ```

- **Build iOS app** (macOS only):
  ```bash
  ./gradlew :shared:linkDebugFrameworkIosX64
  cd iosApp && pod install
  ```

### Project Structure

```
CyclingHelper/
├── androidApp/                  # Android-specific code
│   ├── src/
│   │   └── main/
│   │       ├── java/            # Android app code
│   │       ├── res/             # Android resources
│   │       └── AndroidManifest.xml
│   └── build.gradle.kts         # Android app build script
├── shared/                      # Shared code for all platforms
│   ├── src/
│   │   ├── commonMain/          # Common code for all platforms
│   │   ├── commonTest/          # Common tests
│   │   ├── androidMain/         # Android-specific implementations
│   │   ├── androidTest/         # Android-specific tests
│   │   ├── iosMain/             # iOS-specific implementations
│   │   └── iosTest/             # iOS-specific tests
│   └── build.gradle.kts         # Shared module build script
├── iosApp/                      # iOS-specific code (when created)
├── build.gradle.kts             # Root build script
└── settings.gradle.kts          # Gradle settings
```

## Testing Information

### Test Structure

The project follows a standard testing approach with:

- **Unit Tests**: Test individual components in isolation
- **Integration Tests**: Test interactions between components
- **UI Tests**: Test the user interface

Tests are organized in the same structure as the source code:

- `shared/src/commonTest/`: Tests for common code
- `shared/src/androidTest/`: Android-specific tests
- `shared/src/iosTest/`: iOS-specific tests
- `androidApp/src/test/`: Android app unit tests
- `androidApp/src/androidTest/`: Android app instrumented tests

### Running Tests

#### From Android Studio

1. Right-click on a test file or directory
2. Select "Run" or "Debug"

#### From Command Line

- **Run all tests**:
  ```bash
  ./gradlew allTests
  ```

- **Run Android unit tests**:
  ```bash
  ./gradlew :androidApp:testDebugUnitTest
  ```

- **Run Android instrumented tests**:
  ```bash
  ./gradlew :androidApp:connectedDebugAndroidTest
  ```

- **Run shared module tests for a specific target**:
  ```bash
  ./gradlew :shared:iosX64Test        # iOS simulator (Intel)
  ./gradlew :shared:iosSimulatorArm64Test  # iOS simulator (Apple Silicon)
  ```

### Writing Tests

#### Example Test

Here's an example of a test for the `RouteRepository` implementation:

```kotlin
class MockRouteRepositoryTest {
    
    @Test
    fun testGetAllRoutes() = runTest {
        // Arrange
        val repository = MockRouteRepository()
        
        // Act
        val routes = repository.getAllRoutes().first()
        
        // Assert
        assertEquals(3, routes.size, "Repository should be initialized with 3 sample routes")
        assertEquals("City Park Loop", routes[0].name, "First route should be City Park Loop")
    }
    
    @Test
    fun testSaveNewRoute() = runTest {
        // Arrange
        val repository = MockRouteRepository()
        val newRoute = Route(
            id = "",
            name = "Test Route",
            description = "A test route",
            distance = 10.0,
            elevationGain = 100,
            estimatedDuration = 60,
            difficulty = RouteDifficulty.MODERATE,
            waypoints = listOf(
                Waypoint(37.7749, -122.4194, 10.0, "Start"),
                Waypoint(37.7750, -122.4200, 12.0, "End")
            )
        )
        
        // Act
        val savedRoute = repository.saveRoute(newRoute)
        val routes = repository.getAllRoutes().first()
        
        // Assert
        assertNotNull(savedRoute.id, "Saved route should have an ID")
        assertTrue(savedRoute.id.isNotBlank(), "Saved route ID should not be blank")
        assertEquals(4, routes.size, "Repository should now have 4 routes")
    }
}
```

#### Testing Best Practices

1. **Follow the AAA pattern**: Arrange, Act, Assert
2. **Test one thing per test**: Each test should verify a single behavior
3. **Use descriptive test names**: Names should describe what the test is verifying
4. **Use appropriate assertions**: Choose the right assertion for the condition being tested
5. **Mock dependencies**: Use mock objects to isolate the component being tested
6. **Test edge cases**: Include tests for boundary conditions and error scenarios
7. **Keep tests independent**: Tests should not depend on each other

## Development Guidelines

### Code Style

The project follows the official [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html) with the following specifics:

1. **Indentation**: 4 spaces (no tabs)
2. **Maximum line length**: 120 characters
3. **Imports**: No wildcard imports, organize imports alphabetically
4. **Naming conventions**:
   - Classes: PascalCase
   - Functions/Properties: camelCase
   - Constants: UPPER_SNAKE_CASE
   - File names: Match the name of the main class/function
5. **Documentation**: All public APIs should have KDoc comments

### Material 3 Expressive UI Guidelines

The app strictly follows Google's Material 3 Expressive UI guidelines:

1. **Dynamic color**: Adapt to user's wallpaper and preferences
2. **Typography**: Use the Material 3 type scale
3. **Shape**: Apply appropriate shape styles to components
4. **Motion**: Implement natural and intuitive motion patterns
5. **Accessibility**: Ensure the app is usable by everyone

### Multiplatform Development

When developing for multiple platforms:

1. **Maximize code sharing**: Put as much logic as possible in the common module
2. **Platform-specific APIs**: Use the `expect/actual` pattern for platform-specific implementations
3. **UI**: Implement UI separately for each platform using the native UI toolkit
4. **Resources**: Store platform-specific resources in the appropriate directories
5. **Testing**: Test common code in the common module, platform-specific code in platform modules

## Architecture

The app follows a clean architecture approach with the following layers:

### Data Layer

- **Models**: Data classes representing domain entities
- **Repositories**: Interfaces and implementations for data access
- **Data Sources**: Local and remote data sources

### Domain Layer

- **Use Cases**: Business logic components
- **Domain Models**: Core business entities
- **Repository Interfaces**: Abstractions for data access

### Presentation Layer

- **ViewModels**: Manage UI state and handle user interactions
- **UI Models**: Data classes specifically for UI representation
- **UI Components**: Composable functions (Android) or SwiftUI views (iOS)

### Dependency Injection

The app uses Koin for dependency injection:

1. **Module definitions**: Group related dependencies
2. **Scopes**: Control the lifecycle of dependencies
3. **Qualifiers**: Distinguish between similar types

Example:
```kotlin
val appModule = module {
    // Repositories
    single<RouteRepository> { MockRouteRepository() }
    
    // ViewModels
    viewModel { RouteListViewModel(get()) }
}
```