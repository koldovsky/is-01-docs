
# Token Evaluation — android-compose-mvvm-foodies

## Why Token Evaluation Matters

When using AI tools in software development, it is important to understand how large a project is in terms of tokens.  
Large codebases cannot be processed entirely by most models, so developers typically provide only relevant sections.

Approximation rules:

- 1 token ≈ 3–4 characters in English
- 100 lines of Kotlin code ≈ 800–1500 tokens

---

## Project Size Overview

The project consists of a single Android module.

Main directories analyzed:

```
app/src/main/java/com/codingtroops/foodies
```

Key packages:

- di
- model
- ui
- ui/feature/categories
- ui/feature/category_details
- ui/theme

---

## Token Estimation by Component

### Application Core

Files:

- FoodiesApp.kt
- EntryPointActivity.kt

Estimated size:

- ~150–250 lines of code
- ~1,500–2,500 tokens

Purpose:

Application initialization and entry point.

---

### Dependency Injection

Directory:

```
di/
```

Estimated size:

- ~100–150 lines
- ~800–1,200 tokens

Purpose:

Defines Hilt modules used for dependency injection.

---

### Data Models

Directory:

```
model/
```

Estimated size:

- ~150–250 lines
- ~1,000–2,000 tokens

Purpose:

Contains API models and domain models used throughout the application.

---

### UI Layer

Directory:

```
ui/
```

Includes:

- Activity
- Compose UI components
- navigation
- theme

Estimated size:

- ~600–900 lines
- ~5,000–8,000 tokens

This is the largest layer in the project.

---

### Feature: Categories

Directory:

```
ui/feature/categories
```

Estimated size:

- ~200–300 lines
- ~1,500–2,500 tokens

Contains:

- CategoriesScreen
- CategoriesViewModel
- UI state management

---

### Feature: Category Details

Directory:

```
ui/feature/category_details
```

Estimated size:

- ~200–300 lines
- ~1,500–2,500 tokens

Contains:

- detail screen
- ViewModel
- UI rendering logic

---

### Theme System

Directory:

```
ui/theme
```

Estimated size:

- ~150 lines
- ~800–1,200 tokens

Defines:

- colors
- typography
- shapes
- Material theme configuration

---

## Total Estimated Tokens

Approximate token count for the entire project:

| Component | Tokens |
|-----------|--------|
Application Core | 2,000
Dependency Injection | 1,000
Models | 1,500
UI Layer | 6,000
Categories Feature | 2,000
Category Details Feature | 2,000
Theme System | 1,000

Estimated total:

**15,000 – 18,000 tokens**

---

## Recommended Token Strategy for AI Usage

To effectively analyze this project with AI tools, the code should be split into logical units.

Recommended chunk sizes:

- one feature module
- one ViewModel
- one UI screen
- one dependency module

Example prompt chunks:

1. Categories feature
2. Category details feature
3. UI theme system
4. dependency injection modules

Each chunk stays within **1,500–3,000 tokens**, which is optimal for AI-assisted analysis.

---

## Conclusion

The Foodies project is small enough to be analyzed with AI tools when divided into features.  
Its architecture allows clear segmentation into UI features, models, and infrastructure layers, making it well suited for AI-assisted development workflows.
