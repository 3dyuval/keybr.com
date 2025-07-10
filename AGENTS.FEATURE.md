# Colored Characters Feature - Technical Analysis

## Feature Overview

**Colored Characters** feature with 3 modes:

1. **Off** (default) - Feature disabled
2. **Current Character** - Shows only current character in color-coded state
3. **All Characters** - Shows all characters in their color-coded states

## Architecture Analysis

### Affected Components

#### 1. Settings System

- **File**: `packages/keybr-textinput/lib/settings.ts`
- **Analysis**: Need to add new enum `ColoredCharactersStyle` and integrate into `TextDisplaySettings`
- **Impact**: Extends existing settings pattern, minimal disruption

#### 2. Text Display Components

- **File**: `packages/keybr-textinput-ui/lib/chars.tsx`
- **Analysis**: Core rendering logic for text characters via `renderChars` function
- **Impact**: Requires function signature modification to accept lesson data

#### 3. Lesson UI Components

- **File**: `packages/keybr-lesson-ui/lib/Key.tsx`
- **File**: `packages/keybr-lesson-ui/lib/styles.ts`
- **Analysis**: Contains existing color logic that needs to be shared
- **Impact**: Need to extract and export color utilities for reuse

#### 4. Text Styling System

- **File**: `packages/keybr-textinput-ui/lib/styles.ts`
- **Analysis**: Handles text character styling
- **Impact**: Need to add character color styling functions

### Existing Color System Analysis

- **Color Basis**: Confidence levels (slow=red, fast=green)
- **Color Functions**: `useKeyStyles()` hook provides `confidenceColor()` and `keyStyles()`
- **Color Calculation**: Uses `mixColors(min, max, confidence)` from `@keybr/color`
- **Current Usage**: Applied in `Key.tsx` via `style={keyStyles(true, confidence)}`

## Data Flow Architecture

### Key Data Structures

1. **LessonKey** (`packages/keybr-lesson/lib/key.ts:6`):

   - Contains `letter.codePoint`, `confidence`, `isIncluded`
   - Maps to specific characters via `codePoint`

2. **Char** (`packages/keybr-textinput/lib/chars.ts:15`):

   - Contains `codePoint`, `attrs`, `cls`
   - Used in text rendering

3. **Color System** (`packages/keybr-lesson-ui/lib/styles.ts:5`):
   - `useKeyStyles()` provides `confidenceColor(confidence)` function
   - Colors range from slow (red) to fast (green) based on confidence

### Data Flow Mapping

```
LessonKey.letter.codePoint → Char.codePoint → Character Color
LessonKey.confidence → confidenceColor() → Background Color
```

## HTML Structure Analysis

### Current Rendering Architecture

#### 1. **Lesson Keys Display** (Reference implementation):

```html
<span
  class="JbmeqBVGxy wwN0EvYtKj sXEw880Rnp"
  style="background-color: rgb(96, 215, 136);"
  data-code-point="101"
  >E</span
>
```

- Rendered by `Key.tsx` component
- Already implements `background-color` styling
- Includes `data-code-point` attributes for mapping

#### 2. **Practice Text Display** (Target for enhancement):

```html
<span class="nc1oZcWRbC" style="color: rgb(158, 172, 103);">b</span>
<span style="color: var(--textinput__color);">lizzard</span>
```

- Rendered by `chars.tsx` via `renderChars()`
- Currently only applies `color` (text color)
- **Target**: Add `background-color` based on settings
- Cursor character has special class `nc1oZcWRbC`

### Architecture Insight

The feature requires modifying practice text spans to add `background-color` styles that match lesson key colors, controlled by `ColoredCharactersStyle` setting.

### Enhanced Data Flow

```
LessonKey.confidence → confidenceColor() → Practice Text background-color
LessonKey.letter.codePoint → Char.codePoint → Color mapping
```

## Technical Feasibility Assessment

### Complexity Level: **Medium**

- Requires changes across 4-5 files
- Need to thread lesson data through text rendering pipeline
- Settings integration follows existing patterns

### Integration Points

1. **Settings Layer**: Standard enum pattern integration
2. **Data Layer**: Lesson keys need to be passed to text rendering
3. **Rendering Layer**: Color application logic in `renderChars()`
4. **UI Layer**: Settings dropdown integration

### Architectural Constraints

- Must maintain backward compatibility
- Should not impact performance when disabled
- Need to respect existing theme system
- Must work across all supported browsers
