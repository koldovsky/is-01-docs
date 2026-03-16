# Onboarding Documentation — android-compose-mvvm-foodies

## Repository

This documentation is written for the following project:

**GitHub repository:**  
https://github.com/catalinghita8/android-compose-mvvm-foodies

The repository contains a sample Android application demonstrating modern Android development practices using Kotlin and Jetpack Compose.

---

# Project Overview

**Foodies** is a sample Android application built with modern Android technologies and architecture patterns.  
The project demonstrates how to build a scalable Android application using **Kotlin**, **Jetpack Compose**, **MVVM**, and **dependency injection with Hilt**.

The application fetches food categories from a remote API and displays them in a modern Compose-based UI. Users can browse categories and open detailed information about selected items.

The project is intentionally designed to demonstrate modern Android architecture and code organization rather than complex business logic.

### Main Goals of the Project

The repository demonstrates several important Android engineering practices:

- modern **Jetpack Compose UI architecture**
- **MVVM architecture pattern**
- **dependency injection using Hilt**
- **unidirectional data flow**
- **network data fetching with Retrofit**
- **asynchronous programming using Kotlin Coroutines**
- **clean separation between UI and data layers**

This repository can be used as a reference for building modern Android applications.

---

# Tech Stack

The project is built using the following technologies and libraries:

### Programming Language

- **Kotlin**

The entire project is implemented in Kotlin.

### UI Framework

- **Jetpack Compose**

Compose is used for building declarative UI components instead of traditional XML layouts.

### Architecture Pattern

- **MVVM (Model–View–ViewModel)**

The project separates UI logic, business logic, and data sources.

### Dependency Injection

- **Hilt**

Hilt manages dependency creation and injection across the application.

### Networking

- **Retrofit**

Retrofit is used to perform HTTP requests and parse API responses.

### Concurrency

- **Kotlin Coroutines**
- **Kotlin Flow**

These technologies are used for asynchronous data loading and reactive streams.

### Navigation

- **Navigation Compose**

Handles navigation between application screens.

### Image Loading

- **Coil**

Used for efficient image loading inside Compose UI.

### Build System

- **Gradle**

The project uses the Gradle build system with Android Gradle Plugin.

---

# High-Level Architecture

The project follows the **MVVM architecture pattern** with a unidirectional data flow.

High-level architecture:

```
User
 ↓
Compose UI
 ↓
ViewModel
 ↓
Repository
 ↓
Remote API
```

### Responsibilities of Each Layer

| Layer | Responsibility |
|-----|-----|
| UI (Compose) | Displays application state |
| ViewModel | Handles business logic and state |
| Repository | Provides data to ViewModel |
| API | Remote data source |

This architecture ensures clear separation of concerns and improves maintainability.

---

# Project Structure

The project is implemented as a **single-module Android application**.

Main directory structure:

```
app/
 └─ src/main/java/com/codingtroops/foodies
     ├─ di/
     ├─ model/
     ├─ ui/
     │   ├─ feature/
     │   │   ├─ categories/
     │   │   └─ category_details/
     │   ├─ theme/
     │   └─ EntryPointActivity.kt
     ├─ FoodiesApp.kt
     └─ Extensions.kt
```

Below is a detailed explanation of each component.

---

# Dependency Injection Layer

Directory:

```
di/
```

This directory contains dependency injection modules defined using **Hilt**.

Responsibilities of this layer:

- create API clients
- provide repositories
- manage singleton objects
- supply dependencies to ViewModels

Using dependency injection improves modularity and testability.

---

# Data Models

Directory:

```
model/
```

This folder contains **data classes** representing objects used in the application.

Examples include API response models and UI state models.

---

# UI Layer

Directory:

```
ui/
```

The UI layer is implemented entirely using **Jetpack Compose**.

This layer contains:

- composable UI functions
- feature screens
- navigation configuration
- UI themes

---

# Feature-Based Organization

Inside the UI layer, the project follows **feature-based structure**.

Example:

```
ui/feature/
   categories/
   category_details/
```

Each feature contains its own:

- screens
- ViewModels
- UI state models

This structure helps isolate functionality and makes the application easier to scale.

---

# Application Entry Points

## FoodiesApp.kt

This file defines the **Application class**.

Responsibilities:

- initializes global configuration
- integrates Hilt with the application
- prepares application-wide dependencies

---

## EntryPointActivity.kt

This is the **main Activity** of the application.

Responsibilities:

- sets the Compose content
- initializes navigation
- hosts application screens

---

# Navigation Architecture

Navigation is implemented using **Navigation Compose**.

Main navigation flow:

```
Categories Screen
        ↓
Category Details Screen
```

---

# UI System (Jetpack Compose)

The UI is fully implemented using **Jetpack Compose**, a modern declarative UI framework.

Key Compose concepts used in the project:

- composable functions
- state hoisting
- reactive UI updates
- ViewModel state observation
- Material design components

Theme configuration is located in:

```
ui/theme/
```

---

# Data Flow

The project follows **unidirectional data flow**.

```
User Interaction
        ↓
Composable UI
        ↓
ViewModel
        ↓
Repository
        ↓
Remote API
        ↑
     Response
```

---

# Networking

Networking is implemented using **Retrofit**.

Responsibilities include performing HTTP requests, parsing JSON responses, and mapping responses to Kotlin models.

---

# Image Loading

The project uses **Coil** to load images efficiently inside Compose UI.

Example:

```
AsyncImage(
    model = imageUrl,
    contentDescription = null
)
```

---

# Running the Project

### Requirements

- Android Studio
- JDK 17
- Android SDK
- Gradle (via wrapper)

### Steps

Clone the repository:

```
git clone https://github.com/catalinghita8/android-compose-mvvm-foodies
```

Open the project in **Android Studio** and run it on an emulator or physical device.

---

# Debugging and Development Tools

Useful tools:

- Logcat
- Network Inspector
- Layout Inspector
- Compose Preview

Common debugging points include ViewModel state updates, API responses, navigation arguments, and UI recompositions.

---

# Summary

The **Foodies** project demonstrates modern Android architecture using Kotlin, Jetpack Compose, MVVM, and Hilt.

The repository shows:

- clear separation of concerns
- declarative UI with Compose
- scalable architecture
- feature-based organization

This makes the project a useful reference implementation for Android developers learning modern Android architecture.
