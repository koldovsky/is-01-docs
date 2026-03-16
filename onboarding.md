# Onboarding Guide — Android Clean Architecture (Kotlin)

> Repository: [android10/Android-CleanArchitecture-Kotlin](https://github.com/android10/Android-CleanArchitecture-Kotlin)  
> A movies sample app demonstrating Clean Architecture principles in Kotlin.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [Project Structure](#3-project-structure)
4. [Local Setup](#4-local-setup)
5. [Domain Layer Deep Dive](#5-domain-layer-deep-dive)
6. [Testing](#6-testing)
7. [Token Evaluation](#7-token-evaluation)
8. [Contributing & PR Workflow](#8-contributing--pr-workflow)
9. [Glossary](#9-glossary)

---

## 1. Project Overview

This repository is a reference Android application that shows how to apply **Clean Architecture** in a real Kotlin project. The app fetches popular movies from TheMovieDB API and displays them in a list with a detail screen.

Despite simple functionality, every structural decision is intentional and maps directly to production-grade Android work.

**What the app does:**

| Screen | Description |
|--------|-------------|
| Movies List | Fetches and displays popular movies in a RecyclerView |
| Movie Detail | Shows poster, title, overview and release date |
| Error / Empty | Failure state UI with retry support |

---

## 2. Architecture

The project follows Uncle Bob's **Clean Architecture** with three layers. Dependencies point inward — outer layers know about inner layers, never the reverse.

```
┌─────────────────────────────────────────────────────────┐
│                     PRESENTATION                        │
│   Fragment / Activity · ViewModel · UI models           │
└───────────────────┬─────────────────────────────────────┘
                    │ depends on
┌───────────────────▼─────────────────────────────────────┐
│                      DOMAIN  (pure Kotlin)              │
│   UseCase · Repository interface · Entity · Either      │
└───────────────────▲─────────────────────────────────────┘
                    │ implements
┌───────────────────┴─────────────────────────────────────┐
│                       DATA                              │
│   RepositoryImpl · NetworkDataSource · Retrofit · DTOs  │
└─────────────────────────────────────────────────────────┘
```

### Data flow (one request cycle)

1. User interacts with `Fragment`
2. Fragment calls `ViewModel` method
3. ViewModel executes a `UseCase` via `invoke` operator
4. UseCase calls `Repository` interface (domain layer)
5. `RepositoryImpl` (data layer) fetches from network
6. `Either<Failure, T>` propagates back through UseCase → ViewModel
7. ViewModel updates `LiveData` → Fragment re-renders UI

### Key patterns

| Pattern | Where | Purpose |
|---------|-------|---------|
| MVVM | Presentation | Separate UI from business logic |
| UseCase / Interactor | Domain | Single-responsibility business operations |
| Repository pattern | Domain ↔ Data | Abstraction over data sources |
| Either monad | Domain | Explicit error handling without exceptions |
| Coroutines | All layers | Async work with structured concurrency |

---

## 3. Project Structure

All code lives in a single `:app` Gradle module. Layers are separated by package, not by Gradle module — intentional trade-off to keep the sample small.

```
com.fernandocejas.sample
├── AndroidApplication.kt          # App class, DI setup
├── core/
│   ├── extension/                 # Kotlin extension functions
│   ├── functional/                # Either, None
│   └── interactor/                # Base UseCase<P, R>
└── features/
    └── movies/
        ├── data/                  # DATA LAYER
        │   ├── MovieRepositoryImpl.kt
        │   ├── source/            # NetworkDataSource
        │   └── entity/            # Network DTOs
        ├── domain/                # DOMAIN LAYER
        │   ├── Movie.kt
        │   ├── MovieRepository.kt (interface)
        │   └── usecase/           # GetMovies, GetMovieDetails
        └── view/                  # PRESENTATION LAYER
            ├── MoviesFragment.kt
            ├── MovieDetailsFragment.kt
            ├── MoviesViewModel.kt
            └── MoviesAdapter.kt
```

### Key files

| File | Responsibility |
|------|---------------|
| `core/interactor/UseCase.kt` | Abstract base; runs on IO dispatcher |
| `core/functional/Either.kt` | `Either<L,R>` — Left = failure, Right = success |
| `MovieRepository.kt` | Interface defining data contract (domain owns it) |
| `MovieRepositoryImpl.kt` | Concrete impl; delegates to NetworkDataSource |
| `MoviesService.kt` | Retrofit interface for TheMovieDB endpoints |
| `GetMovies.kt` | UseCase: fetches paginated movie list |
| `MoviesViewModel.kt` | Bridges UseCases and UI; exposes LiveData |
| `MoviesFragment.kt` | Observes ViewModel, drives RecyclerView |

---

## 4. Local Setup

### Prerequisites

| Tool | Version |
|------|---------|
| Android Studio | Hedgehog (2023.1.1) or newer |
| JDK | 17 (bundled with Android Studio) |
| Android SDK | API 34 compile / API 21 minimum |
| TheMovieDB API key | Free at [themoviedb.org](https://www.themoviedb.org/) |

### Clone & open

```bash
git clone https://github.com/android10/Android-CleanArchitecture-Kotlin.git
cd Android-CleanArchitecture-Kotlin
# File → Open in Android Studio
```

### API key

Add to `local.properties` (git-ignored):

```
API_KEY="your_key_here"
```

The key is injected via `buildConfigField` in `app/build.gradle`.

### Build & run

```bash
./gradlew assembleDebug
./gradlew installDebug
./gradlew runAcceptanceTests
./gradlew runStaticCodeAnalysis   # detekt
./gradlew runTestCoverage         # unit tests + JaCoCo report
```

---

## 5. Domain Layer Deep Dive

The domain layer has **zero Android imports** — it is pure Kotlin and trivially unit-testable.

### UseCase base class

```kotlin
abstract class UseCase<in Params, out Type> {
    abstract suspend fun run(params: Params): Either<Failure, Type>

    operator fun invoke(params: Params, onResult: (Either<Failure, Type>) -> Unit) {
        val job = CoroutineScope(IO).async { run(params) }
        CoroutineScope(Main).launch { onResult(job.await()) }
    }
}
```

### Either — explicit error handling

Every operation returns `Either<Failure, T>`. The caller must handle both paths at compile time:

```kotlin
getMovies(GetMovies.Params(1)) { result ->
    result.fold(::handleFailure, ::handleMovieList)
}
```

### Failure sealed class

```kotlin
sealed class Failure {
    object NetworkConnection : Failure()
    object ServerError : Failure()
    class FeatureFailure : Failure()  // extend per feature
}
```

---

## 6. Testing

### Test pyramid

| Level | Tools | What is covered |
|-------|-------|----------------|
| Unit | JUnit 4, MockK | UseCases, ViewModels, mappers, Either logic |
| Integration | JUnit 4, real Repository | Repository ↔ DataSource wiring |
| Acceptance (UI) | Espresso | Full user flows on emulator/device |
| Static analysis | Detekt | Code style, complexity, naming |

### Commands

```bash
./gradlew test                    # unit tests
./gradlew runAcceptanceTests      # instrumented (needs device/emulator)
./gradlew runTestCoverage         # unit tests + coverage report
```

---

## 7. Token Evaluation

Estimated token counts using cl100k_base tokeniser (~1 token = 4 chars of code).

| Component | Est. tokens | Notes |
|-----------|-------------|-------|
| `core/functional/Either.kt` | ~220 | Either + None + extensions |
| `core/interactor/UseCase.kt` | ~180 | Base class with coroutine dispatch |
| `core/extension/*.kt` (all) | ~600 | Fragment, View, Activity, Context extensions |
| `Movie.kt` (entity) | ~80 | Simple data class |
| `MovieRepository.kt` (interface) | ~90 | 3–4 suspend fun declarations |
| `GetMovies.kt` + `GetMovieDetails.kt` | ~300 | Two UseCases + Params inner classes |
| `MovieRepositoryImpl.kt` | ~350 | Impl wiring network + cache |
| `MoviesService.kt` (Retrofit) | ~150 | API interface with @GET annotations |
| `NetworkDataSource.kt` | ~280 | Retrofit call execution + error mapping |
| `MovieEntity.kt` (DTO) | ~120 | Network-layer data class |
| `MoviesViewModel.kt` | ~400 | LiveData, UseCase calls, state handling |
| `MoviesFragment.kt` | ~350 | RecyclerView setup, observer wiring |
| `MovieDetailsFragment.kt` | ~200 | Detail display logic |
| `MoviesAdapter.kt` | ~220 | RecyclerView adapter + DiffUtil |
| `AndroidApplication.kt` + DI | ~250 | App init, ServiceLocator |
| `build.gradle` files (all) | ~400 | Dependencies, buildConfig |
| Test files (unit + acceptance) | ~1 800 | ~15 test classes, avg ~120 tokens each |
| **TOTAL** | **~5 990** | Fits in a 16K context window |

The entire codebase can be passed to a modern LLM in a single request for review or refactoring.

---

## 8. Contributing & PR Workflow

### Fork & branch

```bash
# 1. Fork on GitHub (top-right button)
# 2. Clone your fork:
git clone https://github.com/<YOUR_USER>/Android-CleanArchitecture-Kotlin.git

# 3. Add upstream remote:
git remote add upstream https://github.com/android10/Android-CleanArchitecture-Kotlin.git

# 4. Create a feature branch:
git checkout -b docs/onboarding-guide
```

### Commit & push

```bash
git add docs/ONBOARDING.md
git commit -m "docs: add onboarding guide for new contributors"
git push origin docs/onboarding-guide
```

### Open PR

- **Base:** `android10/Android-CleanArchitecture-Kotlin` → `main`
- **Head:** `<YOUR_USER>/Android-CleanArchitecture-Kotlin` → `docs/onboarding-guide`

### PR checklist

- [ ] Title follows Conventional Commits (`docs:` / `feat:` / `fix:`)
- [ ] All existing tests pass (`./gradlew test`)
- [ ] No new detekt violations (`./gradlew runStaticCodeAnalysis`)
- [ ] Description explains what changed and why
- [ ] New code includes unit tests

---

## 9. Glossary

| Term | Definition |
|------|-----------|
| Clean Architecture | Layered design where business rules are isolated from frameworks and I/O |
| UseCase / Interactor | Class representing a single business operation; called by ViewModel |
| Either\<L, R\> | Functional type holding Left (failure) or Right (success) |
| Repository | Abstraction over data sources; interface in domain, impl in data |
| ViewModel | Architecture component surviving config changes; bridges domain and UI |
| LiveData | Lifecycle-aware observable holder; pushes state to Fragment |
| Coroutine | Kotlin's lightweight concurrency primitive |
| Detekt | Kotlin static analysis tool |
| DTO | Data Transfer Object — model used only at the network boundary |
| DI | Dependency Injection — supplying dependencies from outside the class |
```
