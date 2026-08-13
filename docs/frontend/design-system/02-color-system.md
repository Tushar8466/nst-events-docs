# Color System

**Specification Status:** SPECIFICATION FROZEN
**Implementation Status:** PLANNED IMPLEMENTATION

## Semantic Color Architecture
Colors are defined by their semantic purpose rather than their literal hue. **Every color token used anywhere in the docs resolves to the hex codes in this file.** No other file may define a new color.

## Primary Palette (Brand Blue)
Used for brand identity, primary actions, and active states.
* `primary-base`: `#2563EB`
* `primary-hover`: `#1D4ED8`
* `primary-surface`: `#EFF6FF`

## Secondary & Accent Palette
Used for secondary actions, borders, and visual grouping.
* `secondary-base`: `#4B5563`
* `secondary-hover`: `#374151`
* `accent-base`: `#8B5CF6`

## Semantic States
* **Success**: `#16A34A` (Used for approvals, successful registrations)
  * `success-surface`: `#DCFCE7`
* **Warning**: `#D97706` (Used for waitlists, pending states)
  * `warning-surface`: `#FEF3C7`
* **Error**: `#DC2626` (Used for rejections, destructive actions, locked features)
  * `error-surface`: `#FEE2E2`
* **Info**: `#0284C7` (Used for standard announcements)
  * `info-surface`: `#E0F2FE`

## Neutral System
A comprehensive gray scale used for text hierarchy, borders, and surface elevations.
* `gray-50`: `#F9FAFB` (App Background)
* `gray-100`: `#F3F4F6` (Surface / Card Background)
* `gray-200`: `#E5E7EB` (Subtle Borders)
* `gray-300`: `#D1D5DB` (Strong Borders)
* `gray-400`: `#9CA3AF` (Muted Icons)
* `gray-500`: `#6B7280` (Muted Text)
* `gray-600`: `#4B5563` (Secondary Text)
* `gray-700`: `#374151` (Primary Text)
* `gray-800`: `#1F2937` (Headings)
* `gray-900`: `#111827` (Deep Headings)
* `white`: `#FFFFFF`

## Light and Dark Mode Behavior
Colors automatically invert their semantic mapping based on the active theme. 
* `bg-surface-primary`: Maps to `#FFFFFF` in Light Mode and `#111827` in Dark Mode.
* `text-primary`: Maps to `#111827` in Light Mode and `#F9FAFB` in Dark Mode.
* `border-default`: Maps to `#E5E7EB` in Light Mode and `#374151` in Dark Mode.
